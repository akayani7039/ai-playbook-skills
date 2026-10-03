---
name: ui-test-playwright
description: Run Playwright UI tests against a Salesforce org in one of four modes - story UI test (dev), smoke (QA), full regression (UAT) or production smoke. Use when the user asks to run UI, smoke, regression or e2e tests. Pipeline steps 17, 31, 38, 47.
---

# ui-test-playwright

## Tools
- Playwright MCP
- Salesforce CLI (org login / URL)

## Modes

| Mode | Step | Org | Scope |
|---|---|---|---|
| `story` | 17 | Story dev org | Tests for the story's acceptance criteria |
| `smoke` | 31 | QA | Smoke suite after QA deploy |
| `regression` | 38 | UAT | Full regression suite |
| `prod-smoke` | 47 | Production | Read-only smoke suite |

## Steps
1. Confirm the mode and target org.
2. Open the org and log in.
3. Run the suite for the mode.
4. Capture screenshots and results as evidence.
5. Report pass/fail. On failure, show the failing step and screenshot.

## Rules
- `prod-smoke` must not create or change data.

## TODO
- [ ] Test folder layout and suite names
- [ ] Org auth approach for Playwright
- [ ] Where evidence is saved (for PR and approver brief)
- [ ] Test data setup and cleanup
