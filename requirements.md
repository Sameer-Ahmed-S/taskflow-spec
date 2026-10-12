# TaskFlow — Requirements (Single Source of Truth)

## Overview
TaskFlow is a minimal task management product: users create, view, update, complete, and delete tasks.

This spec is **authoritative**. All three repos (`taskflow-db`, `taskflow-backend`, `taskflow-frontend`) implement against it. Changes start here, then fan out per repo.

## Entities

### Task
| Field       | Type      | Constraints                                   |
|-------------|-----------|-----------------------------------------------|
| id          | UUID      | PK, server-generated                          |
| title       | string    | required, 1–200 chars                         |
| description | string    | optional, max 2000 chars                      |
| status      | enum      | `todo` \| `in_progress` \| `done`, default `todo` |
| created_at  | timestamp | server-generated                              |
| updated_at  | timestamp | server-maintained                             |

## Functional Requirements
- FR-1: Create a task with title (required) and optional description/status.
- FR-2: List all tasks, newest first; optional filter by `status`.
- FR-3: Get a single task by id.
- FR-4: Update title, description, and/or status of a task.
- FR-5: Delete a task.
- FR-6: UI shows tasks grouped by status columns (todo / in_progress / done).
- FR-7..FR-10: task priority (see [specs/001-task-priority](./specs/001-task-priority/spec.md)).

## Non-Functional
- NFR-1: API validates all input per `contracts/tasks-api.yaml`; invalid input → 400 with error body.
- NFR-2: Unknown task id → 404.
- NFR-3: SQLite (embedded, file-based) is the datastore. See ADR-005.
