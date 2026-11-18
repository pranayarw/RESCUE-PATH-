# RESQPATH

An AI-powered Global Road Emergency & Safety Assistance Platform that helps users get fast help during road emergencies with live maps, SOS, AI chat, and community alerts.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080, served at `/api`)
- `pnpm --filter @workspace/resqpath run dev` — run the frontend (served at `/`)
- `pnpm --filter @workspace/api-server run seed` — seed DB with emergency services + admin user
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string, `SESSION_SECRET` — JWT signing key

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React 19 + Vite, Tailwind CSS v4, shadcn/ui, wouter, TanStack Query, react-leaflet (OpenStreetMap), recharts, qrcode.react
- API: Express 5, bcryptjs, jsonwebtoken
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec → React Query hooks + Zod schemas)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — source-of-truth API contract (OpenAPI 3.1)
- `lib/api-zod/src/generated/` — generated Zod schemas from spec
- `lib/api-client-react/src/generated/` — generated React Query hooks from spec
- `lib/db/src/schema/` — Drizzle table definitions (users, safety-profiles, emergency-services, incidents, community-alerts, chat-messages)
- `artifacts/api-server/src/routes/` — Express route handlers (auth, users, emergency-services, incidents, community-alerts, chat, dashboard, admin)
- `artifacts/api-server/src/middlewares/auth.ts` — JWT auth middleware + `requireAdmin`
- `artifacts/resqpath/src/pages/` — all UI pages

## Architecture decisions

- Contract-first API: OpenAPI spec → Orval codegen → type-safe React Query hooks. Never hand-write fetch calls.
- JWT tokens stored in `localStorage` (keys: `resqpath_token`, `resqpath_user`); `setAuthTokenGetter` wires them into the generated `customFetch` in `main.tsx`.
- All routes under `/api` are proxied by the Replit shared reverse proxy; frontend calls use relative paths.
- Dark-by-default theme (CSS custom properties), togglable via localStorage; Leaflet tiles inverted via CSS filter in dark mode.
- AI chat is a keyword-matching engine (no external API key needed) with extensible `EMERGENCY_RESPONSES` map.

## Product

- **Login / 3-step Register** with blood group, emergency contacts, vehicle info
- **Dashboard** — live safety score, nearby services count, emergency numbers, community alerts snapshot, SOS shortcut
- **Live Map** — full-screen react-leaflet with 20 seeded Bangalore services, filter by type (hospital/police/ambulance/towing/fuel/tyre_repair), map/list toggle
- **SOS** — pulsing button with 5-second cancellable countdown, creates incident, shows emergency call buttons
- **AI Chat** — RESQ AI assistant with first-aid/CPR/accident guidance, language selector, quick suggestions
- **Incidents** — view/create/resolve personal emergency history
- **Community Alerts** — report road hazards, upvote others' alerts
- **Profile** — user info, safety profile editor, downloadable emergency QR card
- **Admin Panel** — charts (pie + bar), users table, incidents table, platform stats

## Admin credentials (seeded)

- Email: `admin@resqpath.com`
- Password: `admin123`

## User preferences

- Dark default theme with emergency red (#ef4444) primary and blue secondary
- Emergency-first design: SOS always accessible, emergency numbers always visible

## Gotchas

- After schema changes: run `pnpm --filter @workspace/db run push` then `pnpm run typecheck:libs`
- After OpenAPI spec changes: run `pnpm --filter @workspace/api-spec run codegen`
- Re-seed only runs if tables are empty (idempotent check)
- The `@workspace/db` package must be built (`pnpm run typecheck:libs`) before the api-server typecheck passes

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
