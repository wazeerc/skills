---
name: chrome-validation
description: 'Validate or smoke-test a web feature in Chrome browser with Chrome DevTools MCP.'
---

# Chrome Validation

Goal: exercise a feature in a real Chrome session the way a user would, then report PASS, FAIL or BLOCKED with browser evidence.

## Step 0 — MCP check

Confirm the Chrome DevTools MCP tools are available. Treat a configured-but-failing or unauthenticated server as unavailable.

If they're missing, abort — don't substitute other tools or write a report. Tell the user to install and complete the setup from https://github.com/ChromeDevTools/chrome-devtools-mcp/ (add the server to their MCP client config — `npx -y chrome-devtools-mcp@latest` — then restart the session) and re-run this skill once the tools appear.

## Step 1 — Inputs

Ask for anything missing, in one batch:

- Feature: feature or user workflow to validate
- URL: starting URL, route, or local app URL
- Acceptance criteria: observable behaviors that must pass
- Viewport or device: optional; default to desktop and note what you used

## Step 2 — Validate

1. Read the workspace and app instructions only as far as needed to reach the feature. Never modify source, tests, fixtures, or config — validation observes, it doesn't change the app.
2. Open the URL or reuse the running app. If it isn't running, find the documented start command and ask the user to start it.
3. Exercise each criterion as a user would:
   - Use visible text, roles, and labels — not implementation selectors.
   - Cover the happy path, relevant empty/loading/error states, and one useful boundary case when it applies.
   - Wait for observable state changes, not fixed delays.
4. Collect evidence: rendered page and key controls, console errors and warnings, failed requests and status codes, and a screenshot of the final observed state. If validation fails, also capture the failure state when useful.
5. When a criterion fails, reproduce it once. Classify it: product defect, environment/setup problem, or unclear requirement — and don't claim a code-level cause without reading the code.

## Step 3 — Report

Return this format:

```markdown
## Result

PASS | FAIL | BLOCKED

## Environment

- URL:
- Viewport/device:
- Browser/session:

## Criteria

| Criterion | Result | Evidence |
| --- | --- | --- |
| ... | PASS/FAIL/BLOCKED | ... |

## Diagnostics

- Console:
- Network:
- Relevant UI state:
- Screenshot of final observed state:
- Failure-state screenshot, if applicable:

## Notes

- Reproduction steps for failures or blockers
- Category: product defect / environment/setup / unclear requirement
- Follow-up needed, if any
```

Keep the report factual and reproducible: exact error text, failing requests, and the last successful step.
A clean console is not proof a criterion passed — observe the criteria directly.
