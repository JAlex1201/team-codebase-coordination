# PROJECT_LOG.md

Canonical coordination file for this repository. Every agent reads this first and updates it as it works. Append-only where noted. Never edit or delete another agent's entries. On a merge conflict here, keep all entries from both sides and order by timestamp.

---

## Context

The shared mental model a new agent needs before working: what this repo is, the
main components and where they live, key conventions, and any constraints. Keep this
curated and current. When something here becomes wrong, supersede it with a dated note
rather than deleting it, so the history stays intact.

- (example) Stack: Python service in `src/`, React frontend in `web/`.
- (example) All API responses use the `ApiResponse<T>` wrapper. See `src/types/`.

---

## Decisions

Dated, attributed records of choices that are hard to reverse or that affect how other
people's code must behave. Newest at top. One block per decision.

### 0002 | YYYY-MM-DD | handle | accepted
- **Decision:** what was decided, in a sentence.
- **Alternatives:** the other viable options and why they lost.
- **Consequence:** what this commits the team to.

### 0001 | YYYY-MM-DD | handle | accepted
- **Decision:** ...
- **Alternatives:** ...
- **Consequence:** ...

---

## Activity log

Append-only. Newest at top. One entry per unit of work. Update your own entry's status
as it progresses. Do not touch anyone else's.

### YYYY-MM-DD HH:MM | handle | in progress
- Task: short description.
- Branch: `handle/task-slug`.
- Files: main paths being touched.
- Depends on: PR number, or none.

### YYYY-MM-DD HH:MM | handle | done (PR #NN)
- Task: short description.
- Outcome: what shipped.
