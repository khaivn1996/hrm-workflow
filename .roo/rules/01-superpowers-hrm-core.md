# Superpowers HRM Core Rules

Trigger: load for every Roo task in this workspace.

Purpose: bring the Superpowers discipline into the existing HRM modes without replacing them. Use existing HRM modes first. Do not create duplicate modes.

## Core Loop

Use this default loop:

1. Read local context.
2. Reason from verified files, not names or guesses.
3. Plan before edits when the task is multi-step, risky, cross-layer, or user-facing.
4. Change the smallest necessary surface.
5. Verify with the narrowest meaningful check.

Small tasks can stay lightweight. A typo, short explanation, or single obvious docs edit does not need a heavy plan. Still read the relevant file before editing.

## Mode Mapping

Use the existing HRM modes as the Superpowers equivalent:

| Superpowers workflow | HRM mode or workflow |
|---|---|
| using-superpowers | this rule plus `hrm-quick-assistant` for routing |
| brainstorming | `hrm-flow-researcher`, then `hrm-refactor-planner` for design |
| writing-plans | `hrm-refactor-planner`, `.agent/workflows/superpowers-hrm.md` |
| executing-plans | `hrm-code-implementer` with checkpoints |
| test-driven-development | `hrm-test-writer`, then `hrm-code-implementer` |
| systematic-debugging | `hrm-debug-specialist` |
| root-cause-tracing | `hrm-debug-specialist` trace phase |
| verification-before-completion | `.roo/rules/02-superpowers-hrm-verification.md` |
| requesting-code-review | `hrm-code-reviewer` |
| receiving-code-review | `hrm-code-reviewer`, then relevant implementer mode |
| using-git-worktrees | not automatic in HRM; respect user git restrictions |
| finishing-a-development-branch | `hrm-release-checker` |

## HRM Non-Negotiables

- Do not guess query keys, endpoints, permission codes, response shapes, store names, or worker actions.
- Preserve Vietnamese business terms, route names, permission codes, and legacy naming.
- Preserve Query Registry, Batch Registry, service-layer boundaries, auth dependencies, heartbeat session behavior, audit logging, and worker protocol.
- Do not change DB schema, dependencies, env/config, public contracts, or `.roomodes` unless explicitly requested.
- Do not run git at workspace root. The root is a container. `hrm_FE/.git` and `hrm_BE/.git` are separate repos.
- Do not commit, push, reset, clean, checkout, rename, move, disable, delete, or otherwise touch `.git` unless explicitly requested and approved.

## Planning Gate

Plan before editing when any of these are true:

- The task spans frontend and backend.
- Query Registry, permission routing, worker messages, auth, batch handling, reports, or database behavior is involved.
- A refactor could change behavior.
- A bug has not been reproduced or traced to a root cause.
- UI behavior needs manual/browser verification.

The plan must name files to create or edit, files not to touch, and the verification command or manual check.

## Implementation Discipline

- Prefer existing API modules under `hrm_FE/src/apis/`.
- Keep routers thin and service/query logic in existing backend layers.
- Use parameterized SQL and registered query keys.
- Preserve response checks:
  - SELECT: `response.data?.data`
  - CRUD: `response.data?.rows_affected`
  - batch: `response.data?.success`
  - reports: blob or stream behavior may apply.
- Keep diffs small and reviewable. Do not bundle unrelated cleanup.
