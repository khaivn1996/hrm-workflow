---
name: superpowers-hrm-debugging
description: Use for bugs, test failures, API errors, bad data, permission failures, worker errors, and unexpected UI behavior.
---

# Superpowers HRM Debugging

Core rule: no fix before root-cause investigation.

## Trigger

- UI does not load data.
- API returns 400, 401, 403, 500, or unexpected shape.
- Query key, SQL params, batch insert, report download, Excel worker, auth, heartbeat, or permission behavior is wrong.
- A test, build, or deployment check fails.

## Four Phases

### 1. Reproduce and Gather Evidence

- Record exact symptom, route/screen, request payload, response payload, console error, stack trace, or failing command.
- If not reproducible, gather more evidence instead of guessing.

### 2. Trace Root Cause

Trace backward from the failure:

`UI symptom -> component state -> API call -> endpoint -> service/batch/query -> SQL -> data/auth/permission source`.

For deep failures, keep asking: what passed this value, and where did it originate?

### 3. Compare With Working Patterns

- Find a similar working HRM flow.
- Compare query key registration, params, response checks, loading/error handling, permissions, and worker messages.
- List meaningful differences before changing code.

### 4. Fix and Verify

- Add or identify the smallest failing test/check when practical.
- Apply one minimal fix at the root cause.
- Verify the original symptom and a nearby regression path.

## HRM Gotchas

- Wrong query key can look like a frontend bug.
- Report APIs may return blobs, not JSON.
- Permission code mismatches may hide menus or block routes.
- Worker action/payload mismatches can fail away from the caller.
- Heartbeat 401 behavior may trigger logout through interceptors.

## Recommended HRM Mode

Use `hrm-debug-specialist` first. Use `hrm-test-writer` for a regression test, then `hrm-code-implementer` for the fix.
