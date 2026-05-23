---
name: superpowers-hrm-planning
description: Use before large features, cross-layer changes, refactors, mode edits, or any task where implementation risk is not trivial.
---

# Superpowers HRM Planning

Use this skill when a task needs design or sequencing before edits. Do not use it for tiny one-file edits unless the file controls Query Registry, permissions, auth, worker protocol, or public API behavior.

## Trigger

- New module, report, batch flow, worker flow, or backend logic.
- Refactor of a large Vue component or backend service.
- Any task touching both `hrm_FE/` and `hrm_BE/`.
- Any `.roomodes` change request.

## Process

1. Restate the goal in one or two sentences.
2. Read current files first. For HRM flows, trace:
   `Vue component -> API module -> FastAPI router -> service/batch/query registry -> SQL`.
3. Identify actual query keys, endpoints, response shapes, permission codes, stores, and worker actions.
4. Choose the smallest approach that preserves current behavior.
5. Present a plan with:
   - files to create
   - files to modify
   - files not to touch
   - verification for each step
6. Implement only after the plan is accepted or the user already requested execution.

## Refactor Rules

- Impact analysis before edits.
- Preserve public API contracts, permissions, query keys, response shapes, worker actions, and Vietnamese legacy names.
- Split only when it reduces real complexity or matches an existing local pattern.
- Do not introduce dependencies or schema changes.

## Recommended HRM Modes

- Research first: `hrm-flow-researcher`.
- Plan refactor: `hrm-refactor-planner`.
- Implement plan: `hrm-code-implementer`.
- Review result: `hrm-code-reviewer`.
