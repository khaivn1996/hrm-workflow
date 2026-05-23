---
description: Superpowers methodology adapted for HRM Roo/agent workflows
---

# Workflow: Superpowers HRM

Use this workflow for non-trivial HRM tasks. Keep simple tasks lightweight.

## 1. Route the Task

| Task type | First mode/workflow |
|---|---|
| Understand existing flow | `hrm-flow-researcher` |
| Plan large feature/refactor | `hrm-refactor-planner` |
| Implement scoped change | `hrm-code-implementer` |
| Debug bug/failure | `hrm-debug-specialist` |
| Write tests | `hrm-test-writer` |
| Review change | `hrm-code-reviewer` |
| Release/deploy check | `hrm-release-checker` |
| Mode/rule authoring | `hrm-mode-writer` |

## 2. Read

Read real files before conclusions. For feature work, trace:

```text
Vue component
-> API module
-> FastAPI router
-> service or batch handler
-> Query Registry
-> SQL query
-> permission/store/worker flow if relevant
```

## 3. Reason

Identify:

- actual endpoint and API function
- query key and registration
- SQL params and payload params
- response shape
- permission code
- worker action/payload
- current tests and verification commands

If any item cannot be verified, say so before editing.

## 4. Plan or Change

Plan first when the task is risky, cross-layer, or a refactor. The plan must list files to create/modify/not touch and verification steps.

For implementation:

- minimal diff
- no unrelated cleanup
- no new dependency
- no DB schema change
- no public contract change unless requested

## 5. Verify

Verification order:

1. targeted test or reproduction
2. typecheck/lint/syntax
3. build
4. broader suite

For UI work, include concrete manual/browser steps if no automated check exists.

## 6. Report

Use:

```text
Task:
Findings:
Changes:
Verification:
Risks:
Unknowns:
```

Do not say complete, fixed, safe, or ready without verification evidence.
