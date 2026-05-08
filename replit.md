# ESMO MOBA

An esports team management game where the player acts as manager/coach: build a roster, draft LoL champions, configure tactics, and watch simulated matches.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080, proxied at /api)
- `pnpm --filter @workspace/esmo-moba run dev` — run the frontend (port varies, proxied at /)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string, `SESSION_SECRET`

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite + Tailwind CSS + shadcn/ui + Wouter (routing)
- API: Express 5 (port 8080, proxied under /api)
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/esmo-moba/` — React+Vite frontend
- `artifacts/api-server/` — Express 5 backend
- `lib/db/src/schema/` — Drizzle ORM schema (source of truth for DB)
- `lib/api-spec/openapi.yaml` — OpenAPI spec (source of truth for API contract)
- `lib/api-client-react/src/generated/` — auto-generated React Query hooks
- `lib/api-zod/src/generated/` — auto-generated Zod schemas
- `artifacts/api-server/src/engine/simulator.ts` — match simulation engine
- `artifacts/api-server/src/lib/championService.ts` — Data Dragon integration (172 LoL champions)

## Architecture decisions

- Contract-first API: OpenAPI spec → Orval codegen → typed React Query hooks
- `lib/api-zod/src/index.ts` MUST only export `from "./generated/api"` — orval keeps regenerating both exports; the codegen script in api-spec/package.json auto-fixes this
- Match simulation uses real player attributes (precision, reaction, map reading, etc.) vs opponent's average rating
- Champions synced from Data Dragon API v16.x — 172 champions with icons, positions, meta stats
- Club localStorage key: `"esmo_club_id"` — validated on mount (rejects AI clubs, forces re-onboarding)
- AI club validation: App.tsx checks `isAi` field on load; if true, clears localStorage and redirects to onboarding

## Product

- **Onboarding**: Create org name, manager alias, region, financial mode
- **Command Center (Dashboard)**: Real-time stats — balance, payroll, matches, manager XP; roster overview; upcoming matches
- **Roster**: Starting lineup by position
- **Transfer Market**: Browse 50+ free agents with search/filter; sign players with 4-week upfront fee
- **Champions**: Full LoL champion library (172) with splash art, meta stats, ability details, power curves
- **Competitions**: Join Iron/Bronze leagues and cups; round-robin and elimination formats
- **Matches**: Draft phase (ban/pick from 172 champions), match simulation with timeline
- **Talent Tree**: Manager skill upgrades using talent points
- **Advance Week**: Deducts salaries, processes training, progresses game state

## Seed Data

- `/api/game/seed` (POST) — seeds 5 AI clubs, 50 free agents, 172 champions, 3 competitions
- Run ONCE before onboarding (onboarding calls it automatically)
- AI clubs auto-join Iron League; player club joins manually via Competitions page or auto-joins at seed time

## User preferences

- Dark purple esports theme throughout
- All text in English (UI) with Portuguese field names in DB/API (legacy)
- TypeScript strict mode

## Gotchas

- `lib/api-zod/src/index.ts` MUST only have `export * from "./generated/api";` — fixed in codegen script
- `or()` from drizzle-orm must be imported separately (not in the main `@workspace/db` bundle)
- Dashboard API fields: `competicoesAtivas`, `proximasPartidas`, `salarioSemanal`, `semanaAtual`, `roster`
- Match simulate returns 404 if match not found, 400 if no players; clubeId comes from req.body
- `useAdvanceWeek` mutate: `{ id }` only (no `data` field)
- `useSimulateMatch` mutate: `{ id }` only (no `data` field)
- `useReleasePlayer` mutate: `{ id }` only (no `data` field)
- `useSeedDatabase` mutateAsync: no args (void)

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
