---
tipo: projeto
status: pausado
stack: Python, Claude API (Anthropic), Notion API, PDF/PPTX (pypdf, python-pptx)
última_atualização: 2025-04-11
---

# StudyFlow — Assistente Inteligente de Aprendizagem via Notion

## O que é

App Python que transforma slides universitários (PDF/PPTX) em **aulas didáticas personalizadas**, gera exercícios variados por tipo de conteúdo e acompanha progresso via Notion. Fluxo:

1. **Gerar (PC):** extrai slides → estrutura aulas → gera conteúdo com Claude → cria exercícios → popula Notion
2. **Estudar:** lê aula no Notion (PC ou celular), faz aquecimento
3. **Responder (celular):** preenche exercícios direto no Notion
4. **Corrigir (PC — automático):** app detecta respostas, corrige com Claude, adiciona feedback, atualiza Dashboard

## Stack

- **Core:** Python 3
- **API de IA:** Claude (Anthropic) — múltiplos modelos conforme tarefa
- **Integração:** Notion API (criação/atualização de páginas e bancos de dados)
- **Processamento de documentos:** pypdf, python-pptx
- **Config:** variáveis de ambiente (.env)

## Modelos Claude por tarefa

| Tarefa | Modelo |
|--------|--------|
| Estruturar tópicos dos slides | Haiku (rápido, barato) |
| Gerar conteúdo das aulas | Sonnet (streaming) |
| Gerar exercícios | Sonnet |
| Corrigir dissertativo/código/cálculo | Sonnet |
| Gerar material de reforço | Sonnet |
| Múltipla escolha | Sem Claude (comparação local) |

## Tipos de exercício

- **Múltipla Escolha:** aquecimento (início de aula), 1 por aula
- **Dissertativo:** conceitos teóricos, comparações
- **Código (Escrever):** de exemplos dos slides
- **Código (Refatorar):** padrões de qualidade
- **Cálculo:** fórmulas, complexidade, métricas

**Distribuição:** Modo completo = 5 ex./aula | Modo expresso = 3 ex./aula (prova)

## Arquivos principais

- `main.py` — CLI principal (gerar, monitorar, corrigir, status)
- `extractor.py` — extração de PPTX/PDF
- `generator.py` — geração de aulas via Claude
- `exercise_builder.py` — geração de exercícios
- `corrector.py` — correção de respostas
- `notion_writer.py` — criação/atualização no Notion
- `notion_monitor.py` — loop de monitoramento de respostas
- `models.py` — seleção de modelos Claude por tarefa
- `prompts.py` — centralizados
- `config.py` — constantes e configurações

## Comandos principais

```bash
python main.py gerar "Cálculo II" slides.pdf --horas 10 --modo completo
python main.py monitorar   # loop contínuo de detecção e correção
python main.py corrigir "Matéria" --aula 1
python main.py status      # progresso geral (barra visual, notas)
```

## Estado atual

- **Pausado** — estrutura core funcional, integração Notion + Claude operacional
- Último commit: 2025-04-11
- Cache local `.studyflow_cache/` (nunca vai ao Notion nem git)
