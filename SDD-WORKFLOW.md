# TaskFlow SDD Workflow — Approach B (spec-kit in every repo)

Step-by-step playbook for running a cross-repo feature ("task priority") through the
specify → clarify → plan → tasks → implement pipeline.

## Repos

| Repo | Role |
|------|------|
| [taskflow-spec](https://github.com/Sameer-Ahmed-S/taskflow-spec) | Single source of truth: requirements, contracts, ADRs, feature specs |
| [taskflow-db](https://github.com/Sameer-Ahmed-S/taskflow-db) | Postgres schema + migrations |
| [taskflow-backend](https://github.com/Sameer-Ahmed-S/taskflow-backend) | Express API implementing the contract |
| [taskflow-frontend](https://github.com/Sameer-Ahmed-S/taskflow-frontend) | React SPA consuming the contract |

Local root: `C:\Users\sameerahmed.s\taskflow-demo\`

---

## Phase 0 — One-time setup

### 0.1 Install spec-kit CLI
```powershell
pip install specify-cli    # (or: uv tool install specify-cli)
```

### 0.2 Bootstrap spec-kit in all four repos (run once per repo)
```powershell
cd C:\Users\sameerahmed.s\taskflow-demo\spec
specify init --here --ai copilot --force
git add -A; git commit -m "Bootstrap spec-kit"; git push

cd C:\Users\sameerahmed.s\taskflow-demo\taskflow-db
specify init --here --ai copilot --force
git add -A; git commit -m "Bootstrap spec-kit"; git push

cd C:\Users\sameerahmed.s\taskflow-demo\taskflow-backend
specify init --here --ai copilot --force
git add -A; git commit -m "Bootstrap spec-kit"; git push

cd C:\Users\sameerahmed.s\taskflow-demo\taskflow-frontend
specify init --here --ai copilot --force
git add -A; git commit -m "Bootstrap spec-kit"; git push
```

### 0.3 Write the constitution
`.specify/memory/constitution.md` in `taskflow-spec`, encoding ADR-002:
spec-first; change order db → backend → frontend; every PR references the spec commit.

---

## Phase 1 — Specify (session on **taskflow-spec**)
```
/speckit.specify Add priority to tasks: low/medium/high, default medium.
Users can set priority at creation, change it later, and see it in the UI.
```
→ creates `specs/001-task-priority/spec.md`

## Phase 2 — Clarify (same session)
```
/speckit.clarify
```
→ resolves edge cases (can priority be null? sort order? filter by priority?), updates `spec.md`

## Phase 3 — Plan (same session)
```
/speckit.plan Multi-repo: db gets migration, backend implements contract, frontend consumes.
Update contracts/tasks-api.yaml. Respect constitution change order.
```
→ creates `plan.md`, updates `contracts/tasks-api.yaml`, data-model notes

## Phase 4 — Tasks (same session)
```
/speckit.tasks Group tasks by repo: [db], [backend], [frontend]. Respect dependency order.
```
→ creates `tasks.md` with repo-tagged task list

## Phase 5 — Spec PR (review gate)
```powershell
cd C:\Users\sameerahmed.s\taskflow-demo\spec
git checkout -b feature/task-priority
git add -A; git commit -m "Spec 001: task priority"
git push -u origin feature/task-priority
gh pr create --repo Sameer-Ahmed-S/taskflow-spec --title "Spec 001: task priority"
# review → merge
```

---

## Phase 6 — Sync spec into implementation repos

Per repo (db, backend, frontend), on a feature branch:
```powershell
cd C:\Users\sameerahmed.s\taskflow-demo\taskflow-db
git checkout -b feature/task-priority
robocopy ..\spec\specs\001-task-priority specs\001-task-priority /E   # pinned snapshot copy
git add -A; git commit -m "Sync spec 001 from taskflow-spec@<sha>"
```
Repeat for `taskflow-backend` and `taskflow-frontend`.

---

## Phase 7 — Implement per repo (3 child sessions, dependency order)

### 7.1 taskflow-db session
```
/speckit.implement Execute only [db] tasks from specs/001-task-priority/tasks.md
```
→ `migrations/002_add_priority.sql` → push → PR (review gate)

### 7.2 taskflow-backend session (after db PR merges)
```
/speckit.implement Execute only [backend] tasks from specs/001-task-priority/tasks.md
```
→ validation + routes update → push → PR (review gate)

### 7.3 taskflow-frontend session (after backend PR merges)
```
/speckit.implement Execute only [frontend] tasks from specs/001-task-priority/tasks.md
```
→ priority selector + badge → push → PR (review gate)

---

## Phase 8 — Close the loop in the spec repo
```powershell
cd C:\Users\sameerahmed.s\taskflow-demo\spec
# check off completed tasks in specs/001-task-priority/tasks.md
git commit -am "Mark 001 implemented: db#2 backend#3 frontend#4"
git push
```

## Phase 9 — End-to-end verify
```powershell
cd C:\Users\sameerahmed.s\taskflow-demo\taskflow-db
docker compose down -v; docker compose up -d
cd ..\taskflow-backend; npm install; npm run dev
cd ..\taskflow-frontend; npm install; npm run dev
curl -X POST http://localhost:4000/api/tasks -H "Content-Type: application/json" -d '{"title":"test","priority":"high"}'
```

---

## Review gates

| Gate | When | Who |
|------|------|-----|
| Spec PR merge | Phase 5 | You |
| db PR merge | Phase 7.1 | You |
| backend PR merge | Phase 7.2 | You |
| frontend PR merge | Phase 7.3 | You |

Everything else is mechanical.
