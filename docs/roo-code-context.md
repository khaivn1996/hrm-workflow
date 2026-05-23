# Roo Code Context cho HRM

> Dung de dua cho ChatGPT khi can chinh `.roomodes` cua project HRM.
> Cap nhat: 2026-05-23.

## 1. Muc tieu cua bo mode Roo hien tai

Roo Code trong workspace nay duoc cau hinh de lam viec voi he thong HRM theo huong:

| Uu tien             | Y nghia                                                          |
| ------------------- | ---------------------------------------------------------------- |
| Correctness         | Doc code that, khong doan endpoint/query key/response shape      |
| Minimal diff        | Sua dung pham vi, khong refactor lan sang code khac              |
| Low regression risk | Bao toan public contract, permission, query key, worker protocol |
| Easy review         | Output ro file, flow, ly do, rui ro                              |
| Verifiability       | Moi thay doi can co cach kiem tra cu the                         |

Nguyen tac cot loi:

- Read -> Reason -> Change -> Verify.
- Read local project files before editing.
- Prefer existing project patterns over new abstractions.
- Do not introduce dependencies unless explicitly requested.
- Do not change database schema unless explicitly requested.
- Do not commit, push, reset, clean, checkout destructive commands unless user asks/approves.
- Preserve Vietnamese business terms, route names, permission codes, and legacy naming.
- Khi khong chac, phai noi ro can kiem tra them.

## 2. Workspace va Git

Workspace root:

```text
C:\Source\Gitea\HRM
```

Day la container folder, khong phai Git repo chinh:

| Path          | Vai tro                     |
| ------------- | --------------------------- |
| `hrm_FE/.git` | Git repo rieng cua frontend |
| `hrm_BE/.git` | Git repo rieng cua backend  |
| `HRM/.git`    | Khong ton tai               |

Quy tac quan trong:

- Khong rename, move, disable, delete, modify bat ky `.git` nao.
- Khong doi `.git` thanh `.git_disabled`.
- Khi can git status/diff, chay trong `hrm_FE/` hoac `hrm_BE/`, khong chay o root.

## 3. Project stack HRM

Frontend:

| Muc       | Gia tri                                                              |
| --------- | -------------------------------------------------------------------- |
| Framework | Vue 3                                                                |
| Syntax    | Composition API, `<script setup>`                                    |
| Build     | Vite                                                                 |
| UI        | Element Plus                                                         |
| State     | Pinia + persistedstate                                               |
| Router    | Vue Router with permission-based guards                              |
| API       | Centralized API modules in `hrm_FE/src/apis/`                        |
| Excel     | Web Worker + ExcelJS/XLSX                                            |
| Utilities | `showLoading`, `hideLoading`, `Success`, `Error`, `Confirm`, `Alert` |

Backend:

| Muc          | Gia tri                                               |
| ------------ | ----------------------------------------------------- |
| Framework    | FastAPI                                               |
| ORM/DB       | SQLAlchemy 2.0 + SQL Server via pyodbc                |
| Auth         | JWT HS256, bcrypt, heartbeat session                  |
| Architecture | Router -> Service Layer -> Query Registry             |
| SQL          | Query Registry in `hrm_BE/app/queries/`               |
| Batch        | Batch handlers + Batch Insert Registry                |
| Audit        | Preserve audit logging where existing pattern applies |

## 4. Cac file cau hinh Roo/Agent hien tai

| File/Folder                    | Vai tro                                                                     |
| ------------------------------ | --------------------------------------------------------------------------- |
| `.roomodes`                    | Workspace Roo custom modes, source of truth cho cac mode project            |
| `.roo/mcp.json`                | MCP servers Roo dung trong workspace                                        |
| `.roo/commands/autodocs.md`    | Roo slash command `/autodocs` cap nhat `docs/implements.md`                 |
| `.roo/rules-mode-writer/`      | XML instruction files cho mode tao/sua mode                                 |
| `.roo/rules-research/`         | XML instruction files cho mode research                                     |
| `.roo/rules-merge-resolver/`   | XML instruction files cho mode merge conflict                               |
| `.roo/rules-hrm-xml-refactor/` | XML instruction files cho mode XML -> Vue Grid config                       |
| `.rooignore`                   | File/folder Roo nen bo qua                                                  |
| `AGENTS.md`                    | Agent rules chinh cua workspace                                             |
| `.agent/AGENT.md`              | Ban agent rules ngan gon cho HRM                                            |
| `.agent/skills/`               | Agent skills: policy, SQL query, Vue component                              |
| `.agent/workflows/`            | Workflow tai su dung: autodocs, bugfix, newmodule, ref, vb2vue, refactorxml |

