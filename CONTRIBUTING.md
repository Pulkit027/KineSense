# Contributing

## Module ownership

Each module in this repo has a clear owner, which keeps parallel work from colliding:

| Module | Scope | Typical branch prefix |
|---|---|---|
| `ml/` | MediaPipe integration, landmark extraction, normalization, ISL dataset prep, sign classifier training | `feature/ml-*` |
| `server/` | Sign chaining logic, internal alert dispatch, WebSocket relay, auth | `feature/server-*` |
| `mirror/` | Elder-facing camera UI, confidence ring, need cards, Reassurance Drawer | `feature/mirror-*` |
| `dashboard/` | Family-facing alert pop-ups, check-ins, drift cards | `feature/dashboard-*` |

`mirror/` and `dashboard/` are both frontend apps and sometimes get touched together —
pick a sub-boundary (e.g. by component file, not by line) before editing both in the
same change.

## The two contracts — do not change without updating both sides

These are the only interfaces that cross module boundaries:

1. **Landmark → sign** (`ml/` → `mirror/`): fixed-shape landmark array in, `{ sign, confidence }` out.
2. **Alert event** (`server/` ↔ `mirror/`, `dashboard/`): `{ sign, timestamp, confidence }` over the `alert` socket event.

Both are documented in `DEVELOPMENT.md`. If you need to change either shape, update
all affected sides in the same change — a silent one-sided change breaks the other
module without any compile-time warning.

## Workflow

1. Branch off latest `main`:
   ```
   git checkout main && git pull && git checkout -b feature/<module>-<thing>
   ```
2. Commit small and often — a 400-line diff is unreviewable and conflict-prone.
3. Before opening a PR, rebase on latest `main` (not merge):
   ```
   git fetch origin && git rebase origin/main
   ```
4. PR into `main`; at least one reviewer pass — catches contract breaks early.
5. Delete your branch after merge.

## Lockfiles

Each folder (`mirror/`, `dashboard/`, `server/`) has its own `package-lock.json`.
Only run `npm install` inside the folder you're actually touching — don't run it
from the repo root, and don't hand-edit a lockfile. If two branches add deps to the
same folder in parallel, the lockfile is the one file where "just take one side" is
fine to resolve conflicts with (`npm install` regenerates it cleanly either way).

## If you do hit a conflict

Whoever touches the shared contract file last resolves it — don't let git's
auto-merge silently pick one side of a landmark-shape or alert-shape change.
Re-read `DEVELOPMENT.md`'s contract section before resolving.
