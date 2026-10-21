# AGENTS.md

Guidance for AI agents working in this repository.

## Project

Self-hosted household task & schedule manager.
Fastify + SQLite API, React + Vite web app. See `_docs/project_specification.md`.

## Repository layout

```
household-chore-app/
├── package.json      # workspace root: shared scripts + dev tooling only
├── apps/
│   ├── api/          # Fastify API — owns its own dependencies
│   └── web/          # React + Vite SPA — owns its own dependencies
└── _docs/            # specs, backlog, decisions, templates
```

## Docs

- `_docs/project_specification.md` — product specification
- `_docs/BACKLOG.md` — tracked tasks, mirroring the GitHub issues
- `_docs/DECISIONS.md` — locked decisions; do not contradict them silently
- `_docs/TASK_TEMPLATE.md` — grooming template for issues

## Prerequisites

- Node.js 20 LTS or newer
- npm 10 or newer (workspaces)

## Setup

```bash
npm install                  # from the repo root — installs all workspaces
cp .env.example .env         # then fill in JWT secret + SMTP settings
npm run db:migrate           # apply SQLite migrations
```

## Commands

Run from the repository root unless noted.

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start API and web app together |
| `npm run dev:api` | Start API only (http://localhost:3000) |
| `npm run dev:web` | Start web app only (http://localhost:5173) |
| `npm run build` | Build API and web for production |
| `npm run typecheck` | TypeScript check across workspaces |
| `npm run lint` | ESLint |
| `npm run format` | Prettier write |
| `npm test` | Unit tests (Vitest) |
| `npm run test:e2e` | Playwright end-to-end tests |
| `npm run db:migrate` | Apply migrations |
| `npm run db:seed` | Seed development data |
| `npm run db:reset` | Drop and recreate the local database |

Per-app work can also be done directly:

```bash
npm run <script> -w apps/api
npm run <script> -w apps/web
cd apps/api && npm run <script>
```

## Rules

1. **Do not add dependencies without asking first.** Before proposing one, state
   what it is for, why the existing stack cannot do it, its bundle/size cost, and
   the alternative you considered. Wait for approval before installing.
2. **Each app owns its dependencies.** Runtime dependencies for the API go in
   `apps/api/package.json`; runtime dependencies for the frontend go in
   `apps/web/package.json`. Never install an app dependency at the root.
   Use `npm install <pkg> -w apps/api` or `npm install <pkg> -w apps/web`.
3. **The root is a workspace hub, not an app.** The root `package.json` only
   declares the workspaces (`apps/api`, `apps/web`), shared dev tooling
   (lint, format, typecheck, Playwright), and scripts that delegate to the apps.
   Do not put runtime code or runtime dependencies there.
4. **Keep one lockfile.** Commit `package-lock.json` from the root only; never
   create `package-lock.json` inside `apps/api` or `apps/web`.
5. **Match existing patterns.** Follow the file structure, naming, and error
   handling already present in the surrounding code before introducing a new one.
6. **Don't touch the specs silently.** If a change contradicts
   `_docs/project_specification.md`, say so and ask before diverging.
7. **Verify before finishing.** Run `npm run typecheck` and the relevant tests;
   report anything you could not run.
