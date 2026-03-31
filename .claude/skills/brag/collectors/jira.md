# Jira Collector

Instrucoes pra coletar dados do Jira dentro de um range de datas.

## Dados do config

Do `brag.config.yaml`, extraia:
- `sources.jira.type`: `mcp` ou `api`
- `sources.jira.base_url`: URL base do Jira (ex: `https://company.atlassian.net`)
- `sources.jira.project_keys`: lista de project keys (ex: `PROJ`, `PLATFORM`)

## O que coletar

1. **Tickets resolvidos/fechados** pelo usuario no periodo (status mudou pra Done, Closed, Resolved, ou equivalente)

## Via MCP (`type: mcp`)

Use as tools do Jira MCP server.

Busque issues que foram movidas pra status de conclusao pelo usuario no periodo. Pra cada issue, colete:
- Key (ex: PROJ-123)
- Titulo (summary)
- Tipo (Bug, Story, Task, Epic, etc.)
- Descricao
- Labels
- Priority
- Data de resolucao
- Comentarios relevantes (especialmente do usuario)
- URL

## Via API (`type: api`)

Use o Bash tool pra fazer chamadas curl a Jira REST API. O token esta em `JIRA_API_TOKEN` env var. A autenticacao usa email + API token em base64.

O usuario precisa configurar tambem `JIRA_USER_EMAIL` env var.

### Tickets resolvidos

```bash
curl -s -H "Authorization: Basic $(echo -n "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" | base64)" \
  -H "Content-Type: application/json" \
  "{base_url}/rest/api/3/search?jql=project in ({project_keys}) AND status changed to (Done, Closed, Resolved) DURING ({start_date}, {end_date}) AND assignee = currentUser()&fields=summary,issuetype,description,labels,priority,resolutiondate,comment"
```

Pra cada issue retornada, a resposta ja inclui os campos necessarios. Se precisar de mais detalhes:

```bash
curl -s -H "Authorization: Basic $(echo -n "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" | base64)" \
  -H "Content-Type: application/json" \
  "{base_url}/rest/api/3/issue/{issue_key}"
```

## Tratamento de erros

- Se o MCP server nao estiver disponivel, avise: "Jira MCP server nao encontrado. Configure o MCP server ou altere `sources.jira.type` pra `api` no `brag.config.yaml`."
- Se o token API nao estiver configurado, avise: "JIRA_API_TOKEN env var nao encontrada. Configure o token pra usar a Jira API."
- Se `JIRA_USER_EMAIL` nao estiver configurada (pra tipo API), avise: "JIRA_USER_EMAIL env var nao encontrada. Configure o email pra autenticacao na Jira API."
- Se um project key nao existir ou nao tiver acesso, avise e continue com os outros.

## Formato de saida

Retorne uma lista de items no formato:

```
{
  source: "jira",
  type: "ticket_resolved",
  title: "...",
  id: "PROJ-123",
  url: "https://company.atlassian.net/browse/PROJ-123",
  date: "2026-03-15",
  issue_type: "Bug" | "Story" | "Task" | "Epic" | ...,
  context: "... descricao, labels, priority, comments ..."
}
```