## 5. MCP servers

Tu `.roo/mcp.json`:

| Server       | Command                         | Muc dich                                 |
| ------------ | ------------------------------- | ---------------------------------------- |
| `context7`   | `npx -y @upstash/context7-mcp`  | Tra cuu docs/library khi can context moi |
| `playwright` | `npx -y @playwright/mcp@0.0.38` | Browser/UI automation khi can            |

Luu y:

- MCP co the can network qua `npx`.
- Khong dung MCP thay cho viec doc code local khi task lien quan implementation hien tai.

## 6. `.rooignore` hien tai

Roo dang ignore cac nhom sau:

| Nhom                 | Pattern chinh                                                   |
| -------------------- | --------------------------------------------------------------- |
| IDE/VCS              | `**/.git/**`, `.idea`, `.vscode`                                |
| Secret/env/cert      | `.env`, `.env.*`, `*.pem`, `*.key`, `*.crt`, `*.pfx`            |
| Python cache/venv    | `__pycache__`, `.pytest_cache`, `.venv`, `venv`, coverage       |
| Logs/local DB        | `logs`, `*.log`, `*.sqlite`, `*.db`                             |
| Frontend build/cache | `node_modules`, `dist`, `build`, `.vite`, `.cache`, coverage    |
| Generated/minified   | `*.map`, `*.min.js`, `*.min.css`                                |
| Lock files           | `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `bun.lockb` |
| Docker/local storage | `docker-data`, `qdrant_storage`                                 |
| Binary/data/docs     | `*.xlsx`, `*.xls`, `*.csv`, `*.pdf`, archives, `*.tmp`          |

## 7. `.roomodes` schema dang dung

`.roomodes` la YAML co top-level:

```yaml
customModes:
  - slug: example-mode
    name: Example Mode
    description: Short summary
    roleDefinition: |-
      ...
    whenToUse: |-
      ...
    groups:
      - read
      - edit
      - command
      - mcp
    source: project
```

Fields dang duoc dung:

| Field                | Bat buoc/khuyen nghi | Ghi chu                        |
| -------------------- | -------------------- | ------------------------------ |
| `slug`               | Bat buoc             | ID mode, can on dinh           |
| `name`               | Bat buoc             | Ten hien thi                   |
| `roleDefinition`     | Bat buoc             | Prompt chinh cua mode          |
| `groups`             | Bat buoc             | Tool permissions               |
| `description`        | Khuyen nghi          | Tom tat ngan                   |
| `whenToUse`          | Khuyen nghi          | Mo ta trigger cho Orchestrator |
| `customInstructions` | Tuy chon             | Dang co o `project-research`   |
| `source`             | Dang dung            | Hien la `project`              |

Precedence:

- Workspace `.roomodes` co uu tien cao hon global custom modes.
- Neu cung `slug` ton tai ca global va workspace, workspace entry thang.

## 8. Roo custom modes hien tai

Hien co 18 custom modes.

| Slug                   | Muc dich                                                | Groups                              |
| ---------------------- | ------------------------------------------------------- | ----------------------------------- |
| `merge-resolver`       | Resolve merge conflicts bang git history/PR context     | read, edit, command, mcp            |
| `mode-writer`          | Tao/sua custom modes voi validation                     | read, edit restricted, command, mcp |
| `research`             | Explore/analyze HRM codebase read-only                  | read, command, mcp                  |
| `project-research`     | Investigate codebase structure read-only                | read                                |
| `hrm-flow-researcher`  | Trace flow HRM FE -> API -> BE -> Query -> SQL          | read, edit, command, mcp            |
| `hrm-refactor-planner` | Lap plan refactor an toan, khong implement mac dinh     | read, edit, command, mcp            |
| `hrm-code-implementer` | Implement code changes nho, dung pattern HRM            | read, edit, command, mcp            |
| `hrm-code-reviewer`    | Review regressions, architecture consistency            | read, edit, command, mcp            |
| `hrm-sql-auditor`      | Audit Query Registry, SQL params, query-key integration | read, edit, command, mcp            |
| `hrm-docs-writer`      | Viet docs tieng Viet dua tren implementation that       | read, edit, command                 |
| `hrm-debug-specialist` | Diagnose/fix bugs FE/BE/SQL/deploy/worker               | read, edit, command, mcp            |
| `hrm-test-writer`      | Tao/update tests backend/API/auth/batch/frontend logic  | read, edit, command, mcp            |
| `hrm-merge-resolver`   | Resolve HRM merge conflicts, preserve both sides        | read, edit, command, mcp            |
| `hrm-release-checker`  | Check risk truoc deploy/release                         | read, edit, command, mcp            |
| `hrm-mode-writer`      | Tao/refine Roo modes rieng cho HRM workflows            | read, edit, command, mcp            |
| `hrm-ask`              | Tra loi/giai thich implementation HRM                   | read, edit, command, mcp            |
| `hrm-xml-refactor`     | Convert XML column definitions sang Vue Grid configs    | read, edit restricted, command, mcp |
| `hrm-quick-assistant`  | Ho tro nhanh, ngan, low-risk                            | read, command, mcp                  |

## 9. Chuc nang tung mode

### `merge-resolver`

- Dung cho PR conflict resolution.
- Input chinh: PR number nhu `#123`.
- Doc PR title/body, branch, git blame, commit history, diff.
- Resolve conflict dua tren intent cua tung side.
- Co the stage resolved files.

