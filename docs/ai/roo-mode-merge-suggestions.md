# Roo Mode Merge Suggestions

This file is advisory only. `.roomodes` was read and analyzed but not modified.

## Manual Merge Needed

Yes, optional. The current integration works through additive rules, skills, workflows, and docs. Manual `.roomodes` edits are only needed if you want structural enforcement at the mode permission level.

## Existing Mode Mapping

Do not create duplicate SuperRoo modes. Map Superpowers behavior into existing HRM modes:

| Superpowers workflow | Existing HRM mode |
|---|---|
| using-superpowers | `hrm-quick-assistant` plus workspace rules |
| brainstorming | `hrm-flow-researcher`, `hrm-refactor-planner` |
| writing-plans | `hrm-refactor-planner` |
| executing-plans | `hrm-code-implementer` |
| test-driven-development | `hrm-test-writer`, `hrm-code-implementer` |
| systematic-debugging | `hrm-debug-specialist` |
| root-cause-tracing | `hrm-debug-specialist` |
| verification-before-completion | `hrm-release-checker`, workspace verification rule |
| requesting-code-review | `hrm-code-reviewer` |
| receiving-code-review | `hrm-code-reviewer` |
| using-git-worktrees | do not automate for HRM container root |
| finishing-a-development-branch | `hrm-release-checker` |

## Permission Alignment Suggestions

Several modes say they are read-only or should not modify files, but their `groups` include `edit`. If you want tool-level enforcement, consider these changes.

| Mode | Current risk | Suggested groups |
|---|---|---|
| `hrm-flow-researcher` | role says do not modify/create/refactor, groups include `edit` | `read`, `command`, `mcp` |
| `hrm-refactor-planner` | role says do not edit/create, groups include `edit` | `read`, `command`, `mcp` |
| `hrm-ask` | role says do not modify files, groups include `edit` | `read`, `command`, `mcp` |
| `hrm-code-reviewer` | review mode may need to stay read-only until user asks for fixes | either `read`, `command`, `mcp` or keep `edit` but clarify "only after explicit fix request" |
| `hrm-sql-auditor` | audit mode should not modify unless explicitly asked | either `read`, `command`, `mcp` or restricted `edit` for query files only |
| `hrm-release-checker` | release checker should usually not modify | `read`, `command`, `mcp` unless release fixes are explicitly in scope |

Approximate sections from the current scan:

- `hrm-flow-researcher`: slug near `.roomodes:117`, groups near `.roomodes:194`.
- `hrm-refactor-planner`: slug near `.roomodes:200`, groups near `.roomodes:282`.
- `hrm-code-reviewer`: slug near `.roomodes:387`, groups near `.roomodes:492`.
- `hrm-sql-auditor`: slug near `.roomodes:498`, groups near `.roomodes:599`.
- `hrm-release-checker`: slug near `.roomodes:965`, groups near `.roomodes:1037`.
- `hrm-ask`: slug near `.roomodes:1092`, groups near `.roomodes:1131`.

## Suggested RoleDefinition Additions

If editing `.roomodes` later, merge short pointers rather than pasting the full Superpowers methodology into every mode.

### `hrm-code-implementer`

Add:

```text
Before editing, classify the task. If it is cross-layer, risky, or behavior-changing, present a brief plan with files to modify and verification. For logic/backend changes, prefer TDD: failing test, minimal implementation, passing test. Do not claim completion without fresh verification evidence.
```

### `hrm-debug-specialist`

Add:

```text
No fix before root cause. Reproduce the symptom, trace backward through component/API/router/service/query/SQL or worker/auth/permission layers, compare with a working pattern, then apply one minimal fix and verify the original symptom.
```

### `hrm-refactor-planner`

Add:

```text
Perform impact analysis before planning. Preserve API contracts, query keys, response shapes, permission behavior, worker protocol, auth/session behavior, and Vietnamese legacy naming. Prefer incremental migration with verification after each step.
```

### `hrm-test-writer`

Add:

```text
For behavior changes and bug fixes, write the smallest failing test first and verify it fails for the expected reason before implementation. Cover Query Registry, auth, permission, batch, worker, and response-shape regressions where relevant.
```

### `hrm-code-reviewer`

Add:

```text
Findings first, ordered by severity. Check correctness, missing verification, response shapes, Query Registry, permissions, auth/session, worker/report protocol, and regression risk. Avoid style-only feedback unless it affects behavior or maintainability.
```

### `hrm-release-checker`

Add:

```text
Do not mark ready without verification evidence. Check targeted tests/builds, smoke paths, deployment assumptions, env/config dependencies, rollback notes, and unresolved risks.
```

## Do Not Merge

- Do not add separate `using-superpowers`, `test-driven-development`, `systematic-debugging`, or other duplicate SuperRoo modes unless the existing HRM mode set is intentionally redesigned.
- Do not paste generic SuperRoo `.roomodes` over HRM `.roomodes`.
- Do not weaken HRM-specific constraints around Query Registry, permissions, response shapes, worker protocol, heartbeat, audit logging, and legacy Vietnamese naming.
