# Synthesis

Pipeline de integração técnica + fundamental. Lê os outputs do TemplateAnalizer e do MacroSentiment, adiciona calendário económico, e produz análise integrada por asset e resumo do dia.

---

## O que faz

1. Lê `latest.json` do TemplateAnalizer (análise técnica por asset e TF)
2. Lê `latest.json` do MacroSentiment (análise fundamental e notícias)
3. Pesquisa calendário económico das próximas 48h
4. Corre um agente Sonnet por asset — integra técnica + fundamental + calendário
5. Produz resumo do dia com ranking de oportunidades e mapa de risco
6. Publica página HTML com tudo integrado

**~90-120 segundos. ~$0.30-0.50 por run.**

---

## Agentes

| Agente | Modelo | Input | Output |
|---|---|---|---|
| Calendário | Haiku + web search | Data | Eventos próximas 48h |
| Asset (×N) | Sonnet | Técnica + Macro + Calendário | Análise integrada por asset |
| Resumo do dia | Sonnet | Todos os reports de asset | Narrativa + ranking + alertas |

---

## Stack

| Componente | Tecnologia | Custo |
|---|---|---|
| Pipeline cloud | Modal.com | ~$0/mês compute |
| Calendário | Claude Haiku (web search) | ~$0.01/run |
| Análise por asset | Claude Sonnet | ~$0.15-0.25/run |
| Resumo | Claude Sonnet | ~$0.05/run |
| Hosting | GitHub Pages | Grátis |

---

## Estrutura

```
synthesis/
    pipeline_synth.py   ← pipeline Modal
    run_synth.py        ← script local
    config.json         ← keys locais (não commitar)
```

---

## Configuração inicial

### 1. Segredo Modal

```bash
modal secret create synthesis \
  ANTHROPIC_API_KEY=sk-ant-... \
  GITHUB_PAT=ghp_... \
  GITHUB_REPO_TV=sergioB79/analise \
  GITHUB_REPO_MACRO=sergioB79/macro-sentiment \
  GITHUB_REPO_SYNTH=sergioB79/synthesis
```

### 2. `config.json` local

```json
{
    "GITHUB_PAT": "ghp_...",
    "GITHUB_REPO_TV": "sergioB79/analise",
    "GITHUB_REPO_MACRO": "sergioB79/macro-sentiment",
    "GITHUB_REPO_SYNTH": "sergioB79/synthesis"
}
```

### 3. Deploy

```bash
python -m modal deploy pipeline_synth.py
```

---

## Uso

Correr sempre depois dos outros dois pipelines:

```bash
# 1. Análise técnica (TemplateAnalizer)
python TemplateAnalizer\run.py

# 2. Análise macro (MacroSentiment)
python MacroSentiment\run_macro.py

# 3. Síntese integrada (este)
python Sintetizador\run_synth.py
```

Output:

```
Synthesis Pipeline
────────────────────────────────────────
Data: 2026-04-09
Fontes: TradingView + Macro Sentiment
────────────────────────────────────────
A enviar para Modal...
────────────────────────────────────────
Pipeline concluída.
Página: https://sergiob79.github.io/synthesis/2026-04-09.html
Assets: EURUSD, GER40, XAUUSD
Duração: 112s
────────────────────────────────────────
CUSTO DO RUN:
  Haiku:  $0.012
  Sonnet: $0.341
  ──────────────────────────
  TOTAL:  ~$0.353
────────────────────────────────────────
```

---

## Output

**Índice:** `https://sergiob79.github.io/synthesis/`
**Análise por data:** `https://sergiob79.github.io/synthesis/YYYY-MM-DD.html`

Cada página contém:
- Resumo do dia com narrativa, ranking de oportunidades e alertas de calendário
- Calendário económico das próximas 48h
- Análise integrada por asset (técnica + fundamental + risco de calendário)

---

## Dependências

Este pipeline depende de:

| Pipeline | Repo | Dados lidos |
|---|---|---|
| TemplateAnalizer | `sergioB79/analise` | `latest.json` — análise técnica por asset |
| MacroSentiment | `sergioB79/macro-sentiment` | `latest.json` — análise macro e notícias |

Corre os outros dois primeiro. Se `latest.json` não existir ou for de data diferente, o pipeline avisa.

---

## Parte do sistema

```
TemplateAnalizer  →  análise técnica de gráficos
MacroSentiment    →  análise fundamental e notícias
Synthesis         →  integração técnica + fundamental  ← este repo
```

---

*S. Batalha · Nazaré · 2026*