### `mode-writer`

- Dung khi tao custom mode moi hoac sua mode hien co.
- Biet workspace modes `.roomodes` va global modes.
- Uu tien roleDefinition ro rang, `whenToUse` tot, permissions dung.
- Co rule files rieng trong `.roo/rules-mode-writer/`.
- Edit restricted vao `.roomodes`, `.roo/*.xml`, `.yaml`.

### `research`

- Read-only research cho HRM.
- Trace feature flow va giai thich pattern.
- Khong edit file.
- Co rule files rieng trong `.roo/rules-research/`.

### `project-research`

- Read-only codebase investigation.
- Bat dau bang file structure va docs.
- Output can cite paths, line numbers, types, implementation, dependencies.
- Co `customInstructions` inline trong `.roomodes`.

### `hrm-flow-researcher`

- Trace flow that cua mot feature HRM:

```text
Vue component
-> API module
-> FastAPI router
-> service function
-> query registry / batch handler
-> SQL query or stored procedure
```

- Xac dinh files, functions, endpoints, query keys, permissions, stores, response shape, worker action neu co.
- Role noi read-only, khong modify/create/refactor.

### `hrm-refactor-planner`

- Lap plan refactor an toan truoc khi implement.
- Preserve behavior, API contracts, permissions, query keys, worker protocol.
- Dung cho large Vue components, composables, constants, API helper boundaries, backend router/service/query boundaries.
- Role noi khong edit/create va khong perform refactor unless explicitly asked.

### `hrm-code-implementer`

- Mode implementation chinh cho code changes.
- Read relevant files before editing.
- Minimal targeted changes.
- Khong guess query keys, API paths, permissions, payloads, response shapes.
- Frontend: API modules, loading/error/success utilities, Element Plus, Pinia permission.
- Backend: thin routers, service layer, Query Registry, parameter binding, auth dependencies.
- Query: add/register keys consistently, check duplicate, ensure payload params match SQL params.
- Worker: preserve `postMessage` action/payload protocol.

### `hrm-code-reviewer`

- Review actual changed files.
- Focus correctness/regression risk over style.
- Check Vue, Element Plus, Pinia auth/permission, API paths, Query Registry, response shape, transaction behavior, worker protocol.
- Output findings first, ordered by severity.

### `hrm-sql-auditor`

- Dung khi add/modify/review SQL query keys, Query Registry entries, SQL params, batch registry integration.
- Bat buoc doc relevant query file va `queries/__init__.py`.
- Khong approve query key neu chua verify registration.
- Check duplicate keys, naming, endpoint type, SQL parameter binding, frontend params/response handling, backend service/router integration.
- Query naming guidance:

