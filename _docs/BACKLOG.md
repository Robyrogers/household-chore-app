# Backlog

53 tasks across six phases, derived from `_docs/project_specification.md` and the
locked decisions in `_docs/DECISIONS.md`.

Each task is tracked as a GitHub issue titled `[ID] Name`. Grooming fills in goal,
scope, out-of-scope, and acceptance criteria using `_docs/TASK_TEMPLATE.md`.

## Phase 1 — Foundation & database

| ID | Task | Description |
|---|---|---|
| FND-01 | Monorepo scaffold | Root `package.json` with npm workspaces for `apps/api` and `apps/web`, base `tsconfig`, `.gitignore`, `.editorconfig`. |
| FND-02 | Workspace shells | Create both apps with their own `package.json`, scripts, and TypeScript config so each owns its dependencies. |
| FND-03 | Quality tooling | ESLint, Prettier, Vitest, Playwright, and `concurrently` installed at the root only. |
| FND-04 | Root script contract | Wire every root command documented in `AGENTS.md` so the docs stay accurate. |
| FND-05 | Env config | `.env.example` and validated environment loading that fails fast on missing secrets. |
| FND-06 | SQLite connection | `better-sqlite3` singleton with WAL mode, foreign keys on, and a busy timeout. |
| FND-07 | Migration runner | Versioned SQL migrations tracked in a `schema_migrations` table, safe to re-run. |
| FND-08 | Migration 001 | `users`, `invites`, `tasks`, and `subtasks` exactly as specified. |
| FND-09 | Migration 002 | Task series and occurrences, invite lifecycle, `app_settings`, `updated_at`, and the `CANCELLED` status. |
| FND-10 | Dev data | `db:seed` with a sample household and `db:reset` that drops and recreates the database. |
| FND-11 | Settings service | Typed read/write for `app_settings` with the household timezone seeded on first run. |

## Phase 2 — Auth & users

| ID | Task | Description |
|---|---|---|
| AUTH-01 | PIN utilities | Bcrypt hashing, 6-digit validation, and a weak-PIN blocklist shared by login and invite acceptance. |
| AUTH-02 | `create-admin` CLI | Interactive root-admin creation that refuses when any user already exists. |
| AUTH-03 | Login | Email plus PIN login issuing an HttpOnly JWT cookie, with generic error messages. |
| AUTH-04 | Rate limiting | Per-IP and per-email limits on login and invite acceptance, with lockout on repeated failures. |
| AUTH-05 | Session handling | Cookie/JWT verification middleware plus `GET /auth/me` and `POST /auth/logout`. |
| AUTH-06 | Invite creation | Admin-only invites with a hashed single-use token and a seven-day expiry. |
| AUTH-07 | Invite management | List, revoke, and resend invites, tracking `accepted_at` so tokens cannot be reused. |
| AUTH-08 | Mailer abstraction | Nodemailer over Gmail SMTP, falling back to logging the invite link when SMTP is unset. |
| AUTH-09 | Invite acceptance | Validate the token, collect a name, set the PIN, create the user, and invalidate the token atomically. |
| AUTH-10 | Auth hardening | Origin checks, security headers, and production-safe cookie flags. |
| AUTH-11 | Auth tests | Admin creation, login, lockout, and the full invite lifecycle via `fastify.inject`. |

## Phase 3 — Tasks & scheduling API

| ID | Task | Description |
|---|---|---|
| TASK-01 | Shared schemas | Zod schemas and DTOs for tasks, subtasks, invites, and settings shared by API and web. |
| TASK-02 | Task CRUD | Create, read, update, and delete with per-type validation: start and end for time blocks and appointments, due for deadlines. |
| TASK-03 | Status rules | Pending to in-progress to completed, `COMPLETED`/`CANCELLED` for appointments, subtasks rejected on appointments. |
| TASK-04 | Filtering | Filter by assignee, status, priority, category, type, and date range. |
| TASK-05 | Subtasks | Create, update, delete, toggle completion, and reorder subtasks with cascade delete. |
| TASK-06 | Recurrence engine | Pure `nextOccurrence` function covering hourly through yearly with intervals, unit tested. |
| TASK-07 | Recurrence integration | Link occurrences with `series_id` and insert the next instance when a recurring task completes. |
| TASK-08 | Calendar query | Range endpoint returning time blocks, appointments, and deadlines for a window. |
| TASK-09 | Authorization guard | Everyone reads, creators edit and delete, assignees complete, admin unrestricted. |
| TASK-10 | Task tests | Validation matrix, recurrence edge cases, and authorization denials. |

## Phase 4 — Frontend foundation

| ID | Task | Description |
|---|---|---|
| WEB-01 | App scaffold | Vite, React, TypeScript, and Tailwind with a dev proxy to the API. |
| WEB-02 | Routing and shell | Router, shared layout, navigation, auth guard, and redirect logic. |
| WEB-03 | API client | Typed fetch wrapper reusing the shared schemas and normalizing API errors. |
| WEB-04 | Server state | TanStack Query configuration with cache keys and invalidation for task and auth data. |
| WEB-05 | `PinPad` | Six-digit keypad with auto-submit, error shake, lockout messaging, and keyboard support. |
| WEB-06 | Auth screens | Login and invite-acceptance screens. |
| WEB-07 | UI primitives | Shared buttons, inputs, modal, badges, toasts, and empty, loading, and error states. |

## Phase 5 — Frontend features

| ID | Task | Description |
|---|---|---|
| FEAT-01 | Task modal | Create and edit modal with the three timing modes and conditional fields. |
| FEAT-02 | Recurrence picker | Type, interval, and end condition controls with a plain-language summary. |
| FEAT-03 | Task list | Grouped tasks with filters, quick complete, and priority and category chips. |
| FEAT-04 | Subtask checklist | Inline creation, toggling, reordering, and progress display. |
| FEAT-05 | Calendar views | Week and day timelines plus a month grid, with deadlines visually distinct from time blocks. |
| FEAT-06 | Admin invites | Dashboard to send, resend, revoke, and copy invite links. |
| FEAT-07 | Settings | Users change their own PIN and name; admins edit the household timezone and members. |
| FEAT-08 | Appointment band | Locked calendar bands that cannot be dragged and warn when they overlap a time block. |

## Phase 6 — Test, harden, deploy

| ID | Task | Description |
|---|---|---|
| OPS-01 | E2E flows | Playwright coverage of admin creation, invite, PIN, task creation, and completing a recurring instance. |
| OPS-02 | Security pass | Review rate limits, cookie flags, and token hashing; run a dependency audit. |
| OPS-03 | Container build | Multi-stage Dockerfile and compose file with a database volume and healthcheck. |
| OPS-04 | Bare-metal deploy | systemd unit plus backup and restore scripts. |
| OPS-05 | Docs | API contract, decisions log, runbook, and upgrade procedure. |
| OPS-06 | Resource check | Measure memory and CPU on Pi-class hardware under seeded load. |

## Deferred

Not tracked as issues yet; record them here until they are promoted to tasks.

- PIN reset for users who forget their 6-digit PIN.
- Invite expiry confirmation (seven-day default).
- Recurrence behaviour across DST transitions and skipped holidays.
- Reminder and notification delivery.
- Data export and backup UX.
