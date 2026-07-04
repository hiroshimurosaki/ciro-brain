---
tipo: projeto
area: estudos
ultima-atualizacao: 2026-07-04
---

# IC — Projeto atual

> Preenchido no onboarding e atualizado conforme a IC evolui.

- **Tema**: Construção de corpora e benchmarks técnicos de PLN para petróleo e gás em português brasileiro. Questão central: "como construir corpora e benchmarks técnicos confiáveis de petróleo & gás em PT-BR — e o que eles revelam sobre o conhecimento de domínio de LLMs abertas?"
- **Vínculo**: bolsa Petrobras / projeto WASP, laboratório RECOGNA (UNESP Bauru, FC/DCo).
- **Orientador(a)**: Prof. Dr. João Paulo Papa (supervisor). Supervisão direta: Gabriel Lino Garcia e Pedro Henrique Paiola (orientadores de pós).
- **Repositório de código**: `~/dev/petroleum-nlp-ic` (fora do vault).
- **Estágio atual**: Sprint 6 em andamento (desde 20/mai/2026). Pipeline de extração LLM-assistida (Gemini 2.5 Flash) substituiu o parser regex como fonte principal do benchmark. Baselines zero-shot já rodados no servidor Recogna (H100, `jesus16`) com controle de quantização — melhor resultado até agora: Qwen 2.5 32B int8, 33,2% no dataset de 633 questões.
- **Próxima entrega / reunião**: apresentação da sprint 6 prevista ~2 semanas após 20/mai/2026 (data exata a confirmar com Fernando/Notion).

## Contexto

- **Dados**: provas técnicas Petrobras (Cesgranrio 2011–2018, múltipla escolha A–E; CEBRASPE 2022, certo/errado), coletadas via PCI Concursos. Glossários técnicos: PetroWiki (5.167 termos, scraping com Selenium por causa de CAPTCHA), Petrolês e ABESPETRO (planejados/em progresso, para uso futuro em RAG/fine-tuning). Corpus PetroGold: 8.946 sentenças técnicas em formato CoNLL-U (POS, lema, dependências), extraídas de monografias e teses.
- **Dois extratores em paralelo**: parser regex (`scripts/benchmark/build_benchmark.py`, baseline de comparação) e pipeline LLM-assistido (Gemini 2.5 Flash multimodal, inspirado na arquitetura EduBench dos orientadores) — este último tornou-se a fonte principal por extrair mais questões, cobrir mais gabaritos e capturar melhor figuras/textos de apoio.
- **Achado central da sprint 5/6**: a qualidade da extração de questões impacta diretamente a acurácia medida dos modelos — trocar só o extrator (regex → Gemini) rendeu +2,7pp em média nos 4 modelos testados. Um bug no prompt do baseline (não repassava `textos_apoio` nem descrições de imagem) foi corrigido no mesmo dia da apresentação da sprint 5.
- **Infraestrutura**: servidor Recogna (`jesus16`/`jesus17`, GPUs H100 e L40S) para rodar modelos via Ollama; OpenRouter como fallback quando o servidor está indisponível.
- Histórico completo de cada sprint em [[04-estudos/ic/sprints]].

## Próximos passos

- Validação manual estratificada dos 238 casos "pending" do cross-check regex×LLM (via HTML viewer já construído).
- Análise por subárea técnica (reservatório, perfuração, completação, geologia, geofísica, refino, produção).
- Re-rodar Qwen 2.5 72B em provider estável (anomalia de respostas vazias via OpenRouter ainda não resolvida).
- Investigar solução para perda de subscritos/sobrescritos do `pdfplumber` em fórmulas (via OCR ou renderização de página inteira).
- Corpora alimentando etapas futuras de RAG / fine-tuning com vocabulário técnico.
