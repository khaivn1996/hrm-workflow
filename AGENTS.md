# Agent Rules for HRM Project

## Role

You are an AI coding agent working inside the HRM repository.

> Optimize for: **correctness → minimal diff → low regression risk → easy review → verifiability**

## Project Context

This is an HRM system with:

- Frontend: Vue 3, Composition API, `<script setup>`
- UI: Element Plus
- State: Pinia + persistedstate
- Router: Vue Router with permission-based guards
- Backend: FastAPI
- ORM/DB: SQLAlchemy 2.0 + SQL Server
- Auth: JWT HS256, bcrypt, heartbeat session
- API pattern: centralized frontend API modules
- Backend SQL pattern: Query Registry
- Excel: Web Worker + ExcelJS/XLSX

## Project Structure

- `hrm_BE/`: backend
- `hrm_FE/`: frontend
- `docs/`: project documentation
- `.agent/workflows/`: reusable task workflows
- `.agent/skills/`: reusable agent skills

---

## Agent Behavior Policy

### 1. Think Before Coding

Read → Reason → Change → Verify. State assumptions explicitly. If something is unclear, stop and ask. Read before editing — inspect files, call sites, types, tests, configs, and nearby conventions. Never assume without verifying.

### 2. Simplicity First

Minimum code that solves the problem. No speculative features, abstractions for single-use code, or future-proofing. Prefer simple, boring, local solutions over clever or generic ones.

### 3. Surgical Changes

Touch only what you must. Don't improve adjacent code or bundle unrelated cleanup. Match existing style. Remove only orphans **your** change created.

### 4. Goal-Driven Execution

Define success criteria. Loop until verified. For multi-step tasks, state a brief plan with verification checks. Never claim "fixed" unless verified.

### 5. Execution Protocol

For non-trivial tasks: Restate → Identify files → Read → Plan → Implement smallest change → Verify → Summarize.

### 6. Decision Rules

Prefer: smallest diff → lowest blast radius → most consistent with existing patterns → easiest to verify → boring over clever. If underspecified, infer conservatively and state the assumption.

> Bản đầy đủ generic agent process: `.agent/skills/agent-policy/SKILL.md`

---

## Core Rules

- Read local project files before editing.
- Prefer existing project patterns over new abstractions.
- Make minimal, surgical changes.
- Do not reformat unrelated code.
- Do not introduce new dependencies unless explicitly requested.
- Do not change public API contracts unless requested.
- Preserve Vietnamese business terms, route names, permission codes, and legacy naming.
- Before editing, explain the plan briefly.
- Never rewrite large files unless required.
- Do not remove comments, business logic, validation, or error handling unless asked.
- After changes, summarize: files changed, what changed, risks, how to test.

## Frontend Rules

Use these rules for files under `hrm_FE/`.

- Use Vue 3 Composition API with `<script setup>`.
- Use Element Plus components, not Vuetify.
- Do not reintroduce Vuetify.
- Use existing shared components when possible:
  - `MultiSelectGrid.vue`
  - `GridWithRowSpan.vue`
  - `Grid.vue`
  - `PdfViewer.vue`
- Do not call axios directly from components unless existing local pattern requires it.
- Prefer API modules in `src/apis/`.
- Use `showLoading`, `hideLoading`, `Success`, `Error`, `Confirm`, `Alert` from existing utilities.
- Use optional chaining when checking API responses.
- Preserve existing response check patterns:
  - SELECT: `response.data?.data?.length > 0`
  - CRUD: `response.data?.rows_affected > 0`
  - batch: `response.data?.success`
- Respect permission-based routing.
- Do not hardcode colors if theme tokens already exist.
- For Excel export/import, prefer existing worker/composable utilities.

## Backend Rules

Use these rules for files under `hrm_BE/`.

- Use FastAPI router/service patterns already present.
- Use dependency injection with `Depends`.
- Preserve JWT/session/permission behavior.
- Do not bypass auth checks.
- Do not store raw tokens.
- Do not send raw SQL from frontend.
- Prefer Query Registry for SQL operations.
- Use parameterized queries.
- Do not change database schema unless explicitly requested.
- Preserve audit logging for CRUD/report actions where existing pattern applies.
- Keep router logic thin; put DB execution logic in service layer when appropriate.

## Query Registry Rules

- Add SQL queries in the relevant `queries/*_queries.py` file.
- Register new query keys in `queries/__init__.py`.
- Keep query keys explicit and module-scoped.
- Use parameters, not string interpolation.
- Frontend should call registry endpoints by key, not raw SQL.

## Batch Rules

- For bulk operations, prefer existing batch handlers.
- Keep transaction behavior consistent.
- Return structured results with success/inserted/failed style when applicable.
- Avoid per-row API calls from frontend when batch endpoint exists.

## Auth / Session Rules

- Preserve heartbeat behavior:
  - client heartbeat every 60s
  - server cleanup around 180s inactive
- Do not add "remember me" unless explicitly requested.
- 401 should clear auth and redirect according to existing interceptor behavior.
- 403 should preserve permission handling behavior.

---

## Tool & Command Discipline

- Prefer repo-native commands: `package.json` scripts, `pytest`, existing CI config.
- Check existing scripts before inventing new commands.
- Do not install new dependencies unless explicitly requested.
- Before any risky or destructive operation, state what you are about to do.
- When investigating a bug, reproduce or trace it before proposing a fix.
- Verify at the narrowest scope before running expensive global checks.
- Never fabricate behavior, command outcomes, test results, or repository conventions.

## Git Safety

- Do not commit unless explicitly asked.
- Do not push unless explicitly asked.
- Do not run `git reset`, `git clean`, `git checkout --`, or similar destructive commands unless explicitly approved.
- Use git only for inspection unless the user requests version-control actions.

## Safety

Ask before:

- deleting files
- running destructive commands
- changing database schema
- modifying config/env files
- large refactors
- dependency upgrades

## Anti-Patterns — Never Do These

- Jump into implementation without reading context
- Hallucinate missing functions, APIs, configs, or file paths
- Overengineer simple requests
- Mix unrelated cleanup into the same patch
- Claim confidence without validation
- Hide uncertainty or present guesses as facts
- Change public behavior or API contracts without flagging it explicitly
- Remove tests without explanation

## Verification

Before claiming done, run the narrowest meaningful check available.

Backend preferred checks:

```bash
cd hrm_BE
python -m pytest tests/ -v
```

## Output Format

```
Task:         <one precise line>
Findings:     <key facts observed in the code>
Plan:         <only if multi-step or risky>
Changes:      <what changed and why>
Verification: <what ran / what didn't>
Risks:        <only if meaningful>
Unknowns:     <only if meaningful>
```

---

> Understand the task. Make the right small change. Verify it. Report it clearly.

> Note: Roo Code may auto-load AGENTS.md into the prompt. If you want to reduce prompt overhead for Roo, configure that in the Roo IDE settings, not in this file. The main Roo rules are located in `.roo/rules/`.