| Prefix        | Y nghia          |
| ------------- | ---------------- |
| `LoadDS_`     | Load list        |
| `LoadInfo_`   | Load detail/info |
| `InsertInfo_` | Insert           |
| `UpdateInfo_` | Update           |
| `DeleteInfo_` | Delete           |
| `Check_`      | Validation/check |
| `Report_`     | Report           |

### `hrm-docs-writer`

- Viet docs tieng Viet dua tren code that.
- Khong invent query keys, endpoints, permission codes, response shapes, file names.
- Document FE -> API -> BE -> SQL flow.
- Include files/functions, endpoint paths, query keys, permissions, request/response shape, gotchas, suggested tests.
- Co the edit docs, khong modify code unless explicitly asked.

### `hrm-debug-specialist`

- Diagnose bugs across FE, BE, SQL, auth, deploy, worker.
- Start from symptom/evidence.
- Trace actual flow end-to-end.
- Rank possible causes if multiple.
- Check frontend state/API payload/response shape/loading/permission/interceptor.
- Check backend route/auth/schema/service/exception/response.
- Check SQL query key registration/params/procedure vs registry.
- Check batch transaction/row errors/response.
- Check worker action/payload/returned data.
- Check deployment env/CORS/ports/healthcheck/reverse proxy.

### `hrm-test-writer`

- Tao/update tests.
- Read implementation before tests.
- Follow existing pytest/frontend test style.
- Khong invent APIs, fixtures, schemas, response shapes.
- Test targets: auth, permission routes, Query Registry endpoints, batch, users CRUD, sys lookup auth/public split, Excel worker helper, response conventions.
- Khong change production code unless explicitly asked.

### `hrm-merge-resolver`

- Resolve HRM conflicts.
- Read full conflicted file.
- Preserve both sides when possible.
- Special risk files: `queries/__init__.py`, router, API modules, Vue components, batch handlers, worker files.
- Checklist: no conflict markers, imports valid, query keys not duplicated, permission codes match, worker actions match caller.

### `hrm-release-checker`

- Check release/deploy risk.
- Focus runtime, auth, data, SQL, deployment, permissions.
- Check FE build/imports/routes/API/response/loading.
- Check BE route registration/auth/CORS/env/DB/service/query.
- Check SQL key registration/params/duplicates.
- Check Docker compose names, port mapping, healthcheck, CORS, reverse proxy.
- Role noi khong modify files unless explicitly asked.

### `hrm-mode-writer`

- Mode writer rieng cho HRM.
- Tao/refine custom modes cho HRM workflows.
- Mode responsibilities narrow/practical.
- Include project constraints and HRM gotchas.
- RoleDefinition co the bang English de model behave tot hon.
- Explanation/output nen dung Vietnamese.

### `hrm-ask`

- Tra loi cau hoi va giai thich implementation HRM.
- Answer based on actual codebase.
- Khong modify files, khong invent behavior.
- Neu depends on implementation, read relevant files first.
- Output Vietnamese, concise, practical.

### `hrm-xml-refactor`

- Convert legacy XML `<coldgrtieuchuan>` sang 4 Vue Grid config blocks:

```text
Grid_X_cols_all
Grid_X_cols_display
Grid_X_cols_labels
Grid_X_cols_types
```

- Column names must be UPPERCASE.
- Sort by `positionCol`.
- Preserve STT/TT if existing.
- Only modify target Grid_X blocks.
- Edit restricted vao `hrm_FE/src/components/.*\.vue$`.
- Co rule files rieng trong `.roo/rules-hrm-xml-refactor/`.

### `hrm-quick-assistant`

- Low-risk, lightweight help.
- Explain snippets, summarize logs, draft comments/docs/checklists/prompts.
- Khong modify files unless explicitly asked.
- Neu task complex/risky/deep multi-file, suggest stronger HRM mode.

## 10. HRM gotchas encoded trong modes

