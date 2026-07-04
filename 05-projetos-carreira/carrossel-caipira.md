---
tipo: projeto
titulo: Carrossel Caipira - Small Size League (SSL-EL)
status: ativo
data-inicio: 2024
area: robótica, IA, sistemas em tempo real
tags:
  - futebol de robô
  - IEEE Very Small Size Soccer
  - C++
  - ROS2
  - IA/agentes
reunioes: terça 19h (~8 pessoas)
github: https://github.com/carrossel-caipira/SSL-EL
---

# Carrossel Caipira — SSL-EL

## Descrição

**Carrossel Caipira** é o time de futebol de robôs da LTIA/Unicamp que compete no **IEEE Very Small Size Soccer (SSL)**, usando a categoria **SSL-EL (Entry Level)**. 

O projeto é uma implementação moderna e modular de um sistema completo de futebol robótico, desenvolvida em **C++** com **ROS2 (Humble)** como backbone de comunicação. Foca em código limpo, manutenível e acessível para futuras gerações do time.

**Fernando participa:** desenvolvimento de **IA e agentes autônomos** para decisão tática do time.

## Arquitetura do Sistema

### Flow Geral
```
SSL-Vision → Vision Module (Kalman filter)
             ↓
SSL GameController → Referee Module
             ↓
Strategy Module (STP) → Control Module → Communication
             ↓                             ↓
           Plays/Tactics/Skills      grSim ou Rádio
             ↓
         GUI (Qt) — debug e testes
```

### Módulos Principais

| Módulo | Responsabilidade | Tecnologia |
|--------|------------------|-----------|
| **Strategy** | STP (Skills, Tactics, Plays); máquina de estados; orquestração | C++ + ROS2 |
| **Control** | Cálculo de trajetória (STOx), velocidade e ângulo | Álgebra linear, cinemática |
| **Vision** | Recebe SSL-Vision, aplica Kalman filter, publica posições | Filtro de estado |
| **Referee** | Recebe SSL GameController, traduz para ROS2 topics | Protobuf |
| **Communication** | Conecta com robôs (rádio termios) ou grSim (simulador) | Sockets, terminais |
| **GUI** | Interface Qt para logs, visualização, testes em tempo real | Qt5 |

### Padrão STP (Skills, Tactics, Plays)

- **Skills**: primitivas de movimento (ex: `GoToPoint`, `OrientTo`, `Kick`)
- **Tactics**: combinações de Skills (ex: "receber passe + posicionar")
- **Plays**: estratégias de time (ex: "ataque coordenado com cobertura")

## Frentes Técnicas Identificadas

### 1. **Controle de Movimento (STOx Path Planning)**
- Algoritmo de planejamento de trajetória em tempo real
- Evita obstáculos (robôs aliados/adversários)
- Módulo `Control` — ponto de integração para RL de path planning

### 2. **Estratégia Tática (STP + IA)**
Estudadas 5 estratégias com IA aplicadas a SSL:

#### a. **Pass Scoring com ML Incremental** (em uso no TIGERs Mannheim)
- Avalia passes candidatos com heurísticas geométricas
- Ajusta weights das funções durante a partida (RL online)
- Prós: aprende com poucos dados, totalmente interpretável
- Ponto de integração: módulo `Strategy`

#### b. **RL para Path Planning** (validado em simulação por RobôCIn/UFPE)
- Agente RL treina desvio dinâmico de obstáculos
- 60% ganho de tempo vs. algoritmos clássicos
- Ponto de integração: módulo `Controller` + rSoccer (ambiente de treino)

#### c. **Adaptive Play Selection (Multi-Armed Bandit)**
- Seleciona Plays com base em taxa de sucesso na partida
- Extremamente leve (sem rede neural)
- Ponto de integração: sistema `Plays`

#### d. **MARL Descentralizado (MARLadona, ETH)** — Futuro distante
- Políticas emergentes de trabalho em equipe sem tática explícita
- Validado em simulação (Isaac Lab) até 11v11
- Sim-to-real gap significativo — ainda não testado em hardware SSL

#### e. **RL + Stack Clássica (sub-behaviors)** — Híbrido promissor
- 4 sub-agentes RL (navegação, controle de bola, chute, defesa)
- Seleção heurística entre eles
- Validado em competição (SPL 2024)
- Estrutura análoga aos Skills do Carrossel

## Repositórios Relacionados

- **SSL-EL (main)**: ~/dev/SSL-EL
- **grSim**: ~/dev/grSim (simulador RoboCup SSL, forked para SSL-EL)
- **teste_SSL**: ~/dev/teste_SSL (testes locais)

## Tecnologias

- **Linguagem**: C++
- **Framework**: ROS2 Humble
- **Serialização**: Protocol Buffers
- **Interface**: Qt5
- **Compilação**: CMake
- **Simulação**: grSim
- **Datasets**: Papers RL em SSL (em ~/Documentos/Carrossel)

## Próximos Passos (Sugestões)

1. **Curto prazo**: integrar Pass Scoring com ML incremental ao módulo `Strategy`
2. **Médio prazo**: implementar Adaptive Play Selection (multi-armed bandit)
3. **Médio-longo**: treinar agente RL para path planning em rSoccer + sim-to-real
4. **Futuro**: arquitetura de sub-behaviors (Skills → agentes RL semi-autônomos)

## Referências

- ETDP 2025 — TIGERs Mannheim Pass Selection: ssl.robocup.org
- RobôCIn RL for Path Planning (2024): arxiv.org/abs/2404.15410
- MARLadona — MARL (ETH Zürich, 2024): arxiv.org/abs/2409.20326
- WisTex United — RL + Stack (SPL 2024): arxiv.org/abs/2412.09417

---

**Última atualização:** 2026-07-04
