# ESMO Manager

Plataforma brasileira de gerenciamento de times de esports MOBA. Gerencie seu time, dispute partidas ranqueadas, participe de campeonatos e domine o pick & ban com informações completas de campeões.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — rodar o servidor de API (porta 8080)
- `pnpm --filter @workspace/lol-manager run dev` — rodar o frontend (porta 19633)
- `pnpm run typecheck` — verificar tipos em todos os pacotes
- `pnpm run build` — build completo
- `pnpm --filter @workspace/api-spec run codegen` — regenerar hooks e schemas Zod do OpenAPI
- `pnpm --filter @workspace/db run push` — aplicar mudanças no schema do DB (dev apenas)
- Env necessária: `DATABASE_URL` — string de conexão PostgreSQL

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Validação: Zod (`zod/v4`), `drizzle-zod`
- Codegen API: Orval (a partir do spec OpenAPI)
- Build: esbuild (bundle CJS)
- Frontend: React + Vite + Tailwind CSS + shadcn/ui

## Onde as coisas ficam

- `artifacts/api-server/src/routes/` — rotas Express (manager, team, roster, matches, draft, champions, dashboard, championships)
- `artifacts/api-server/src/lib/` — lógica de negócio (simulation.ts, ddragon.ts, seed.ts, logger.ts)
- `artifacts/lol-manager/src/pages/` — todas as páginas do frontend
- `artifacts/lol-manager/src/components/` — componentes reutilizáveis (Layout.tsx)
- `lib/api-spec/openapi.yaml` — contrato OpenAPI (fonte da verdade)
- `lib/db/src/schema/` — schema do banco de dados Drizzle

## Arquitetura

- Contrato-primeiro: spec OpenAPI define todos os endpoints, Orval gera hooks React Query e schemas Zod
- Simulação de partidas no servidor com sistema de pontos de patente
- Dados de campeões carregados da API DDragon do LoL em tempo real
- Chat da equipe em memória por partida (chatStore no servidor)
- Seed automático na inicialização: times AI, agentes livres, campeonatos

## Product

- **Onboarding**: Criar gerente e time com nome e região
- **Dashboard**: Stats do time, classificação geral com patentes coloridas, partidas recentes
- **Elenco**: Gerenciar jogadores com stats (mecânica, decisão, teamwork, consistência), pool de campeões, contratar do mercado
- **Partidas**: Disputar amistosas (sem pontos), ranqueadas (±30 pts) e campeonatos (±50 pts) com resultado detalhado
- **Pick & Ban**: Tela de draft com imagens reais dos campeões, tier, tipo de ataque (físico/mágico/híbrido), vantagens e desvantagens
- **Campeões**: Browser com 172+ campeões, filtros por role/tier/tipo de ataque, painel de detalhes com matchups
- **Estratégia**: Configurar estilo de jogo, early/late game, prioridade no draft, gestão de minions e visão
- **Campeonatos**: Criar e se inscrever em campeonatos com taxa de entrada e premiação

## User preferences

- Toda a interface deve estar em português brasileiro
- Tema escuro gaming com dourado como cor primária e ciano como acento
- Patentes: Ferro, Bronze, Prata, Ouro, Platina, Diamante, Mestre, Grão-Mestre, Desafiante

## Gotchas

- O codegen do Orval regenera `lib/api-zod/src/index.ts` — não ter `schemas: { path: "generated/types" }` na config zod do orval.config.ts
- O servidor usa pino para logging; nunca usar console.log em código de servidor
- A API retorna 404 JSON quando o gerente não existe (comportamento correto, frontend exibe onboarding)
- Dados de campeões vêm da DDragon API do LoL — requer acesso à internet

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
