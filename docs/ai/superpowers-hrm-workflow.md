# Superpowers HRM Workflow

This document adapts the Superpowers methodology to the existing HRM Roo Code workspace.

Sources reviewed:

- `https://github.com/obra/superpowers`
- `https://github.com/Benny-Lewis/super-roo`
- Local HRM files: `.roomodes`, `AGENTS.md`, `.agent/AGENT.md`, `.agent/skills/*`, `.agent/workflows/*`, `.roo/rules-*`, `.roo/commands/*`, `.rooignore`.

## Goal

Keep HRM work disciplined without replacing existing HRM modes:

```text
Read -> Reason -> Plan or Change -> Verify
```

The integration is additive. It does not create new Roo modes and does not modify `.roomodes`.

## How Roo Should Use It

Automatically loaded workspace rules:

- `.roo/rules/01-superpowers-hrm-core.md`
- `.roo/rules/02-superpowers-hrm-verification.md`

On-demand references:

- `.roo/skills/superpowers-hrm-planning/skill.md`
- `.roo/skills/superpowers-hrm-debugging/skill.md`
- `.roo/skills/superpowers-hrm-tdd/skill.md`
- `.roo/skills/superpowers-hrm-review/skill.md`
- `.agent/workflows/superpowers-hrm.md`

If Roo Code does not auto-discover `.roo/skills`, reference the skill path in the prompt or merge pointers into the relevant existing HRM mode later.

## Superpowers Mapping

| Superpowers skill | HRM mapping |
|---|---|
| using-superpowers | core rules plus `hrm-quick-assistant` routing |
| brainstorming | `hrm-flow-researcher`, then `hrm-refactor-planner` |
| writing-plans | `hrm-refactor-planner`, `.agent/workflows/superpowers-hrm.md` |
| executing-plans | `hrm-code-implementer` with checkpoints |
| test-driven-development | `hrm-test-writer`, then `hrm-code-implementer` |
| systematic-debugging | `hrm-debug-specialist` |
| root-cause-tracing | `hrm-debug-specialist` trace phase |
| verification-before-completion | `.roo/rules/02-superpowers-hrm-verification.md` |
| requesting-code-review | `hrm-code-reviewer` |
| receiving-code-review | `hrm-code-reviewer` feedback process |
| using-git-worktrees | manual only; HRM root is not a git repo |
| finishing-a-development-branch | `hrm-release-checker` |

## Task Routing

| Request | Start with |
|---|---|
| "Trace this feature" | `hrm-flow-researcher` |
| "Plan a refactor" | `hrm-refactor-planner` |
| "Implement this plan" | `hrm-code-implementer` |
| "Fix this bug" | `hrm-debug-specialist` |
| "Add tests" | `hrm-test-writer` |
| "Review this change" | `hrm-code-reviewer` |
| "Check before deploy" | `hrm-release-checker` |
| "Update Roo modes" | `hrm-mode-writer` |

## HRM Guardrails

- Do not invent query keys, endpoint paths, permission codes, response shapes, or worker actions.
- Preserve Query Registry and Batch Registry patterns.
- Preserve Vue 3 `<script setup>`, Element Plus, Pinia, router guards, API modules, FastAPI router/service boundaries, SQLAlchemy 2.0, SQL Server params, auth, heartbeat, and audit logging.
- Preserve Vietnamese business terms and legacy names.
- Do not change DB schema, dependencies, env/config, or `.roomodes` unless explicitly requested.
- Do not run git at `C:\Source\Gitea\HRM`; it is a container folder.

## Lightweight Rule

Do not force heavy process onto tiny tasks. A small explanation, typo fix, or docs-only adjustment can use a short read/change/verify loop.

Use full planning when the task is cross-layer, risky, user-facing, a refactor, a bug without root cause, or touches Query Registry, permissions, worker protocol, auth, batch, reports, or release behavior.

## Verification

Before saying work is complete:

- state the verification command or manual steps
- run the narrowest meaningful check when possible
- report checks not run and why
- list unresolved risk if any

Backend default:

```bash
cd hrm_BE
python -m pytest tests/ -v
```

Frontend commands must be read from `hrm_FE/package.json` before use.
