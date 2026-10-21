# Orchestration Runbook

How the main session runs a development pass. Agents do the work; the main
session only schedules them, verifies the pass state, and merges at the end.

The main session is the **orchestrator**. It is the only actor allowed to push to
`main`. The PM, dev, and QA subagents never merge, never close issues, and never
push to `main`.

## Actors and limits

| Actor | Parallelism | Per-pass limit | Trigger |
| --- | --- | --- | --- |
| PM | 1 at a time | 5 issues groomed | `status:backlog` exists |
| Dev | 2 spawned, one spawn at a time | 2 issues | `status:ready` with satisfied dependencies |
| QA | 2 at a time | one verdict per dev handoff | issue reaches `status:ready-for-test` |
| Orchestrator | n/a | 1 merge phase | dev and QA phases have settled |

## State model

| Label | Meaning | Set by |
| --- | --- | --- |
| `status:backlog` | Not groomed | creation |
| `status:ready` | Groomed, or QA failed; waiting for a dev | PM pass, QA failure |
| `status:in-progress` | Claimed by a dev | dev |
| `status:ready-for-test` | Branch pushed, waiting for QA | dev |
| `status:testing` | Claimed by QA | QA |
| `status:ready-for-merge` | Verified, waiting for the orchestrator | QA |
| `status:blocked` | Waiting on a decision, dependency, or conflict | any agent |

Labels are the only state. A pass can be interrupted at any point and resumed by
re-reading GitHub; the orchestrator keeps no memory between passes.

## Preflight

Run once before the pass starts:

```bash
gh auth status
git switch main
git status --short          # must be empty
git pull --ff-only
git worktree list           # note existing worktrees
```

Stop the pass if `main` is dirty, if there are unmerged commits from a previous
pass, or if `gh` is not authenticated. Do not "fix" a dirty tree by stashing or
resetting; report it.

Record the pass start: open issue counts by status.

## Phase 1 — PM (serial, max 5)

1. Read the count of open `status:backlog` issues. If zero, skip the phase.
2. Spawn **one** `pm` subagent with an explicit issue number: the lowest-numbered
   open `status:backlog` issue.
3. Wait for it to finish. Read its report.
4. Repeat until either 5 issues have been groomed or the PM reports
   `nothing-to-do`.

Rules:

- Never spawn two PM runs. Grooming mutates issue bodies and the overlap scan
  depends on a stable view of the tracker.
- Do not retry a `blocked` PM result in the same pass.
- A groomed issue is `status:ready`; a parked one is `status:blocked`.
- If the PM returns `Result: skipped` twice in a row, stop the phase and report —
  the tracker is inconsistent.

## Phase 2 — Dev selection

Build the candidate set:

1. Open issues labelled `status:ready`.
2. For each, read `Blocked by` from the body. Every referenced issue must be
   **closed** (merged). If any is open, the candidate is not eligible this pass.
3. Read the `Files expected to change` section and the `Parallel-safe` note.
4. Drop any candidate whose section is missing, or whose `Parallel-safe` is `no`.

Then pick at most **two** candidates that are disjoint:

- neither lists the other in `Blocked by` / `Blocks`, transitively
- their file lists do not intersect
- neither touches `package.json`, `package-lock.json`, `tsconfig.base.json`,
  `AGENTS.md`, or `.gitignore`
- both are eligible per steps 1–4

If only one qualifies, run one. If none qualify, skip the phase.

## Phase 3 — Dev and QA handoff

Spawn devs **one after the other**, each with an explicit issue number:

1. Spawn dev for candidate A. Wait until it has claimed the issue
   (`status:in-progress`) or returned.
2. If it implemented the issue, spawn QA for A **immediately** — do not wait for
   candidate B. QA may run while B is being developed.
3. Spawn dev for candidate B only if A was spawned and the two remain disjoint.
4. Do not spawn a third dev in this pass.

QA rules:

- At most **two** QA runs at once. If two are already running, queue the next
  handoff until one finishes.
- QA tests the branch it was handed. Never re-run QA in the same pass; a failure
  returns the issue to `status:ready` for the **next** pass.
- If QA reports `blocked`, leave it blocked and report it. Do not fix it.
- If QA reports `passed`, the issue is `status:ready-for-merge` and needs no
  further action this pass.

## Phase 4 — Merge

Only after every dev and QA run in this pass has settled. The orchestrator merges;
no subagent does.

Process `status:ready-for-merge` issues in **ascending issue number**:

```bash
git switch main
git pull --ff-only
git merge --no-ff issue/<n>-<slug> -m "merge: #<n> <title>"
npm ci
npm run typecheck
npm test
git push
gh issue close <n> --reason completed
git worktree remove .worktrees/<n> --force
git branch -d issue/<n>-<slug>
```

Rules:

- Run the checks **after each merge**, before pushing. Two individually verified
  branches can still break each other.
- If checks fail: `git reset --hard ORIG_HEAD`, comment on the issue with the
  failing output, set `status:blocked`, leave the branch intact, and continue with
  the next issue.
- If the merge conflicts: `git merge --abort`, comment with the conflicting paths,
  set `status:blocked`, leave the branch intact, and continue. A conflict means
  the parallel-safety check was wrong — record it in the pass report.
- Do not force-push, squash, or rebase. A rebased branch loses its QA verdict and
  must return to `status:ready-for-test`.
- Do not merge anything that is not `status:ready-for-merge`. No exceptions for
  "obviously fine" work.

## Stop rules

- **One pass, then stop.** After the merge phase, report and wait for the next
  instruction.
- **Never restart a pass automatically.** If the PM grooms 2 of 5, or a dev
  blocks, or a QA fails, the pass continues with what it has and ends normally.
  The next pass starts from labels, not from the last pass's plan.
- **A merge failure ends the merge phase.** Do not retry the merge, do not
  re-run QA, do not spawn a dev to fix the conflict in the same pass.
- **Never merge mid-pass.** Merging before all QA verdicts land invalidates the
  branches still in flight.
- A human merging to `main` while a pass is running invalidates the branches. If
  that happens, stop the pass and report; do not rebase agent branches silently.

## Pass report

End every pass with:

```
Pass: <start time> to <end time>
PM:      <n> groomed, <n> parked, <n> remaining backlog
Dev:     #<n> implemented, #<n> blocked
QA:      #<n> passed, #<n> failed, #<n> blocked
Merged:  #<n>, #<n>
Blocked: #<n> <reason>
Next pass ready: <yes | no — reason>
```

## What the orchestrator must never do

- Edit source code or issue bodies itself. It schedules and merges, nothing else.
- Bypass a label transition to make progress.
- Reassign an issue that is `status:in-progress`, `status:testing`, or
  `status:ready-for-merge`.
- Spawn a dev for an issue with unsatisfied dependencies.
- Run more than one PM, more than two devs, or more than two QAs at once.
- Merge with a failing test suite.
