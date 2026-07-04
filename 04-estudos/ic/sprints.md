---
tipo: projeto
area: estudos
ultima-atualizacao: 2026-07-04
---

# IC — Linha do tempo das sprints

> Resumo garimpado de `~/dev/petroleum-nlp-ic/docs/SPRINTS.md`. Detalhes técnicos completos ficam no repositório de código, fora do vault. Ver também [[04-estudos/ic/projeto-atual]].

## Sprint 1 — coleta inicial de corpus (início de 2026)
- Coleta de livros técnicos (Thomas, SPE Handbook, Craft & Hawkins, Bourgoyne).
- Busca por datasets prontos: 3W Dataset (Petrobras), dados ANP/BDEP.
- Esboço inicial de script de extração de PDF.
- Aprendizado: bibliografia técnica é majoritariamente em inglês, faltam fontes PT-BR.

## Sprint 2 — parseamento + benchmark inicial
- Mais livros parseados com sucesso.
- Início da coleta de provas antigas Petrobras (PCI Concursos) para montar benchmark.
- Glossário PetroWiki: 5.167 termos coletados.
- Baseline `llama3.2:3b`: 28,1% de acurácia (dataset drbodebench, 591 questões, 141 de petróleo).
- Esclarecimento aos orientadores sobre o PetroGold: corpus CoNLL-U de 8.946 sentenças técnicas (POS, lema, dependências) extraídas de teses/monografias — não é dicionário.

## Sprint 3 — parser regex de provas
- `build_benchmark.py`: parser regex completo (Cesgranrio múltipla escolha + CEBRASPE certo/errado), gabarito cargo-aware.
- Output: 640 questões, 545 com gabarito, 141 classificadas como petróleo.
- Aprendizado técnico: `pdfplumber` perde subscritos/sobrescritos em fórmulas (problema que só seria resolvido na sprint 5/6).

## Sprint 4 — primeiro baseline + acesso a servidor
- Baseline `llama3.2:3b` (GTX 1650) em 128 questões: 28,1% (aleatório = 20%).
- Acesso confirmado ao servidor Recogna: L40S 48GB (`jesus17`) e H100 (host posterior).
- Orientadores aprovaram seguir para baselines em modelos maiores.

## Sprint 5 — pipeline LLM (Gemini) + baselines via OpenRouter (apresentação 19/mai/2026)
- Pipeline de extração assistida por LLM (Gemini 2.5 Flash multimodal), inspirado no EduBench dos orientadores, com 5 camadas de validação contra falso-positivo.
- Extração: 642 questões (vs 599 do regex), 633 com gabarito (+16%), 223 de petróleo (+58%).
- Cross-check regex×LLM: 423 validadas + 238 pendentes de revisão manual.
- Baselines (pivot: servidor caiu na véspera, rodado via OpenRouter): Gemma 3 27B 29,0%→31,8%, Llama 3.2 3B 26,1%→29,1%, Llama 3.1 8B 25,7%→27,3%, Qwen 2.5 72B 25,5%→28,8% (regex→Gemini). Ganho médio +2,7pp trocando só o extrator.
- Achado pós-apresentação: bug crítico no prompt do baseline (não enviava `textos_apoio` nem descrições de imagem aos modelos). Corrigido no mesmo dia; ganho validado de +1,9pp em questões com texto de apoio e +2,9pp em questões dependentes de figura.
- Reação dos orientadores: resultados "meio estranhos", pediram rodar de fato no servidor (H100) e revisão manual — motivou a investigação que achou o bug do prompt.

## Sprint 6 — em andamento (desde 20/mai/2026)
- Fix de bug na atribuição de página das imagens no `pdf_extract.py` (recursos de PDF compartilhados faziam todas as imagens caírem na página 1). Corrigido usando `get_image_info(xrefs=True)`.
- Re-extração cirúrgica de 109 questões candidatas: 103 recuperadas, 5 viraram `requer_imagem=True` com descrição nova.
- Baselines finais rodados no servidor `jesus16` (1× H100, via Ollama), 6 configurações de modelo/quantização × 2 datasets (633q Gemini, 545q regex), decodificação determinística.
- **Melhor resultado: Qwen 2.5 32B int8 — 33,2% no dataset de 633 questões.**
- Pendências para fechar a sprint: validação manual estratificada dos 238 casos "pending", análise por subárea técnica, re-rodar Qwen 2.5 72B (anomalia de respostas vazias via OpenRouter ainda não resolvida), investigar solução para perda de subscritos em fórmulas.

## Lacunas (só o Fernando sabe)
- Datas exatas das apresentações das sprints 1–4 (documento cita "Notion").
- Data confirmada da apresentação da sprint 6.
- Status atual dos pulls/experimentos WhatsApp com Lino e Paiola sobre bases de dados (arquivos de chat não foram lidos por serem sensíveis).
