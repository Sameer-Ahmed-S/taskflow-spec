# Tasks — Feature 001: Task Priority

**Plan**: [plan.md](./plan.md) · Execution order: db → backend → frontend (constitution Principle III)

## Phase A — taskflow-db
- [x] T001 [db] Create `migrations/002_add_priority.sql` adding `priority TEXT NOT NULL DEFAULT 'medium' CHECK (priority IN ('low','medium','high'))` to `tasks`
- [x] T002 [db] Open PR referencing this spec; merge after review

## Phase B — taskflow-backend (after T002)
- [x] T003 [backend] `src/validate.js`: add `priority` to allowed fields; reject values outside `low|medium|high` with 400
- [x] T004 [backend] `src/routes/tasks.js` POST: insert `priority` (default `medium`)
- [x] T005 [backend] `src/routes/tasks.js` PATCH: allow `priority` updates
- [x] T006 [backend] Verify acceptance criteria 1–4 locally with curl
- [x] T007 [backend] Open PR referencing this spec; merge after review

## Phase C — taskflow-frontend (after T007)
- [x] T008 [frontend] Create form: priority `<select>` (low/medium/high, default medium) sent in `createTask`
- [x] T009 [frontend] Task card: colored priority badge (high=red, medium=amber, low=gray)
- [x] T010 [frontend] Task card: priority editable via select (PATCH)
- [x] T011 [frontend] `src/App.css`: badge styles
- [x] T012 [frontend] Verify acceptance criterion 5–6 in browser
- [x] T013 [frontend] Open PR referencing this spec; merge after review

## Phase D — close the loop (taskflow-spec)
- [x] T014 [spec] End-to-end verify; check off all tasks; commit
