---
tipo: projeto
status: pausado
stack: Python, Riot API (match-v5), python-pptx, Chart.js, Pillow
última_atualização: 2025-05-31
---

# flex-analyser — Análise Estatística de League of Legends (Grupo Flex)

## O que é

Análise detalhista de estatísticas League of Legends (fila Flex, servidor BR) de um grupo de 8 amigos. Pipeline de coleta → processamento → entrega em PowerPoint (33 slides) e dashboard HTML interativo.

Objetivo: **apresentação estratégica + zoação calibrada** sobre o grupo, com rigor estatístico (IC95%, incerteza sempre visível).

## Os 8 jogadores

| Nick (slide) | Riot ID | Role canônica |
|---|---|---|
| Hiroshi | Hiroshi#10102 | Selva |
| Qiak | Qiak#000 | Atirador |
| Yuya Freecss | Yuya Freecss#BR1 | Meio |
| Thokyru | Thokyru#BR1 | Suporte |
| Top Mogger | Top Mogger#Dasky | Topo |
| Nashorn | Nashorn#Shiro | Selva |
| Lukyy | Lukyy#Luky | Topo |
| Nyachi | Nyachi#mee | Atirador |

(Hiroshi é o usuário.)

## Stack

- **Coleta:** Riot API (match-v5, Account-V1, Summoner-V4)
- **Análise:** Python (numpy, json)
- **Apresentação:** python-pptx (gráficos nativos, não matplotlib)
- **Dashboard:** Chart.js (CDN), HTML inlinado
- **Renderização visual:** Pillow (ícones, heatmaps PNG)

## Pipeline

1. **Coleta (flex-analyser.py):** puxa ~100 partidas/jogador, filtra fila (só Summoner's Rift sério: Ranked, Flex, Draft, Clash). Cache em disco, rate-limiting. Output: `analise_flex.json`, `resumo_flex.txt`.

2. **Solo vs. Grupo (solo_vs_grupo.py):** separa partidas solo vs. em grupo (2+ do grupo), calcula Δ de percentil de impacto. Output: `solo_vs_grupo.json`.

3. **Matemática Discreta (discrete_stats.py):** extrai 146 campos/jogador + 128 desafios da API + timeline. Calcula: WR condicional (pós-vitória/derrota, hot/cold streak), WR por lado (blue/red), taxa FF, domínio de lane, qualidade de combate, objetivos, disciplina, heatmaps. Output: `discrete_flex.json`.

4. **Entregáveis:** 
   - **Deck PowerPoint** (33 slides, tema navy+dourado, gráficos nativos, heatmaps, cards de comportamento)
   - **Dashboard HTML** (interativo, Chart.js, filtro por jogador, 3 visões de comparação, IC95)

## Achados principais (262 partidas, pós-filtro)

- **WRs:** Yuya 64.6%, Thokyru 60.0%, Top Mogger 60.0%, Hiroshi 57.8%, Nashorn 54.7%, Qiak 52.6%, Nyachi 50.0%, Lukyy 44.9%
- **Hiroshi = motor:** lidera em dano (56x top-1), ouro (47x), KP (50x); divide farm com Qiak
- **Nyachi cai em grupo:** farm −19.3, ouro −23.0, dano −15.3 (achado sólido, 94 jogos solo)
- **Lukyy WR real 44.9%** (não 34.9% — Arena afundava); Gnar dele 52.4% (21j); toda dupla com ele afunda
- **"Afoga no late" CONFIRMADO:** 78.7% (<20min) → 40.7% (35min+); 209 jogos no 35+
- **Thokyru = suporte clássico:** dano/farm baixos, rei da visão (36x top-1)

## Regra crítica: FILTRO DE FILAS

**EXCLUÍDOS:** Arena (CHERRY, queueId 1700/1750), ARAM (450/720), URF (900), Swiftplay (480).  
**INCLUÍDOS:** Ranked Solo (420), Flex (440), Draft Normal (400), Clash (700).  
Essa filtragem derrubou 156 de 418 partidas brutos (Arena = 33% do dataset). Sem filtro, estatísticas são contaminadas.

## Métodos de análise valorizados

1. **Rastrear efeito de segunda ordem** — não parar em "X perde de Y", perguntar cadeia
2. **Leitura caridosa** — perguntar "o que faria um bom jogador parecer assim?" antes de culpar
3. **Terminar toda stat numa alavanca** — não "Lukyy é Gnar-dependente", mas "protejam o early dele"

## Variáveis de ambiente

```bash
RIOT_API_KEY=RGAPI-xxx  # Dev key (expira 24h); rodar scripts com --from-cache para usar cache
```

## Como rodar

```bash
pip install requests python-pptx
RIOT_API_KEY=RGAPI-xxx python3 flex-analyser.py      # ~1h-1h30 (1ª vez)
python3 flex-analyser.py --from-cache                 # sem API, rápido
python3 solo_vs_grupo.py                              # segundos
python3 discrete_stats.py                             # segundos
python3 build_deck.py && python3 build_dashboard.py   # gera entregáveis
```

## Estado atual

- **Pausado** — pipeline completo (coleta, análise, entregáveis) operacional
- Último commit: 2025-05-31
- ⚠️ **matplotlib quebrado** (conflito numpy 1.x/2.x) — gráficos nativos python-pptx no deck, Chart.js no dashboard
