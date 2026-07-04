---
tipo: projeto
codinome: SINAPSE
status: ideia
criado: 2026-07-04
ultima-atualizacao: 2026-07-04
---

# SINAPSE — enriquecimento do vault via H100 (sem vazar o sensível)

> **Como invocar:** é só me dizer **"SINAPSE"** (ou "roda o SINAPSE", "vamos de SINAPSE"). Eu abro esta nota, retomo o plano de onde parou e a gente executa. Pode virar um `/sinapse` de verdade (slash command do Claude Code) quando você quiser — é só pedir.

## O que é

Deixar o vault **o mais rico possível**: uma LLM forte varre tudo, cria wikilinks, acha padrões não-óbvios entre domínios (saúde ↔ carreira ↔ dinheiro ↔ estudos) e gera embeddings por baixo. O objetivo é muita matéria bruta + inteligência varrendo = segundo cérebro conectado.

O problema que o SINAPSE resolve: aproveitar a **H100 do Recogna** (grátis pra Fernando como pesquisador) sem colocar o dado íntimo (saúde mental, finanças, CPF) num servidor compartilhado, onde root, backups e `rm`-que-não-apaga deixam sombra que não some.

## A sacada central

**Ocultar identificador ≠ ocultar significado.** Redigir o *tema* de uma nota de saúde mata justamente o sinal que a gente quer conectar. Então não se tenta "esconder e mandar tudo". Em vez disso:

**Embedding é portátil e o encoder é o mesmo modelo dos dois lados.** Logo dá pra embedar a fatia sensível *localmente* e juntá-la ao grafo que a H100 devolveu — as conexões cross-domain se formam na máquina do Fernando, e o dado sensível nunca sobe.

## Arquitetura

1. **Na H100 (volume não-sensível: IC, projetos, estudos — ~90% da massa)**
   - Roda um **encoder de embedding dedicado** (BGE-M3 / E5-multilingual / gte — multilíngue, pt-BR), **não** o LLM generativo. Gerar embedding com 70B é britadeira em parafuso.
   - LLM forte varre esse subconjunto: propõe wikilinks e padrões não-óbvios.
   - Tokeniza identificadores no que subir, por higiene (`Fernando`→`P-A`, valores→`VALOR-n`), com tabela de mapeamento que **nunca sai da máquina local**.
   - Devolve: a **matriz de embeddings** + as conexões propostas.

2. **Local (fatia sensível: saúde, dinheiro — conjunto pequeno)**
   - Roda o **mesmo encoder** localmente (embedding é barato, roda até em CPU) sobre as notas sensíveis.
   - **Junta** esses embeddings contra a matriz que voltou → conexões saúde↔carreira↔dinheiro nascem aqui, offline.
   - O insight generativo em prosa sobre o sensível fica com o Ciro (sob demanda) ou um LLM local do Fernando — nunca na H100.

3. **Portão de revisão**
   - Tudo entra como **proposta** (git diff), nunca escrita cega. Conexão boa é precisão, não volume.

## O que se ganha e o que se abre mão

- **Ganha:** vault riquíssimo, conexões cross-domain reais, compute pesado grátis, zero dado íntimo em backup alheio.
- **Abre mão:** o LLM generativo forte não lê o texto das notas sensíveis pra escrever prosa sobre elas — isso passa a ser feito local/sob demanda, sobre um conjunto pequeno.

## Ética (registrado, não escondido)

Rodar o volume de pesquisa na H100 do Recogna, como pesquisador do Recogna, é uso-fim defensável. O que o SINAPSE deliberadamente **não faz** é subir saúde/finanças pra lá — não por regra do lab, mas porque é o material mais íntimo do Fernando e servidor compartilhado é o lugar errado pra ele morar. A arquitetura foi desenhada pra não precisar disso.

## Pendências antes de rodar

- [ ] Escolher o encoder de embedding (candidatos: BGE-M3, E5-multilingual-large, gte-multilingual).
- [ ] Confirmar acesso/perfil no `jesus16` e política de uso pessoal (conversa informal no lab).
- [ ] Definir o formato de saída das conexões (proposta em `.md` para revisão via git diff).
- [ ] Escolher o LLM local do passo generativo sensível (ou deixar com o Ciro).
- [ ] Escrever os scripts: (a) tokenização + upload, (b) embedding+varredura na H100, (c) join local, (d) geração de propostas.

## Relacionadas

- [[04-estudos/ic/projeto-atual]] — a IC roda no mesmo servidor (`jesus16`, H100).
- [[02-saude/metricas]], [[03-dinheiro/metricas]] — as fatias que ficam locais.
