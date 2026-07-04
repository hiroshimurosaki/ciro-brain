---
tipo: projeto
status: abandonado
---

# Projetos Abandonados / Shelved

Arquivo centralizado de projetos iniciados mas não concluídos ou pausados indefinidamente.

---

## Shelved — Tracker de Séries de TV

**Stack:** Next.js 16 + React 19 + TypeScript | Tailwind CSS 4 + shadcn/ui + Lucide + Framer Motion | Supabase (PostgreSQL) | TMDB API

**O que era:**  
App para acompanhar progresso de séries de TV, avaliar episódios/temporadas, descobrir novos shows. Estrutura completa:
- Autenticação (Supabase Auth: email/senha + Google OAuth)
- Rotas protegidas: home, show, discover, upcoming, person
- Banco de dados: Users, Series, Seasons, Episodes, Avaliations (series/seasons/episodes)
- UI robusta com shadcn/ui (Dialog, Popover, Select, Calendar, etc.)
- CRUD centralizado em `crud_client.ts`

**Por que parou:**  
Não registrado explicitamente. Provável: escopo amplo com autenticação + integração TMDB + gerenciamento de estado complexo. Prioridades mudaram ou complexidade de coordenar features (revalidação ISR 43200s, múltiplas tabelas de avaliação).

**Última atividade:** 2025-04-01

**Como retomar:**  
- Projeto estruturado, componentes bem organizados
- Faltam: testes, Docker, migrações de banco versionadas
- TMDB já integrada (wrapper em `lib/tmdb/api_tmdb.ts`)
- Autenticação e rotas já funcionam

---

## Notas

- Shelved está em `/dev/Shelved/` com git history preservada
- Para retomar qualquer projeto: ler CLAUDE.md (se existir), revisar últimos commits, validar dependências
