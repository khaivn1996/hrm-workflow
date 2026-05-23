---
description: Create a persistent Markdown completion report for the current Roo task
argument-hint: <short-task-name>
---

Create a Markdown completion report for the current task.

Save it as a real file under:

`docs/ai/task-reports/YYYY-MM-DD_HH-mm-<short-task-name>.md`

Use the current local date/time. If `<short-task-name>` is not provided, derive a short kebab-case name from the task.

Do not create a preview-only document. Use file editing/write tools to create the `.md` file in the workspace.

The report must include:

# Task Report: <task title>

## Metadata

- Date:
- Tool / Mode:
- Status:
- Original request:
- Report path:

## Summary

- What was requested.
- What was done.

## Files Created

- path: purpose

## Files Modified

- path: what changed and why

## Files Not Touched

- important files intentionally not changed

## HRM Flow / Impact

- frontend impact, if any
- backend impact, if any
- Query Registry impact, if any
- permission/route impact, if any
- worker/report impact, if any

## Verification

- commands/tests/manual checks run
- results
- checks not run and why

## Risks / Follow-up

- remaining risks
- recommended next steps

Rules:

- Be factual.
- Do not invent commands, files, query keys, endpoints, permission codes, or test results.
- If verification was not run, say exactly that.
- If no files were changed, say "No files changed."
- Keep the report concise but useful.
- Only create or overwrite the single report file for this export.
- Do not modify source code, `.roomodes`, config, or other docs.
- Create `docs/ai/task-reports/` if it does not exist.
- Include enough context for another AI coding tool to resume the task.
