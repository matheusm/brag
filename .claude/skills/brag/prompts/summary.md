# Prompt: Resumo Consolidado

Voce recebeu o conteudo de multiplos arquivos de periodo do brag document. Sua tarefa e gerar um resumo executivo consolidado.

## Formato de saida

```markdown
# Resumo - {periodo}

## Destaques

[3-5 bullets com as contribuicoes de maior impacto do periodo, selecionadas de todas as categorias]

## Por categoria

### Features entregues
[Resumo em 2-3 frases das features mais relevantes. Quantas no total.]

### Bug fixes
[Resumo. Quantos no total. Destacar os mais criticos.]

### Code reviews
[Resumo. Quantas no total. Temas recorrentes nos reviews.]

### Lideranca tecnica
[Resumo das contribuicoes de lideranca.]

### Melhorias de processo
[Resumo das melhorias.]

## Numeros

- Total de contribuicoes: X
- Features: X | Bugs: X | Reviews: X | Lideranca: X | Processo: X | Outros: X
```

## Regras

1. **Os destaques devem ser as contribuicoes que voce mais recomendaria mencionar** numa conversa de performance review. Priorize por impacto no negocio, nao volume de trabalho.

2. **Cada secao de categoria deve sintetizar, nao listar.** O dev ja tem as listas detalhadas nos arquivos de periodo. O summary e pra visao de alto nivel.

3. **Se uma categoria estiver vazia** (nao houver itens), omita a secao.

4. **Use as categorias do config**, nao as do set padrao, se forem customizadas.

5. **O tom deve ser profissional e orientado a impacto.** Este resumo pode ser usado diretamente em self-reviews ou conversas com gestores.

6. **Inclua metricas quando disponiveis.** Se os itens originais mencionam numeros (latencia, uptime, reducao de bugs), traga esses numeros pro resumo.

7. **Mantenha o resumo conciso.** O objetivo e que o dev leia em 2-3 minutos e tenha uma visao clara do periodo.