| Gotcha                                       | Can nho                                                                                                               |
| -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| API response shape khac nhau                 | SELECT: `response.data?.data`, CRUD: `response.data?.rows_affected`, batch: `response.data?.success` / `rowsInserted` |
| `registryBatchInsert` co the unwrap response | Co helper tra object truc tiep thay vi axios response                                                                 |
| Query key phai match exact                   | Sai key thuong trong nhu frontend bug                                                                                 |
| Query Registry la single point of failure    | Add query phai register trong `ALL_QUERIES`                                                                           |
| SQL params phai khop payload                 | Mismatch thuong chi fail runtime                                                                                      |
| Permission codes legacy                      | Thuong co dang `1frmXxx`, khong rename tuy tien                                                                       |
| Report API                                   | Co the tra blob/stream, khong xu nhu JSON                                                                             |
| Worker Excel                                 | Can preserve `postMessage` action/payload protocol                                                                    |
| Memory pagination                            | Mot so endpoint intentionally dung memory pagination                                                                  |
| Heartbeat session                            | Client heartbeat ~60s, cleanup server ~180s inactive                                                                  |
| Docker                                       | Service name co the khac container name                                                                               |

## 11. Frontend rules can giu trong modes

- Use Vue 3 Composition API with `<script setup>`.
- Use Element Plus, do not reintroduce Vuetify.
- Prefer shared components:

```text
MultiSelectGrid.vue
GridWithRowSpan.vue
Grid.vue
PdfViewer.vue
```

- Do not call axios directly from components unless local pattern requires it.
- Prefer API modules in `src/apis/`.
- Use utilities:

```text
showLoading, hideLoading, Success, Error, Confirm, Alert
```

- Use optional chaining for API responses.
- Respect permission-based routing.
- For Excel import/export, prefer existing worker/composable utilities.

## 12. Backend and Query Registry rules can giu trong modes

- Use FastAPI router/service patterns already present.
- Use dependency injection with `Depends`.
- Preserve JWT/session/permission behavior.
- Do not bypass auth checks.
- Do not store raw tokens.
- Do not send raw SQL from frontend.
- Prefer Query Registry for SQL operations.
- Use parameterized queries.
- Do not change database schema unless explicitly requested.
- Preserve audit logging where existing pattern applies.
- Keep router logic thin.
- Add SQL queries in relevant `queries/*_queries.py`.
- Register query keys in `queries/__init__.py`.
- Keep query keys explicit and module-scoped.

## 13. Workflow files dang co

| Workflow                          | Muc dich                                                                         |
| --------------------------------- | -------------------------------------------------------------------------------- |
| `.agent/workflows/autodocs.md`    | Cap nhat `docs/implements.md`, incremental mac dinh, full rescan khi user noi ro |
| `.agent/workflows/bugfix.md`      | Debug/fix bug tu symptom -> root cause -> fix -> verify                          |
| `.agent/workflows/newmodule.md`   | Tao module/report moi end-to-end FE -> BE -> SQL                                 |
| `.agent/workflows/rebuildUIS.md`  | Centralize UI theme tu hardcoded colors sang semantic tokens                     |
| `.agent/workflows/ref.md`         | Smart context loading, targeted lookup, khong quet toan bo project               |
| `.agent/workflows/refactorxml.md` | XML column definitions -> Vue Grid config                                        |
| `.agent/workflows/vb2vue.md`      | Migrate legacy VB.NET WinForms logic sang Vue 3                                  |

## 14. Skill files dang co

| Skill                                  | Khi dung                                            |
| -------------------------------------- | --------------------------------------------------- |
| `.agent/skills/agent-policy/SKILL.md`  | Moi coding task, quy tac read/reason/change/verify  |
| `.agent/skills/sql-query/SKILL.md`     | Khi tao/sua query, stored procedure, Query Registry |
| `.agent/skills/vue-component/SKILL.md` | Khi tao/sua/review Vue component trong `hrm_FE/`    |

## 15. Docs can biet

| Doc                                    | Noi dung                                              |
| -------------------------------------- | ----------------------------------------------------- |
| `docs/implements.md`                   | Tai lieu implementation chinh cua project             |
| `docs/response-checking-guide.md`      | Huong dan response shape cho registry/procedure/batch |
| `docs/startup-flow.md`                 | Startup flow                                          |
| `docs/heartbeat_session_management.md` | Heartbeat/session management                          |
| `docs/walkthrough_diemdanh_batch.md`   | Walkthrough batch diem danh                           |
| `docs/theme_analysis.md`               | Theme/UI analysis                                     |

