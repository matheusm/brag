# Prompt: Classificacao em Categorias

Voce recebeu um item coletado de uma fonte de dados. Sua tarefa e classificar o item na categoria de impacto mais adequada.

## Categorias disponiveis

Use as categorias definidas no `brag.config.yaml` do projeto. Se nao houver config, use o set padrao:

- Features entregues
- Bug fixes
- Code reviews
- Lideranca tecnica
- Melhorias de processo
- Outros

## Heuristicas de classificacao

Aplique na seguinte ordem de prioridade:

### 1. Sinais explicitos (maior prioridade)

- PR com label `bug` ou ticket com tipo `Bug` -> **Bug fixes**
- PR review onde o usuario e reviewer (nao author) -> **Code reviews**
- Ticket/PR com label `feature`, tipo `Story` ou `Feature` -> **Features entregues**
- Items com labels `docs`, `documentation`, `process`, `rfc` -> **Melhorias de processo**

### 2. Sinais de contexto

- Titulo ou descricao menciona "fix", "hotfix", "patch", "crash", "error" -> **Bug fixes**
- Titulo ou descricao menciona "refactor", "migrate", "upgrade" sem ser bug -> **Features entregues** (se adiciona valor) ou **Melhorias de processo** (se e tecnico/infra)
- Item envolve mentoria, pairing, RFC, ADR, design doc -> **Lideranca tecnica**
- Item envolve CI/CD, tooling, developer experience -> **Melhorias de processo**

### 3. Inferencia por tipo de atividade

- PRs mergeados sem sinais claros -> **Features entregues** (default pra PRs de author)
- Issues fechadas sem sinais claros -> **Features entregues**

### 4. Fallback

- Se nenhuma heuristica se aplica -> **Outros**

## Regras

1. **Uma categoria por item.** Nao divida um item em multiplas categorias.
2. **Quando ambiguo, prefira a categoria que melhor representa o impacto** do ponto de vista de um performance review.
3. **Se o dev customizou as categorias**, mapeie as heuristicas pras categorias disponiveis da melhor forma possivel. Use o nome e a intencao da categoria pra decidir.
4. **Code reviews so contam quando o usuario e reviewer**, nao author. Se o usuario fez o PR e alguem revisou, o PR vai pra outra categoria.
