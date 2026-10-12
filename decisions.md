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
- Backend: 4000, API mounted at `/api`
- Frontend: 5173 (Vite dev server), calls backend via `VITE_API_URL` (default `http://localhost:4000/api`)

## ADR-005: SQLite instead of Postgres
The local demo runs without Docker, so the datastore is **SQLite** (embedded, file at `taskflow-db/taskflow.db`).
Consequences:
- `taskflow-db` holds the schema as plain `.sql` migrations; no server, no docker-compose.
- The **backend applies migrations at startup** and generates UUIDs in the app layer (still server-side per ADR-003).
- Enum becomes `TEXT CHECK (...)`, timestamps become ISO-8601 `TEXT`.
- If we later move to Postgres, only `taskflow-db` migrations and the backend's data layer change; the contract is unaffected.
