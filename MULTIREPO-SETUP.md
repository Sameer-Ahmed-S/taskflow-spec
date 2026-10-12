# TaskFlow Multi-Repo — From-Scratch Setup with GitHub spec-kit (VS Code)

Complete recipe to recreate the four-repo SDD setup: a dedicated spec repo plus
one repo per implementation concern. Companion docs: [SDD-WORKFLOW.md](./SDD-WORKFLOW.md)
(per-feature playbook) and [MONOREPO-SETUP.md](https://github.com/Sameer-Ahmed-S/taskflow/blob/main/MONOREPO-SETUP.md)
(single-repo variant).

> **Convention used throughout:** each speckit step shows the slash command first.
> If it doesn't autocomplete, use the **fallback**: type the plain-English form
> *"Use the speckit-\<name\> skill to …"* in Agent mode. The full fallback
> walkthrough is in Step 7 — the same pattern applies to every later step.

## Repos

| Repo | Role |
|------|------|
| taskflow-spec | **Single source of truth**: constitution, requirements, contracts, per-feature specs |
| taskflow-db | SQLite schema + migrations |
| taskflow-backend | Express API implementing the contract |
| taskflow-frontend | React SPA consuming the contract |

---

## Step 0 — Prerequisites (one-time)

```powershell
winget install --id Python.Python.3.13 --silent
# restart the terminal, then:
pip install specify-cli
specify version    # verify
```

Requires: VS Code + GitHub Copilot Chat extension (signed in), git, gh CLI, Node.js.

## Step 1 — Create the four repos + local folders

```powershell
mkdir C:\dev\taskflow-demo; cd C:\dev\taskflow-demo
gh repo create <you>/taskflow-spec     --public --clone
gh repo create <you>/taskflow-db       --public --clone
gh repo create <you>/taskflow-backend  --public --clone
gh repo create <you>/taskflow-frontend --public --clone
```

## Step 2 — Bootstrap spec-kit in EVERY repo

```powershell
foreach ($r in 'taskflow-spec','taskflow-db','taskflow-backend','taskflow-frontend') {
  cd C:\dev\taskflow-demo\$r
  specify init --here --force --non-interactive --integration copilot
}
```

Each repo gets its own `.specify/` + `.github/skills/` so speckit skills work in
whichever repo's VS Code window you're in.

## Step 3 — Write the constitution (spec repo only)

Edit `C:\dev\taskflow-demo\taskflow-spec\.specify\memory\constitution.md`.
Minimum principles: spec-first (non-negotiable), contract-first
(`contracts/tasks-api.yaml`), **multi-repo change order** (spec → db → backend →
frontend, one PR per repo), traceability (every PR references the spec commit),
server-side IDs/timestamps.

## Step 4 — Seed the spec repo

Create in `taskflow-spec`:
- `requirements.md` — entities + functional/non-functional requirements
- `contracts/tasks-api.yaml` — the OpenAPI contract
- `decisions.md` — ADRs (repo split, change order, server-side fields, datastore)

```powershell
cd C:\dev\taskflow-demo\taskflow-spec
git add -A; git commit -m "Initial spec: requirements, contract, ADRs, constitution"
git push -u origin main
```

## Step 5 — Scaffold the implementation repos

Create the base code in each repo (backend: Express app + validation + routes;
frontend: Vite+React board; db: `migrations/001_init.sql`). Commit and push each.

> Tip: in a real run you'd generate this from feature 000 via the pipeline
> (Steps 6–10) instead of hand-scaffolding.

## Step 6 — Open the SPEC repo in VS Code

`File → Open Folder → C:\dev\taskflow-demo\taskflow-spec`, then Copilot Chat
(**Ctrl+Alt+I**), switch to **Agent mode** (dropdown at the bottom of chat input).
Agent mode is required — Ask mode cannot create files or run scripts.

> The spec-phase agent works from **artifacts** (requirements, contracts, ADRs,
> constitution) — it doesn't need the other repos' code. Code-level impact
> analysis happens per repo during implementation.

## Step 7 — Specify the feature

```
/speckit.specify Add priority to tasks: low/medium/high, default medium.
Users can set priority at creation, change it later, and see it in the UI.
```

### Fallback (no autocomplete) — detailed walkthrough

Skills are instruction files under `.github/skills/speckit-*/SKILL.md`; the agent
loads one when your message names it. To trigger `speckit-specify` without a
slash command:

1. Confirm you're in **Agent mode** (Step 6).
2. Type exactly:
   ```
   Use the speckit-specify skill to create a feature spec for: Add priority to
   tasks: low/medium/high, default medium. Users can set priority at creation,
   change it later, and see it in the UI.
   ```
3. **Approve the permission prompts** as they appear:
   | Prompt | What it does | Action |
   |---|---|---|
   | Run `create-new-feature.ps1`? | spec-kit script creating branch `001-<name>` + folder `specs/001-<name>/` | Allow |
   | Write `spec.md`? | Generates the spec from the template | Allow |

   (Dropdown on **Allow** → "Always allow" reduces prompts for the session.)
4. **Verify the output** in Explorer: `specs/001-task-priority/spec.md` exists;
   `git status` shows a new branch + untracked files. If the agent only
   *described* a spec without creating files, reply:
   **"You haven't created the files — follow the skill's procedure and write them now."**

### Troubleshooting
- "No such skill" → check `.github/skills/speckit-specify/SKILL.md` exists in the
  opened folder and you opened the repo ROOT.
- No prompts, nothing happens → you're in Ask mode; switch to Agent mode.
- Skills need a recent VS Code (1.105+) — check Help → About and update.

## Step 8 — Clarify → Plan → Tasks (same chat, spec repo)

```
/speckit.clarify
```
Fallback: `Use the speckit-clarify skill to review specs/001-task-priority/spec.md`

```
/speckit.plan Multi-repo: taskflow-db gets a migration, taskflow-backend implements
the contract, taskflow-frontend consumes it. Update contracts/tasks-api.yaml.
Respect the constitution's change order.
```
Fallback: `Use the speckit-plan skill to create the implementation plan. Multi-repo: db migration, backend implements contract, frontend consumes. Respect the constitution's change order.`

```
/speckit.tasks Group tasks by repo: [db], [backend], [frontend]. Respect dependency order.
```
Fallback: `Use the speckit-tasks skill to generate tasks grouped by repo: [db], [backend], [frontend]`

## Step 9 — Spec PR (review gate)

```powershell
cd C:\dev\taskflow-demo\taskflow-spec
git add -A; git commit -m "Spec 001: <feature>"
git push -u origin <feature-branch>
gh pr create --repo <you>/taskflow-spec --title "Spec 001: <feature>"
# review → merge → note the merge commit SHA
```

## Step 10 — Sync the spec snapshot into each implementation repo

Each repo carries a pinned copy of the feature folder it implements:

```powershell
foreach ($r in 'taskflow-db','taskflow-backend','taskflow-frontend') {
  cd C:\dev\taskflow-demo\$r
  git checkout -b feature/<name>
  robocopy ..\taskflow-spec\specs\001-<name> specs\001-<name> /E
  git add -A; git commit -m "Sync spec 001 from taskflow-spec@<sha>"
}
```

## Step 11 — Implement per repo (in change order: db → backend → frontend)

For each repo, open it in VS Code (`File → Open Folder`), Agent mode, then:

```
/speckit.implement Execute only the [db] tasks from specs/001-task-priority/tasks.md
```
Fallback: `Use the speckit-implement skill to execute only the [db] tasks from specs/001-task-priority/tasks.md`

(replacing `[db]` with `[backend]`, then `[frontend]` in that repo's window)

After each repo's tasks pass verification:

```powershell
git add -A; git commit -m "Implement spec 001 [db] tasks"
git push -u origin feature/<name>
gh pr create --title "Spec 001: <feature> (db)" --body "Implements taskflow-spec specs/001-<name> @ <sha>"
# review → merge, THEN proceed to the next repo
```

## Step 12 — Close the loop + verify

```powershell
cd C:\dev\taskflow-demo\taskflow-spec
# check off completed tasks in specs/001-<name>/tasks.md
git commit -am "Mark 001 implemented: db#X backend#Y frontend#Z"; git push

cd ..\taskflow-backend;  npm install; npm run dev   # :4000 (auto-migrates ../taskflow-db)
cd ..\taskflow-frontend; npm install; npm run dev   # :5173
```

---

## Key differences from the monorepo setup

| Aspect | Monorepo | This multi-repo setup |
|---|---|---|
| `specify init` | once, at root | once per repo (4×) |
| Spec distribution | everyone reads `spec/` directly | snapshot sync per repo (Step 10) |
| PRs per feature | 1 spanning folders | 4 (spec + 3 implementations), merged in order |
| Agent context | full codebase | artifacts at the seams; own repo's code at implementation |
| VS Code windows | one | one per repo during implementation |
