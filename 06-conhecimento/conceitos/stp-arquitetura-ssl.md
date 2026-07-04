---
tipo: conhecimento
titulo: Arquitetura STP — Skills, Tactics, Plays em SSL
tags:
  - arquitetura
  - STP
  - SSL
  - futebol-robo
  - design-patterns
  - ROS2
fonte: "SSL-EL (Carrossel Caipira); ER-Force Framework; padrão de design internacional"
data: 2026-07-04
relacionado:
  - "[[05-projetos-carreira/carrossel-caipira]]"
  - "[[06-conhecimento/conceitos/rl-futebol-robo-ssl]]"
---

# STP — Arquitetura de Skills, Tactics, Plays

## Conceito

**STP** é o padrão de design dominante em futebol de robô (SSL). Decompõe o comportamento de um time em **três camadas hierárquicas** com responsabilidades bem definidas.

```
┌─────────────────────────────┐
│  Play (Estratégia de Time)  │  "ataque coordenado com cobertura"
│  Orquestra qual Tactic      │
│  cada robô deve executar     │
└────────┬────────────────────┘
         │
┌────────▼────────────────────┐
│ Tactic (Comportamento)      │  "receber passe + posicionar"
│ Combina Skills em           │  "defender perto da bola"
│ sequência lógica            │
└────────┬────────────────────┘
         │
┌────────▼────────────────────┐
│ Skill (Primitiva)           │  "ir para ponto (x,y)"
│ Comando atômico para        │  "orientar para ângulo θ"
│ um robô                     │  "chutar com potência P"
└─────────────────────────────┘
```

---

## Camadas Explicadas

### **Skill** (O Quê? — O mais concreto)

**Responsabilidade:** primitiva de **movimento ou ação** de um robô individual.

**Exemplos:**
- `GoToPoint(x, y)` — navegar até coordenada específica
- `OrientTo(angle)` — girar para ângulo θ
- `Kick(power)` — chutar com potência especificada
- `ReceiveBall()` — posicionar-se para receber passe
- `StayInRegion(zona)` — ficar em área definida

**Implementação:**
- Função C++ que recebe estado (posição, velocidade, bola) e retorna comando de movimento (vx, vy, ω)
- Feedback loop rápido (comunicação com firmware do robô, ~100 Hz)
- Sem conhecimento de outros robôs ou estratégia global
- Executar de forma independente

**No Carrossel:** módulo `Control` implementa Skills; cada uma é uma state machine pequena.

---

### **Tactic** (Como? — Sequência de comportamentos)

**Responsabilidade:** **combina Skills em sequências** para realizar um comportamento mais complexo que requer múltiplos passos.

**Exemplos:**
- `ReceiveAndPass`: GoToPoint → ReceiveBall → Kick (esperar bola → receber → passar)
- `DefendZone`: StayInRegion(zona) → OrientTowardsBall (ficar na zona e olhar para a bola)
- `DriveToGoal`: GoToPoint(perto do alvo) → OrientTo(angulo de chute) → Kick(potência alta)

**Máquina de estados:**
```
ReceiveAndPass:
  estado IDLE → espera bola chegar
  estado POSITION → GoToPoint até ponto de recepção
  estado CONTROL → controlar bola (mini-loops de ajuste)
  estado KICK → executar Kick quando alinhado
```

**No Carrossel:** módulo `Strategy` orquestra Tactics; cada tática é uma máquina de estados que chama Skills.

---

### **Play** (Por quê? — Estratégia de time)

**Responsabilidade:** **atribui Tácticas a cada robô** do time, definindo quem faz o quê em tempo real.

**Exemplos:**
- `AttackPlay`: robô 0 dribla para o gol, robô 1 se posiciona como apoio, robô 2 defende
- `DefensePlay`: robô 0 marca atacante, robô 1 fecha passagem central, robô 2 volante
- `CoordinatedPass`: robô 0 tem a bola, robô 1 se move para passe, robô 0 passa e segue movimento

