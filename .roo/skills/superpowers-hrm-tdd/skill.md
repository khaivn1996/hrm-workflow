---
name: superpowers-hrm-tdd
description: Use when implementing backend logic, bug fixes, behavior changes, validation, auth/session behavior, Query Registry integration, or reusable frontend logic.
---

# Superpowers HRM TDD

Core rule: for logic and bug fixes, prefer RED -> GREEN -> REFACTOR.

## When Required

- Backend service/router behavior.
- Query Registry integration or parameter behavior.
- Batch handling.
- Auth, heartbeat, permission checks.
- Utility, composable, worker helper, or reusable frontend logic.
- Bug fixes where an automated regression test is feasible.

## Lightweight Exceptions

TDD may be skipped for pure docs/rules, generated text, trivial copy edits, or UI-only layout changes without an existing test harness. Still define manual verification.

## Cycle

1. RED: write one focused failing test for the desired behavior or bug.
2. Verify RED: run the test and confirm it fails for the expected reason.
3. GREEN: write the smallest implementation that passes.
4. Verify GREEN: run the targeted test again.
5. Refactor only if needed, keeping the test green.

## HRM Test Targets

- Query key exists and is registered in `ALL_QUERIES`.
- SQL params match frontend/backend payload names.
- SELECT, CRUD, batch, and report response shapes are handled correctly.
- Permission behavior matches route/menu/store usage.
- Worker action and payload protocol stay compatible.

## Commands

Use existing project commands. Backend default:

```bash
cd hrm_BE
python -m pytest tests/ -v
```

For frontend, inspect `hrm_FE/package.json` before choosing test, lint, or build commands.
