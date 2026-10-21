# Task Template

Used by PMs and agents when grooming a backlog issue. Filling this in is the
grooming step; issues are created as plain titles with a one-line description.

## How to use

1. Copy the skeleton below into the issue body, or run
   `gh issue create --title "[ID] Name" --body-file _docs/TASK_TEMPLATE.md`
   and replace the skeleton.
2. Fill `Goal`, `Scope`, and `Acceptance criteria` at minimum.
3. Swap the label `status:backlog` for `status:ready` when it is groomed.
4. Anything you decide is out of scope becomes a **new issue**, not a deleted bullet.

---

## Skeleton (copy this)

```markdown
## Goal

## Scope
-

## Out of scope
-

## Acceptance criteria
- [ ]

## Dependencies
- Blocked by:
- Blocks:

## Notes
```

---

## Section guide

### Goal

*One or two sentences. State the outcome, not the activity.*

Answer: what changes for the household once this is done? If the sentence only
restates the title, it is not a goal.

> **Good:** A forgotten 6-digit PIN can be recovered without shell access, via an
> admin-issued single-use reset link.
>
> **Poor:** Implement PIN reset.

### Scope

*Bulleted list of what is included, described as concrete artefacts: an endpoint,
a table, a screen, a behaviour.*

Rule of thumb: a reviewer reading the bullets should be able to tell whether the
work is finished. Vague bullets become undiscussable acceptance criteria later.

> **Good:**
> - `POST /api/admin/users/:id/reset-link`, admin-only, returns the link instead of emailing it
> - `password_resets` table: `token_hash`, `user_id`, `expires_at`, `used_at`
> - Admin screen button "Reset PIN" that copies the link
>
> **Poor:**
> - PIN reset stuff
> - Backend changes

### Out of scope

*Bulleted list of adjacent work you are deliberately not doing.*

This is the section that prevents scope creep. Every item here is a candidate for
a future issue, so write it as work, not as a disclaimer.

> **Good:**
> - Sending the reset link by email — the admin copies it manually
> - Forced re-login of existing sessions after a reset
>
> **Poor:**
> - Not much, it's a small task

### Acceptance criteria

*Checkbox list of verifiable statements. Each one starts with a verb and names an
observable condition. Aim for 3–7.*

They must be checkable without reading the diff: by running a command, hitting an
endpoint, or looking at the screen.

> **Good:**
> - [ ] An expired or already-used token returns 400 and does not reveal whether the email exists
> - [ ] A successful reset invalidates every other outstanding token for that user
> - [ ] The new PIN is accepted at login immediately, and the old PIN is rejected
>
> **Poor:**
> - [ ] Reset works
> - [ ] Code is clean

### Dependencies

*Issue references that must land first, and issues this one unblocks.*

Use `#123` so GitHub links them, and name the blocking decision or migration when
that is the real constraint.

> **Blocked by:** #14 (migration 002), DEC-05 timezone policy
> **Blocks:** #31 (login screen), #47 (account settings)

### Notes

*Optional. Constraints, files worth touching, risks, spec links, open questions.*

Keep it short. A link plus two bullets beats a pasted design document. If an
unresolved question changes the shape of the work, put it here as a checkbox so
it cannot be silently ignored.

> - Spec: `_docs/project_specification.md` §3
> - Tokens stored as SHA-256 hashes, never in plaintext — mirrors the invite flow
> - [ ] Confirm reset links expire in 24 hours

---

## Worked example

A groomed `FND-09` issue:

```markdown
## Goal

Add the columns and tables the original spec omits, so recurrence, invite
lifecycle, household settings, and appointment status can be built without
follow-up migrations.

## Scope
- `tasks.series_id` (TEXT, nullable, references `tasks.id`) linking occurrences to a series
- `tasks.recurrence_until` (TEXT, nullable)
- `task_occurrences` table: `id`, `task_id`, `occurrence_date`, `completed_at`
- `invites.accepted_at` and `invites.token_hash` (replaces the raw token column)
- `app_settings` table (key/value) seeded with the household timezone
- `updated_at` on `users`, `tasks`, and `subtasks`
- `CANCELLED` added to the task status values

## Out of scope
- Backfilling existing data — no production database exists yet
- Changes to the migration runner itself (FND-07)
- Category normalization logic — that lives in application code (DEC-06)

## Acceptance criteria
- [ ] `npm run db:migrate` applies 001 then 002 cleanly against an empty database
- [ ] Re-running the migration is a no-op and exits 0
- [ ] Deleting a task cascades to its subtasks and occurrences
- [ ] `app_settings` contains a `timezone` row with an IANA value after first migration
- [ ] `npm run typecheck` and existing tests pass

## Dependencies

- Blocked by: FND-07 (migration runner)
- Blocks: TASK-07, AUTH-06, AUTH-09, FND-11

## Notes
- Spec: `_docs/project_specification.md` §3
- Occurrences are materialized rather than virtual (DEC-02)
- Store the timezone as an IANA name, e.g. `Asia/Kathmandu`
```

---

## What does not belong in an issue

- Estimates or story points — grooming adds those later if needed.
- Restating the whole spec — link to it.
- More than one deliverable — split it.
- "Improve", "refactor", "clean up" without a named target.