**Decisão durante partida:**
- Play é selecionada baseada em **estado do jogo** (quem tem a bola, fase da partida, score)
- Pode ser estática (se time ativa SEMPRE a mesma play) ou **dinâmica** (muda a cada situação)

**Adaptação a adversário (RL entry point):**
- Play selection pode usar **multi-armed bandit** para aprender qual Play funciona melhor contra este adversário
- Se `AttackPlay` marcou 3 gols, aumentar probabilidade dela ser selecionada

**No Carrossel:** módulo `Strategy` também define Plays; implementa máquina de estados de time.

---

## Flow Implementação no Carrossel (ROS2)

```
1. Referee Node recebe SSL GameController
   ↓
2. Strategy Node (STP) recebe:
   - posições dos robôs (Vision Node)
   - estado do jogo (Referee Node)
   - bola
   ↓
3. Strategy decide: qual Play rodar?
   (Estado machine em nível de time)
   ↓
4. Play ativa Tácticas:
   robô 0 → Tactic A
   robô 1 → Tactic B
   robô 2 → Tactic C
   (Máquinas de estado para cada robô)
   ↓
5. Cada Tactic executa Skills:
   Tactic A chamando: GoToPoint → ReceiveBall → OrientTo → Kick
   (Feedback loop de movimento)
   ↓
6. Control Node recebe comandos dos Skills
   Calcula trajetória (STOx path planning)
   ↓
7. Communication Node envia velocity commands
   para robôs físicos via rádio
   ou para grSim (simulador)
```

---

## Vantagens da Arquitetura STP

✅ **Modularidade:** cada camada é independente; trocar um Skill não quebra Tactics  
✅ **Reusabilidade:** `GoToPoint` é usado em múltiplas Tácticas  
✅ **Interpretabilidade:** fácil entender por que o time fez X (qual Play? qual Tactic?)  
✅ **Escalabilidade:** adicionar novo Skill/Tactic/Play não exige refatoring massivo  
✅ **Testabilidade:** testar cada camada isoladamente é viável  
✅ **Adaptação iterativa:** trocar lógica de Play selection (ex: heurística → multi-armed bandit) é simples  

---

## STP + RL — Pontos de Integração

### Nível Skill
- **RL para path planning** → melhorar `GoToPoint` com agente RL que aprende desvio dinâmico
- **Treinamento:** rSoccer (simulador OpenAI Gym-compatible)

### Nível Tactic
- **RL para sequenciamento** → agente decide próxima ação na máquina de estados (ex: passar agora ou esperar?)
- **Treinamento:** simulador de campo completo

### Nível Play
- **Multi-armed bandit** → seleciona Play com base em taxa de sucesso na partida
- **Aprendizado:** online, durante a partida (zero overhead)

### Nível Híbrido (Sub-behaviors)
- **4 agentes RL independentes** (navegação, controle de bola, chute, defesa)
- **Seleção heurística** entre eles → evolução da máquina de estados de Play

---

## Exemplos em Competição

- **TIGERs Mannheim** (tetracampeão 2021-2025): STP clássica + Pass Scoring com ML incremental
- **ER-Force** (framework open source): STP referência para muitos times
- **Carrossel Caipira**: arquitetura STP em ROS2, Skills/Tactics bem estruturadas, Play selection ainda heurística

---

## Por Que Importa para Fernando

A arquitetura STP é o **ponto de ancoragem** para qualquer implementação de RL no Carrossel:

1. **Skills já existem** → ponto natural para inserir RL em path planning
2. **Tácticas são máquinas de estado** → fácil adicionar agente RL como seletor de próxima ação
3. **Play selection é determinística** → direto trocar por multi-armed bandit
4. **Estrutura do código já está lá** → apenas "plugar" RL nos pontos certos

Sem entender STP, RL no Carrossel seria "RL descontextualizado". Com STP clara, é "RL pragmático integrado ao fluxo existente".

---

**Última atualização:** 2026-07-04  
**Referência técnica para decisões de arquitetura no projeto Carrossel Caipira**
