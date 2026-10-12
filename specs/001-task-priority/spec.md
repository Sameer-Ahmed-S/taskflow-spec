# Feature 001: Task Priority

**Status**: specified · clarified
**Feature branch**: `feature/task-priority`
**Constitution**: `.specify/memory/constitution.md` (spec-first, contract-first, change order)

## User Story
As a TaskFlow user, I want to assign a priority (low / medium / high) to each task,
so that I can see at a glance which tasks matter most.

## Requirements

### Functional
- FR-7: Every task has a `priority`: enum `low | medium | high`, default `medium`.
- FR-8: Users can set priority when creating a task (optional; defaults to `medium`).
- FR-9: Users can change a task's priority at any time.
- FR-10: The UI displays a colored priority badge on each task card
  (high = red, medium = amber, low = gray).

### Clarifications (from /speckit.clarify)
- **Q: Can priority be null/unset?** A: No. Always one of the three values; existing rows get `medium` via migration default.
- **Q: Does priority affect list ordering?** A: No. Ordering stays `created_at DESC` (FR-2 unchanged).
- **Q: Can tasks be filtered by priority?** A: Not in this feature. Out of scope.
- **Q: Is priority required in TaskInput?** A: No — optional in both create and update; server applies default.

### Non-Functional
- NFR-1 (existing) applies: invalid priority value → 400 with `{ "error": ... }`.

## Entity Changes

### Task (updated)
| Field | Type | Constraints |
|-------|------|-------------|
| ...existing fields unchanged... | | |
| priority | enum `low\|medium\|high` | NOT NULL, default `medium` |

## Acceptance Criteria
1. `POST /api/tasks` with `{"title":"x","priority":"high"}` returns 201 with `"priority":"high"`.
2. `POST /api/tasks` with `{"title":"x"}` returns 201 with `"priority":"medium"`.
3. `POST`/`PATCH` with `priority:"urgent"` returns 400.
4. `PATCH /api/tasks/:id` with `{"priority":"low"}` returns 200 with updated priority.
5. UI shows a priority badge on every card and a priority selector in the create form.
6. Existing tasks (pre-migration) show priority `medium`.

## Out of Scope
- Sorting/filtering by priority
- Priority in list query params
- Per-user priority defaults