## 16. Diem can ra soat neu sap chinh `.roomodes`

Mot so mode co roleDefinition noi read-only hoac "do not modify files", nhung `groups` lai co `edit`. Neu toi uu `.roomodes`, nen can nhac giam permission:

| Slug                   | Role noi gi                                              | Groups hien tai          |
| ---------------------- | -------------------------------------------------------- | ------------------------ |
| `hrm-flow-researcher`  | Do not modify/create/refactor                            | read, edit, command, mcp |
| `hrm-refactor-planner` | Do not edit/create; do not perform refactor unless asked | read, edit, command, mcp |
| `hrm-code-reviewer`    | Review, do not rewrite everything                        | read, edit, command, mcp |
| `hrm-sql-auditor`      | Do not modify files unless explicitly asked              | read, edit, command, mcp |
| `hrm-release-checker`  | Do not modify files unless explicitly asked              | read, edit, command, mcp |
| `hrm-ask`              | Do not modify files                                      | read, edit, command, mcp |

Goi y:

- Read-only modes nen chi co `read`, `command`, `mcp` neu can search/inspect.
- Reviewer/auditor/checker co the giu `edit` neu muon cho phep sua sau khi user yeu cau, nhung roleDefinition can noi ro dieu kien.
- Neu role cam edit tuyet doi, bo `edit` khoi groups de enforce bang tool permission.
- Neu mode chi nen edit mot loai file, dung restricted edit group voi `fileRegex`.

## 17. Checklist khi edit `.roomodes`

- Preserve `customModes:` top-level.
- Khong doi `slug` neu mode dang duoc dung.
- Keep YAML indentation exactly.
- Dung block scalar `|-` cho prompt dai.
- Cap nhat `description` va `whenToUse` neu roleDefinition thay doi.
- Kiem tra `groups` co khop voi roleDefinition.
- Neu them/sua mode co rule files, tao/cap nhat `.roo/rules-[slug]/`.
- Khong them behavior speculative.
- Khong xoa HRM gotchas ve response shape, Query Registry, permission, worker, heartbeat.
- Khong sua `.git`, `.env`, lock files, binary files.
- Sau khi sua, validate bang cach doc lai `.roomodes` va dem mode/slug.

## 18. Prompt ngan co the dua cho ChatGPT

```markdown
Ban dang giup toi chinh `.roomodes` cho project HRM.

Project context:

- Frontend: Vue 3 Composition API, `<script setup>`, Vite, Element Plus, Pinia, Vue Router permission guards.
- Backend: FastAPI, SQLAlchemy 2.0, pyodbc, SQL Server.
- Architecture: centralized FE API modules, FastAPI Router -> Service Layer -> Query Registry, Batch Processing, Web Worker Excel, JWT/heartbeat session.
- Root `C:\Source\Gitea\HRM` la container, khong co `.git`; `hrm_FE/.git` va `hrm_BE/.git` la hai repo rieng.

Rules:

- Preserve correctness, minimal diff, low regression risk.
- Read code before editing.
- Do not guess query keys, endpoints, permission codes, response shapes, worker actions.
- Preserve Vietnamese business terms and legacy naming.
- Do not change database schema/dependencies/config unless explicitly requested.
- Response gotchas: SELECT uses `response.data?.data`, CRUD uses `response.data?.rows_affected`, batch uses `success/rowsInserted`, reports may return blob/stream.

Current Roo files:

- `.roomodes`: 18 custom modes.
- `.roo/mcp.json`: context7 and playwright MCP.
- `.roo/rules-*`: external XML rules for mode-writer, research, merge-resolver, hrm-xml-refactor.
- `.rooignore`: ignores git, env/secrets, caches, node_modules, dist, locks, binaries.

When editing `.roomodes`:

- Keep YAML schema and indentation valid.
- Keep `slug`, `name`, `roleDefinition`, `groups`; update `description` and `whenToUse` together with behavior.
- Align `groups` permissions with the role. If a mode is read-only, remove `edit` unless intentionally allowed.
- Prefer restricted edit permissions with `fileRegex` for mode-specific writers/refactors.
- Do not weaken HRM safety rules around Query Registry, auth, permission routing, worker protocol, heartbeat, audit logging.
```
