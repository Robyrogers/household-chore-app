# Decisions Log

Locked decisions that shape the schema, API, and UI. Anything not listed here is
still open and should be raised as a decision issue before implementation.

| ID | Decision | Choice |
|---|---|---|
| DEC-01 | Root admin bootstrap | CLI command, not an HTTP setup wizard |
| DEC-02 | Recurrence model | Materialize the next occurrence on completion |
| DEC-03 | Permissions | Shared calendar, own edits |
| DEC-04 | Task type semantics | Appointments are committed; time blocks are flexible |
| DEC-05 | Timezones | Store UTC, display in one household timezone |
| DEC-06 | Categories | Free text with autocomplete and normalization |
| DEC-07 | Dependencies | Agreed set approved; anything new needs a separate ask |

## DEC-01 — Root admin bootstrap

The first user is created with an interactive CLI command
(`npm run create-admin`). It refuses to run once any user exists. There is no
`POST /api/setup` endpoint and no setup wizard in the frontend; the app starts at
the login screen.

Consequences: no credentials in `.env`, no bootstrap endpoint to protect, and
deployment requires shell access once.

## DEC-02 — Recurrence model

Tasks carry `series_id`. Completing a recurring instance marks that occurrence
complete and inserts the next occurrence as a new row in the same series. No
scheduler or cron job is involved.

Consequences: completion history is preserved per occurrence; list queries stay
simple; the recurrence interval is stored once on the series definition.

## DEC-03 — Permissions

| Actor | Read | Create | Edit / delete | Complete / update status |
|---|---|---|---|---|
| Member (creator) | all tasks | yes | own tasks | own tasks |
| Member (assignee) | all tasks | yes | — | assigned tasks |
| Admin | all tasks | yes | all tasks | all tasks |

Only admins manage users and invites.

## DEC-04 — Task type semantics

`TIME_BLOCK` is work the household intends to do: movable, editable, may carry
subtasks. `APPOINTMENT` is a fixed external commitment: rendered as a locked band
in the calendar, no subtasks, status limited to `PENDING`, `COMPLETED`, or
`CANCELLED`, and it warns when it overlaps a time block. `DEADLINE` has only a due
datetime.

Consequences: `CANCELLED` is added to the status values; appointment validation
rejects subtasks; the calendar renders appointments differently from time blocks.

## DEC-05 — Timezones

All datetimes are stored as UTC ISO-8601 strings. The household timezone is stored
in `app_settings` and used for all rendering and for recurrence math. All-day items
store a local date plus `is_all_day = 1`. Device timezones are ignored.

Consequences: two users always see the same wall-clock times; recurring events do
not drift across DST; changing the household timezone changes display, not data.

## DEC-06 — Categories

`tasks.category` stays a free-text column. Values are trimmed and normalized to a
single casing on save. The UI suggests existing categories from
`SELECT DISTINCT category`.

Consequences: no category CRUD or admin screen; the spec schema is unchanged;
"Kitchen" and "kitchen" do not fragment filters.

## DEC-07 — Dependencies

The following are approved for use as each phase needs them:

- Shared/backend: `zod`, `@fastify/cookie`, `@fastify/cors`, `@fastify/static`, `tsx`
- Frontend: `react-router-dom`, `@tanstack/react-query`, `date-fns`, `date-fns-tz`,
  `clsx`, `tailwind-merge`, `lucide-react`
- Tooling (root only): `vitest`, `@playwright/test`, `eslint`, `prettier`,
  `concurrently`, `typescript` and type packages

Anything outside this list requires a separate approval request per `AGENTS.md`.

## Outstanding decisions

These surfaced during planning and are not yet resolved:

- PIN reset flow for users who forget their 6-digit PIN.
- Invite expiry window (proposed: 7 days).
- Recurrence behaviour across DST transitions and skipped holidays.
- Reminder and notification delivery.
- Data export and backup UX.
