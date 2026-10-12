# Plan — Feature 001: Task Priority

**Spec**: [spec.md](./spec.md) · **Constitution**: spec-first, contract-first, change order db → backend → frontend

## Summary
Add `priority` (low/medium/high, default medium) to the Task entity across all three
implementation repos. Purely additive; no breaking contract change (new optional field).

## Technical Context
- Datastore: SQLite (embedded), backend applies migrations at startup (ADR-005)
- Backend: Express 4, better-sqlite3, validation in `src/validate.js`
- Frontend: React 18 + Vite, API client in `src/api.js`, board UI in `src/App.jsx`

## Contract Change (`contracts/tasks-api.yaml`)
Add to components:
- New schema `TaskPriority`: `type: string, enum: [low, medium, high]`
- `Task.properties.priority`: `$ref: TaskPriority` (added to `required`)
- `TaskInput.properties.priority`: `$ref: TaskPriority` (optional)

## Per-Repo Plan (constitution Principle III order)

### taskflow-db
- New migration `migrations/002_add_priority.sql`:
  `ALTER TABLE tasks ADD COLUMN priority TEXT NOT NULL DEFAULT 'medium' CHECK (priority IN ('low','medium','high'));`
- Existing rows automatically get `medium` (clarification Q1).

### taskflow-backend
- `src/validate.js`: add `priority` to allowed fields; validate enum membership.
- `src/routes/tasks.js`: include `priority` in INSERT (default `medium` when absent) and in PATCH's updatable fields.
- No new endpoints; response shapes gain `priority`.

### taskflow-frontend
- `src/App.jsx`: priority `<select>` in create form (default `medium`); colored badge
  on each card; priority included in status-change row (editable via a select on the card).
- `src/App.css`: `.badge.priority-high|medium|low` styles (red/amber/gray).
- `src/api.js`: no change needed (passes fields through).

## Risk / Compatibility
- Additive contract change → old clients keep working (they just don't send priority).
- Migration is additive with default → no data rewrite.
- Backend must tolerate absence of the column only until db migration merges — hence ordered PRs.

## Verification
Per acceptance criteria in spec.md, plus end-to-end: create a high-priority task in UI,
confirm it round-trips to SQLite with correct defaulting.
