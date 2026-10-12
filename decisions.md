# Architecture Decision Records

## ADR-001: Three repos, one spec
- `taskflow-db` — schema migrations + local Postgres (docker-compose). Owns the **data model**.
- `taskflow-backend` — Express API. Owns the **contract** (`spec/contracts/tasks-api.yaml` lives here as the served copy).
- `taskflow-frontend` — React SPA. **Consumes** the contract only; never talks to the DB.

## ADR-002: Contract-first changes
Any change touching more than one repo is made in this order:
1. Update spec (requirements + OpenAPI contract).
2. `taskflow-db`: migration for schema changes.
3. `taskflow-backend`: implement contract.
4. `taskflow-frontend`: consume contract.

## ADR-003: IDs and timestamps are server-side
Clients never send `id`, `created_at`, `updated_at`. Backend generates UUIDs (`gen_random_uuid()`), DB maintains timestamps.

## ADR-004: Ports
- DB: 5432 (docker-compose)
- Backend: 4000, API mounted at `/api`
- Frontend: 5173 (Vite dev server), calls backend via `VITE_API_URL` (default `http://localhost:4000/api`)
