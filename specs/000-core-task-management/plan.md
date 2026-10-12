# Plan — Feature 000: Core Task Management (backfilled)

**Spec**: [spec.md](./spec.md) · **Status**: implemented

## Summary
Base CRUD for tasks over a REST API, with a three-column status board UI.
As-built: Express 4 + better-sqlite3 backend, React 18 + Vite frontend, SQLite
migrations in taskflow-db.

## Technical Context (as built)
- taskflow-db: `migrations/001_init.sql` — tasks table, status CHECK, updated_at trigger
- taskflow-backend: `src/index.js` (Express app), `src/db.js` (SQLite + startup migration runner),
  `src/validate.js` (contract-shaped input validation), `src/routes/tasks.js` (CRUD)
- taskflow-frontend: `src/api.js` (contract client), `src/App.jsx` (board), `src/App.css`

## Contract
`contracts/tasks-api.yaml` v1.0.0 — five endpoints under `/api/tasks`.

## Per-Repo Implementation (constitution order)
1. taskflow-db — schema migration
2. taskflow-backend — validation + routes implementing the contract
3. taskflow-frontend — board UI consuming the contract

## Verification
Performed live: create/list/update/delete via curl (201/200/204), 400 on invalid input,
404 on unknown id, UI round-trip in browser at :5173.
