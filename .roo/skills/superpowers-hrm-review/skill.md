---
name: superpowers-hrm-review
description: Use after implementation, for code review feedback, and before release or deploy checks.
---

# Superpowers HRM Review

Review is a quality gate, not a style pass. Findings must be grounded in files and behavior.

## Requesting Review

Use `hrm-code-reviewer` after meaningful implementation and before broad rollout. Focus on:

- correctness and regressions
- Query Registry registration and params
- response shape compatibility
- permission/auth/session behavior
- worker/report protocol
- transaction and batch behavior
- missing or weak verification

Report findings first, ordered by severity.

## Receiving Review

Before applying feedback:

1. Read the full feedback.
2. Restate unclear items or ask a focused question.
3. Verify the suggestion against HRM code reality.
4. Implement one item at a time.
5. Verify each fix.

Push back with technical evidence when a suggestion breaks HRM patterns, violates YAGNI, changes public contracts, or conflicts with legacy compatibility.

## Release Check

Use `hrm-release-checker` before deploy/release. Check:

- frontend build risk and route imports
- backend route registration and auth dependencies
- Query Registry keys and SQL params
- permissions and menu visibility
- worker/report compatibility
- env/config assumptions without modifying them
- smoke test list and rollback notes when relevant

Do not claim ready unless fresh verification evidence exists.
