---
name: brag
description: "Collects your contributions from GitHub, Jira, and Linear and generates a brag document with impact-focused descriptions. Use this skill whenever the user mentions brag document, performance review, self-review, tracking accomplishments, collecting contributions, generating impact summaries, or wants to document what they've shipped — even if they don't say 'brag' explicitly."
---

# Brag Document Skill

Coleta dados de diversas fontes (GitHub, Jira, Linear) e gera automaticamente um brag document com descricoes de impacto.

## Comandos

Este skill responde a 3 comandos:

- `/brag init` — Setup interativo que gera o `brag.config.yaml`
- `/brag collect` — Coleta dados e gera entradas no brag document
- `/brag summary` — Gera um resumo consolidado de multiplos periodos

Identifique qual comando o usuario esta invocando pelos argumentos passados. Se nenhum argumento for passado, pergunte o que o usuario quer fazer.

---

## `/brag init`

Guie o dev pelo setup interativo. Faca uma pergunta por vez, preferindo multipla escolha:

1. **Granularidade do periodo:** daily | weekly | monthly | quarterly
2. **Fontes de dados:** Quais quer configurar? (GitHub, Jira, Linear)
3. **Pra cada fonte selecionada:**
   - Tipo de conexao: MCP ou API?
   - Se GitHub: username e lista de repos (formato `org/repo`)
   - Se Jira: base URL e project keys. Token via `JIRA_API_TOKEN` env var
   - Se Linear: team name. Token via `LINEAR_API_KEY` env var (se API)
   - Se GitHub API: token via `GITHUB_TOKEN` env var
4. **Categorias:** Usar o set padrao ou customizar?
   - Set padrao: Features entregues, Bug fixes, Code reviews, Lideranca tecnica, Melhorias de processo, Outros
5. **Schedule:** Quer configurar execucao automatica?
   - Opcao recomendada: Claude CronCreate
   - Alternativas: cron local, GitHub Action

Depois de coletar todas as respostas, leia o template em `templates/config.yaml` e gere o `brag.config.yaml` na raiz do projeto preenchido com as respostas do dev.

Se o dev escolheu schedule:
- **CronCreate:** Use a tool CronCreate pra agendar. Ex: `/brag collect --period yesterday` diariamente
- **Cron local:** Mostre o comando pronto pra copiar: `0 9 * * 1 cd /path/to/repo && claude -p "/brag collect --period last-week"`
- **GitHub Action:** Gere o arquivo `.github/workflows/brag-collect.yml` usando o template apropriado

---

## `/brag collect`

Coleta dados das fontes configuradas e gera entradas no brag document.

### Parametros

O usuario DEVE fornecer um range de data. Exemplos:
- `--period last-week` (semana calendario anterior, segunda a domingo)
- `--period yesterday` (dia anterior)
- `--period last-month` (mes calendario anterior)
- `--period 2026-03-01:2026-03-15` (range explicito, inclusivo)

Se o usuario nao fornecer, pergunte qual periodo quer coletar.

### Fluxo de execucao

**Passo 1: Ler config**

Leia o `brag.config.yaml` na raiz do projeto. Se nao existir, avise o dev pra rodar `/brag init` primeiro.

**Passo 2: Resolver datas**

Converta o periodo relativo em datas absolutas (inicio e fim). Todos os periodos relativos usam limites de calendario:
- `yesterday` = dia anterior completo
- `last-week` = segunda a domingo da semana anterior
- `last-month` = primeiro ao ultimo dia do mes anterior

**Passo 3: Coletar dados de cada fonte**

Pra cada fonte configurada no `brag.config.yaml`, leia as instrucoes do collector correspondente:
- GitHub: leia `collectors/github.md`
- Jira: leia `collectors/jira.md`
- Linear: leia `collectors/linear.md`

Siga as instrucoes do collector pra buscar os dados do periodo.

**Passo 4: Gerar narrativas de impacto**

Pra cada item coletado, leia `prompts/narrative.md` e siga as instrucoes pra gerar a descricao de impacto.

**Passo 5: Classificar em categorias**

Pra cada item, leia `prompts/classify.md` e siga as instrucoes pra classificar na categoria correta.

**Passo 6: Escrever no arquivo de periodo**

Determine o arquivo de saida baseado na config `period` e na data do item:
- `daily`: `{output_dir}/YYYY-MM-DD.md`
- `weekly`: `{output_dir}/YYYY-WNN.md`
- `monthly`: `{output_dir}/YYYY-MM.md`
- `quarterly`: `{output_dir}/YYYY-QN.md`

Se o arquivo ja existir, faca append das novas entradas nas secoes de categoria corretas.
Se o arquivo nao existir, crie-o usando o template em `templates/period.md`.

**Passo 7: Resumo**

Mostre pro dev um resumo do que foi adicionado: quantos itens por fonte e por categoria.

---

## `/brag summary`

Gera um resumo consolidado a partir de multiplos periodos.

### Parametros

- `--range 2026-Q1` (trimestre inteiro)
- `--range 2026-01:2026-03` (range de meses)

### Fluxo

1. Leia o `brag.config.yaml` pra saber o `output_dir` e o `period`
2. Identifique quais arquivos de periodo estao dentro do range solicitado
3. Leia todos esses arquivos
4. Leia `prompts/summary.md` e siga as instrucoes pra gerar o resumo executivo
5. Apresente o resumo ao dev e pergunte se quer salvar como arquivo (ex: `{output_dir}/summary-2026-Q1.md`)
