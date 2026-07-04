---
tipo: projeto
status: pausado
stack: Next.js 16, TypeScript, Tailwind CSS, shadcn/ui, Supabase (Prisma v7), TanStack Query
última_atualização: 2025-07-04
---

# WhoFed — Web App de Estatísticas para League of Legends

## O que é

Web app **mobile-first** de estatísticas gamificadas para League of Legends. Grupos de amigos comparam performance via quiz de múltipla escolha contra competidores. Interface interativa, cache inteligente da API da Riot (TTL 24h), deploy em Vercel.

## Stack

- **Frontend:** Next.js 16 (App Router) + TypeScript + Tailwind CSS + shadcn/ui + Framer Motion
- **Backend:** Supabase (PostgreSQL) com Prisma ORM v7
- **State:** TanStack Query (React Query)
- **API Externa:** Riot API (Account-V1, Summoner-V4, Match-V5, Champion-Mastery-V4)
- **Deploy:** Vercel

## Perguntas do quiz implementadas

1. Maior winrate
2. Mais mortes
3. Pior KDA
4. Monochamp (se houver)
5. Pior partida individual

## Perguntas planejadas (Fase 6 em desenvolvimento)

- O Elo Fraco (pior com campeão em comum)
- O Esquisito (bonecos impopulares)
- ARAM God

## Estado atual

- **Em pausa** — fase de planejamento de monetização (Google AdSense)
- Requisição de Production Key da Riot **aguardando aprovação**
- Site deve ser 100% gratuito (requisito Riot)
- Estrutura core: rotas de API (`/api/player`, `/api/compare`, `/api/search`), integração cache, quiz comparativo funcional

## Regras críticas

- **Cache da Riot API:** TTL 24h (tabelas: players, matches, masteries)
- **Prisma v7:** URLs no `prisma.config.ts`, não no schema; sempre rodar `prisma generate` após mudanças
- **Conexão DB:** Porta 6543 (Transaction mode) em produção; 5432 (Session mode) localmente
