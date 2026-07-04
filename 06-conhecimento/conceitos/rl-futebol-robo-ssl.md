---
tipo: conhecimento
titulo: Reinforcement Learning em Futebol de Robô (SSL)
tags:
  - RL
  - futebol-robo
  - IA
  - multi-agent
  - path-planning
  - decision-making
fonte: "estudos internos (TIGERs, RobôCIn, ETH Zürich, WisTex); papers ICRA/RoboCup 2024"
data: 2026-07-04
relacionado:
  - "[[05-projetos-carreira/carrossel-caipira]]"
---

# Reinforcement Learning em Futebol de Robô (SSL)

## Conceito

**RL em SSL é fundamentalmente diferente de RL em jogos de vídeo:** há muito poucos dados (5-7 partidas/time/competição), latência de comunicação, sim-to-real gap significativo e recompensa extremamente esparsa (gol). Por isso, os times do mundo não usam RL end-to-end, mas **RL cirúrgico** em pontos específicos da stack clássica de controle.

## 5 Estratégias Viáveis em Ordem de Complexidade

### 1. **Pass Scoring com ML Incremental** (Baixo-Médio | Em Uso)

**O quê:** Sistema heurístico de pontuação de passes que aprende os pesos durante a partida.

**Como:** 
- Gera candidatos de passe (pontos no campo)
- Filtra inviáveis (bloqueado, fora de área útil)
- Cada candidato recebe score = f(ângulo_chute, prob_interceptação, timing)
- **A cada gol marcado:** rastreia que passes levaram ao gol e aplica bônus naquele tipo de jogada

**Prós:** aprende com ~0 dados pré-existentes, totalmente interpretável, sem sim-to-real gap  
**Contras:** não generaliza para novos adversários, plays criativas nunca tentadas não surgem

**Onde:** módulo `Strategy` do Carrossel — sistema de avaliação de passes

---

### 2. **RL para Path Planning** (Médio-Baixo | Validado em Simulação)

**O quê:** Agente RL treina em simulador para desviar dinamicamente de obstáculos.

**Como:**
- Ambiente: simulação grSim do campo SSL
- Observação: posição, velocidade, destino, obstáculos
- Ação: velocidade alvo
- Recompensa: +para chegar ao destino, -colisão, -tempo

**Resultado:** 60% ganho de tempo vs. algoritmos clássicos; desvio emergente de obstáculos em movimento

**Prós:** ganho mensurável, aprende em simulação, relativamente simples  
**Contras:** sim-to-real gap — calibração em hardware físico é necessária

**Onde:** módulo `Control` do Carrossel + ambiente rSoccer para treino

**Referência:** RobôCIn (UFPE) — arxiv.org/abs/2404.15410

---

### 3. **Adaptive Play Selection (Multi-Armed Bandit)** (Baixo | Em Uso Observado)

**O quê:** Seleciona estratégias (Plays) com base em taxa de sucesso durante a partida.

**Como:**
- Mantém um conjunto de Plays pré-definidas
- Monitora taxa de gol/sucesso de cada Play
- Ajusta probabilidades em tempo real (UCB ou epsilon-greedy)
- Explora Plays que funcionam contra este adversário específico

**Prós:** extremamente leve (sem rede neural), não requer dados, adaptativo em tempo real  
**Contras:** só aprende dentro de 1 partida, espaço de Plays precisa ser pré-definido

**Onde:** sistema `Plays` do Carrossel

**Nota:** Observado em CMDragons (Carnegie Mellon), sem paper público

---

### 4. **RL + Stack Clássica (Sub-Behaviors)** (Médio-Alto | Validado em Competição)

**O quê:** Decompõe comportamento em 4 agentes RL semi-independentes, selecionados por heurística.

**Os 4 sub-agentes:**
1. Navegação geral
2. Aproximação e controle de bola
3. Chute/decisão de passe
4. Posicionamento defensivo

**Como:** em cada momento, módulo heurístico decide qual sub-agente está ativo.

**Treinamento:** simulador baixa fidelidade (treino em campo completo) + simulador alta fidelidade (bola)

**Resultado:** 7 vitórias em 8 jogos na SPL 2024; placar 39×7

**Prós:** funciona em hardware real, conhecimento humano nas transições, cada sub-agente é simples, fácil de depurar  
**Contras:** seleção heurística ainda codificada manualmente (gargalo), não aborda coordenação multi-robô

**Onde:** estrutura de Skills do Carrossel → cada Skill poderia evoluir para agente RL

**Referência:** WisTex United — arxiv.org/abs/2412.09417 (nota: SPL, não SSL, mas conceitos transferem)

---

### 5. **MARL Descentralizado (Multi-Agent RL)** (Alto | Pesquisa, Futuro)

**O quê:** Cada robô é um agente RL independente que aprende política compartilhada emergentemente.

**Arquitetura (MARLadona, ETH Zürich):**
- Global Entity Encoder: cada agente codifica todos os entidades via max pooling (invariante a permutação)
- CTDE: treinamento centralizado, execução descentralizada
- Self-play para gerar adversários progressivamente duros
- Isaac Gym (GPU-acelerado)

**Resultado:** 66,8% taxa de vitória vs. HELIOS em 11v11

**Prós:** comportamento emergente de trabalho em time, escala para times completos, robusto a mudanças de adversário  
**Contras:** testado APENAS em simulação, sim-to-real gap MASSIVO, requer GPU, difícil depurar ("por quê?"), sem garantias de segurança

**Onde:** pesquisa futura, ponto de chegada de longo prazo

**Referência:** MARLadona — arxiv.org/abs/2409.20326

---

## Por Que Importa

1. **Cada estratégia resolve um problema diferente:**
   - Pass Scoring → decisão tática (onde passar)
   - Path Planning RL → movimentação eficiente
   - Adaptive Selection → adaptação ao adversário
   - Sub-behaviors → híbrido com controle humano
   - MARL → coordenação completa (pesquisa)

2. **Progresso incremental:** não é tudo ou nada. Começar com Pass Scoring (implementação 1-2 semanas) gera ROI imediato. Sub-behaviors é evolução natural dos Skills existentes.

3. **Pesquisa validada:** estratégias 1-3 já estão em uso em competição (não teoria). Estratégia 4 foi validada em SPL (conceitos transferem para SSL).

## Onde Aplicar (Carrossel Caipira)

**Curto prazo (próximos 2-3 meses):**
- Pass Scoring com ML incremental → módulo `Strategy`
- Multi-armed bandit para Plays → sistema de orquestração

**Médio prazo (6-12 meses):**
- RL para path planning → treinar em rSoccer + sim-to-real no `Control`
- Explorar papers de sub-behaviors para evolução dos Skills

**Longo prazo (1-2 anos):**
- Arquitetura híbrida (Skills → agentes RL semi-autônomos)
- Pesquisa em MARL descentralizado (se houver tempo + GPU disponível)

---

**Última atualização:** 2026-07-04  
**Mantém Fernando Hiroshi Murusaki atualizado sobre o estado da arte em RL/IA para SSL**
