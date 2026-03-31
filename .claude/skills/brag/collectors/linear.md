# Linear Collector

Instrucoes pra coletar dados do Linear dentro de um range de datas.

## Dados do config

Do `brag.config.yaml`, extraia:
- `sources.linear.type`: `mcp` ou `api`
- `sources.linear.team`: nome do team no Linear

## O que coletar

1. **Issues completadas** pelo usuario no periodo (status Done ou Canceled com resolucao)

## Via MCP (`type: mcp`)

Use as tools do Linear MCP server.

Busque issues completadas atribuidas ao usuario no periodo. Pra cada issue, colete:
- Identifier (ex: ENG-123)
- Titulo
- Descricao
- Labels
- Priority
- Data de conclusao
- Comentarios relevantes
- URL
- Project (se aplicavel)

## Via API (`type: api`)

Use o Bash tool pra fazer chamadas curl a Linear GraphQL API. O token esta em `LINEAR_API_KEY` env var.

### Issues completadas

```bash
curl -s -X POST \
  -H "Authorization: $LINEAR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "query { issues(filter: { team: { name: { eq: \"{team}\" } }, completedAt: { gte: \"{start_date}\", lte: \"{end_date}\" }, assignee: { isMe: { eq: true } } }) { nodes { identifier title description url completedAt priority labels { nodes { name } } comments { nodes { body user { isMe } } } project { name } } } }"
  }' \
  "https://api.linear.app/graphql"
```

A query retorna todas as issues completadas pelo usuario no periodo com os campos necessarios.

## Tratamento de erros

- Se o MCP server nao estiver disponivel, avise: "Linear MCP server nao encontrado. Configure o MCP server ou altere `sources.linear.type` pra `api` no `brag.config.yaml`."
- Se o token API nao estiver configurado, avise: "LINEAR_API_KEY env var nao encontrada. Configure o token pra usar a Linear API."
- Se o team nao existir, avise e pare a coleta do Linear.

## Formato de saida

Retorne uma lista de items no formato:

```
{
  source: "linear",
  type: "issue_completed",
  title: "...",
  id: "ENG-123",
  url: "https://linear.app/team/issue/ENG-123",
  date: "2026-03-15",
  context: "... descricao, labels, priority, comments, project ..."
}
```
