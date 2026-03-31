# GitHub Collector

Instrucoes pra coletar dados do GitHub dentro de um range de datas.

## Dados do config

Do `brag.config.yaml`, extraia:
- `sources.github.type`: `mcp` ou `api`
- `sources.github.username`: username do dev no GitHub
- `sources.github.repos`: lista de repos no formato `org/repo`

## O que coletar

1. **PRs mergeados** pelo usuario no periodo
2. **PR reviews** feitas pelo usuario (onde ele e reviewer, nao author)
3. **Issues fechadas** pelo usuario no periodo

## Via MCP (`type: mcp`)

Use as tools do GitHub MCP server. Pra cada repo na lista:

### PRs mergeados

Busque PRs mergeados pelo usuario no periodo. Pra cada PR, colete:
- Titulo
- Numero do PR
- URL
- Descricao (body)
- Labels
- Data do merge
- Diff summary (files changed, additions, deletions)
- Review comments relevantes

### PR Reviews

Busque PRs onde o usuario fez review no periodo. Pra cada review, colete:
- Titulo do PR
- Numero do PR
- URL
- Author do PR (pra contexto)
- Tipo de review (approved, changes requested, commented)
- Comentarios do reviewer

### Issues fechadas

Busque issues fechadas pelo usuario no periodo. Pra cada issue, colete:
- Titulo
- Numero
- URL
- Labels
- Body/descricao

## Via API (`type: api`)

Use o Bash tool pra fazer chamadas curl a GitHub API. O token esta em `GITHUB_TOKEN` env var.

### PRs mergeados

```bash
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/issues?q=type:pr+author:{username}+repo:{org/repo}+merged:{start_date}..{end_date}"
```

Pra cada PR retornado, busque detalhes adicionais:

```bash
# Detalhes do PR (inclui body, labels)
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/{org/repo}/pulls/{number}"

# Files changed (diff summary)
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/{org/repo}/pulls/{number}/files"
```

### PR Reviews

```bash
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/issues?q=type:pr+reviewed-by:{username}+repo:{org/repo}+updated:{start_date}..{end_date}"
```

Pra cada PR, busque os reviews do usuario:

```bash
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/repos/{org/repo}/pulls/{number}/reviews"
```

Filtre apenas reviews feitas pelo usuario e dentro do periodo.

### Issues fechadas

```bash
curl -s -H "Authorization: token $GITHUB_TOKEN" \
  "https://api.github.com/search/issues?q=type:issue+assignee:{username}+repo:{org/repo}+closed:{start_date}..{end_date}"
```

## Tratamento de erros

- Se o MCP server nao estiver disponivel, avise o dev: "GitHub MCP server nao encontrado. Configure o MCP server ou altere `sources.github.type` pra `api` no `brag.config.yaml`."
- Se o token API nao estiver configurado, avise: "GITHUB_TOKEN env var nao encontrada. Configure o token pra usar a GitHub API."
- Se um repo nao existir ou o usuario nao tiver acesso, avise e continue com os outros repos.

## Formato de saida

Retorne uma lista de items no formato:

```
{
  source: "github",
  type: "pr_merged" | "pr_review" | "issue_closed",
  title: "...",
  id: "org/repo#123",
  url: "https://github.com/...",
  date: "2026-03-15",
  context: "... descricao, labels, diff summary, comments ..."
}
```
