# Feature 000: Core Task Management

**Status**: implemented (backfilled — authored retroactively to complete artifact history)
**Implemented by**: initial scaffold commits — spec `a55f5df`, db `84146bb`, backend `06c12e4`, frontend `2528193`
**Constitution**: `.specify/memory/constitution.md`

## User Story
As a TaskFlow user, I want to create, view, update, complete, and delete tasks on a
status board, so that I can track my work.

## Requirements

### Functional
- FR-1: Create a task with title (required) and optional description/status.
- FR-2: List all tasks, newest first; optional filter by `status`.
- FR-3: Get a single task by id.
- FR-4: Update title, description, and/or status of a task.
- FR-5: Delete a task.
- FR-6: UI shows tasks grouped by status columns (todo / in_progress / done).

### Non-Functional
- NFR-1: API validates all input per `contracts/tasks-api.yaml`; invalid input → 400 with `{ "error": ... }`.
- NFR-2: Unknown task id → 404.
- NFR-3: SQLite (embedded, file-based) is the datastore (ADR-005).

## Entity

### Task
| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | PK, server-generated |
| title | string | required, 1–200 chars |
| description | string | optional, max 2000 chars |
| status | enum `todo\|in_progress\|done` | default `todo` |
| created_at | timestamp | server-generated |
| updated_at | timestamp | server-maintained |

## Acceptance Criteria
1. `POST /api/tasks` with valid title → 201 with server-generated id/timestamps, status `todo`.
2. `GET /api/tasks` → 200, newest first; `?status=done` filters.
3. `GET /api/tasks/:id` → 200 or 404.
4. `PATCH /api/tasks/:id` partial update → 200; `updated_at` advances.
5. `DELETE /api/tasks/:id` → 204; second delete → 404.
6. Empty title, unknown field, or bad status → 400 with error body.
7. Board UI renders three status columns; create form, status select, delete button work.

## Notes
This feature predates spec-kit adoption (see SDD-WORKFLOW.md). It was driven by
`requirements.md` + `contracts/tasks-api.yaml` directly; this file completes the
per-feature artifact trail. Datastore was later amended from Postgres to SQLite (ADR-005).
