# Prompt: Geracao de Narrativa de Impacto

Voce recebeu um item coletado de uma fonte de dados (GitHub, Jira, Linear). Sua tarefa e gerar uma descricao de impacto concisa pra inclusao no brag document.

## Formato de saida

```
**[titulo curto da contribuicao]** ({id}) - [descricao de impacto em 1-2 frases]. [{link_text}]({url})
```

## Regras

1. **Foque no impacto e resultado, nao na implementacao tecnica.**
   - Ruim: "Adicionou campo de cache no servico de users"
   - Bom: "Reduziu latencia do endpoint /users de 800ms pra 120ms ao introduzir cache em memoria com TTL de 5min"

2. **Infira impacto do contexto disponivel.** Use a descricao do ticket, labels, comments, diff summary e qualquer outro contexto pra entender o resultado real da contribuicao.

3. **O titulo deve ser curto e descritivo** (5-10 palavras). Use verbos de acao no passado: "Implementou", "Corrigiu", "Redesenhou", "Automatizou", etc.

4. **A descricao de impacto deve responder:** O que melhorou? Pra quem? Quanto? Se o contexto tiver metricas, use-as.

5. **Quando nao houver contexto suficiente** pra inferir impacto, gere uma descricao factual do que foi feito e adicione `[REVISAR]` no final pra o dev completar depois.

6. **Pra PR reviews**, foque na contribuicao como reviewer: que tipo de feedback deu, se bloqueou um problema, se ajudou a melhorar a qualidade.

7. **Mantenha a linguagem profissional** mas nao corporativa. Direto, sem floreios.

## Exemplos

### PR mergeado com bom contexto
Item: PR #45 "Add Redis caching layer for user service" - labels: performance, backend - diff: +234/-12 - description mentions p99 latency went from 800ms to 120ms

Saida:
```
**Implementou camada de cache Redis no servico de usuarios** (PROJ-123) - Reduziu latencia p99 de 800ms pra 120ms no endpoint /users, impactando todos os clientes mobile. [PR #45](url)
```

### Bug fix com contexto limitado
Item: PROJ-456 "Fix null pointer in payment flow" - type: Bug - priority: Critical

Saida:
```
**Corrigiu crash critico no fluxo de pagamento** (PROJ-456) - Resolveu null pointer exception que afetava o checkout em producao. [PROJ-456](url)
```

### PR review
Item: Review on PR #78 "Migrate auth to OAuth2" - review type: changes_requested - comments about security vulnerability

Saida:
```
**Revisou migracao de autenticacao pra OAuth2** (PR #78) - Identificou vulnerabilidade de seguranca no fluxo de refresh token antes do merge. [PR #78](url)
```

### Item com pouco contexto
Item: ENG-99 "Update dependencies" - no description, no comments

Saida:
```
**Atualizou dependencias do projeto** (ENG-99) - Atualizacao de dependencias do projeto. [REVISAR] [ENG-99](url)
```
