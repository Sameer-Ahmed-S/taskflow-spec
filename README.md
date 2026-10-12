# taskflow-spec

**Single source of truth** for TaskFlow. All implementation repos derive their work from this spec — changes start here, then fan out per repo (ADR-002).

| Repo | Role |
|------|------|
| **taskflow-spec** (this) | Requirements, API contracts, ADRs |
| [taskflow-db](https://github.com/Sameer-Ahmed-S/taskflow-db) | Postgres schema + migrations |
| [taskflow-backend](https://github.com/Sameer-Ahmed-S/taskflow-backend) | Express API implementing the contract |
| [taskflow-frontend](https://github.com/Sameer-Ahmed-S/taskflow-frontend) | React SPA consuming the contract |

## Contents

- [`requirements.md`](./requirements.md) — entities + functional/non-functional requirements
- [`contracts/tasks-api.yaml`](./contracts/tasks-api.yaml) — OpenAPI contract between frontend and backend
- [`decisions.md`](./decisions.md) — architecture decision records

## Change process (SDD)

1. Update the spec here (requirements + contract + ADR if needed).
2. `taskflow-db`: migration for schema changes.
3. `taskflow-backend`: implement the updated contract.
4. `taskflow-frontend`: consume the updated contract.

Each step is its own PR in its own repo, referencing the spec commit it implements.
