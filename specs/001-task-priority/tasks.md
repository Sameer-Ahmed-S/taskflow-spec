# Tasks — Feature 001: Task Priority

**Plan**: [plan.md](./plan.md) · Execution order: db → backend → frontend (constitution Principle III)

## Phase A — taskflow-db
- [ ] T001 [db] Create `migrations/002_add_priority.sql` adding `priority TEXT NOT NULL DEFAULT 'medium' CHECK (priority IN ('low','medium','high'))` to `tasks`
- [ ] T002 [db] Open PR referencing this spec; merge after review

## Phase B — taskflow-backend (after T002)
- [ ] T003 [backend] `src/validate.js`: add `priority` to allowed fields; reject values outside `low|medium|high` with 400
- [ ] T004 [backend] `src/routes/tasks.js` POST: insert `priority` (default `medium`)
- [ ] T005 [backend] `src/routes/tasks.js` PATCH: allow `priority` updates
- [ ] T006 [backend] Verify acceptance criteria 1–4 locally with curl
- [ ] T007 [backend] Open PR referencing this spec; merge after review

## Phase C — taskflow-frontend (after T007)
- [ ] T008 [frontend] Create form: priority `<select>` (low/medium/high, default medium) sent in `createTask`
- [ ] T009 [frontend] Task card: colored priority badge (high=red, medium=amber, low=gray)
- [ ] T010 [frontend] Task card: priority editable via select (PATCH)
- [ ] T011 [frontend] `src/App.css`: badge styles
- [ ] T012 [frontend] Verify acceptance criterion 5–6 in browser
- [ ] T013 [frontend] Open PR referencing this spec; merge after review

## Phase D — close the loop (taskflow-spec)
- [ ] T014 [spec] End-to-end verify; check off all tasks; commit
