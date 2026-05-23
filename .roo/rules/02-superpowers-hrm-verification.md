# Superpowers HRM Verification Rules

Trigger: use before claiming work is complete, fixed, safe, reviewed, or ready.

Core rule: evidence first. If verification was not run, say exactly what was not run and why.

## Verification Order

Use the narrowest meaningful check first:

1. Targeted test for the touched behavior.
2. Relevant typecheck, lint, or syntax check.
3. Build for the affected app.
4. Broader suite only when risk justifies it.

Do not use a passing linter as proof that runtime behavior works.

## HRM Verification Matrix

| Change area | Minimum verification |
|---|---|
| Backend service/router | targeted pytest or endpoint smoke check |
| Query Registry | key exists, key registered, params match SQL, response shape checked |
| CRUD frontend flow | API function checked, `rows_affected` handling verified |
| SELECT frontend flow | API function checked, `data` handling verified |
| Batch flow | payload rows, transaction behavior, `success` response checked |
| Permission route/menu | route meta, store/menu usage, and legacy code checked |
| Worker/Excel flow | `postMessage` action, payload, response shape, and UI caller checked |
| Report flow | blob/stream handling checked, not treated as normal JSON |
| Auth/session | 401/403, heartbeat interval, cleanup behavior checked |
| UI-only change | browser/manual steps or screenshot check, plus console/network check when relevant |
| Docs/rules only | file presence, Markdown readability, and no forbidden file touched |

## Completion Report

When reporting completion, include:

- Files created or changed.
- What changed and why.
- Verification run, with command or concrete manual steps.
- Verification not run, if any, with reason.
- Remaining risks, only if meaningful.

## Stop Conditions

Stop and ask or report a blocker when:

- The required query key, endpoint, permission code, or worker action cannot be verified locally.
- A change would require DB schema, dependency, env/config, `.roomodes`, or git operations outside the user's stated permission.
- A bug cannot be reproduced and there is not enough evidence to identify root cause.
