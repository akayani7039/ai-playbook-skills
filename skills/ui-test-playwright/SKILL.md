---
name: ui-test-playwright
description: Run Playwright UI tests against a Salesforce org in one of four modes - story UI test (dev), smoke (QA), full regression (UAT) or production smoke. Use when the user asks to run UI, smoke, regression or e2e tests. Pipeline steps 17, 31, 38, 47.
---

# ui-test-playwright

## Tools
- Playwright test runner (`@playwright/test`, config `playwright.config.ts`), from the app repo or the story worktree
- Salesforce CLI: `tests/ui/fixtures/salesforce.ts` logs in through `sf org display` for `TARGET_ORG`. No password needed.

## Modes

| Mode | Step | Org (`TARGET_ORG`) | Command |
|---|---|---|---|
| `story` | 17 | `playbook-dev` (38: `playbook-uat`) | `npm run test:ui:story` |
| `smoke` | 31 | `playbook-qa` | `npm run test:ui:smoke` |
| `regression` | 38 | `playbook-uat` | `npm run test:ui:regression` |
| `prod-smoke` | 47 | `playbook-prod` | `npm run test:ui:prod-smoke` |

Always prefix: `SF_SKIP_NEW_VERSION_CHECK=true NO_COLOR=1 FORCE_COLOR=0 TARGET_ORG=<alias>`. The CLI update notice breaks the login otherwise.

## Steps
1. Confirm the mode and target org. Check the org with `sf org display -o <alias>` (the first call can fail with NoScratchInfo; retry once).
2. `story` mode: write the story's tests in `tests/ui/story/BR014-S<n>.spec.ts`, one `test()` per UI acceptance criterion, named after it. Use the fixture in `tests/ui/fixtures/salesforce.ts` and follow `tests/ui/smoke/lead-record-page.spec.ts`. Use the sample leads; create test data only in dev, QA or UAT, and clean it up.
3. Run the command for the mode.
4. A cold scratch org can render a blank page on the first test. If the only failure is a blank page or timeout on the first load, re-run once and say so.
5. Evidence: `playwright-report/` (HTML) and `test-results/` (screenshots of failures). Both are git-ignored; reference them, don't commit them.
6. Report pass/fail per test. On failure, show the failing step and screenshot path.

## Rules
- `prod-smoke` must not create or change data.
- Never print the frontdoor URL or access token. Traces and video stay off.
- UAT and QA need the 12 sample leads. If they're missing: `npm run data:load -- --target-org <alias>` (never prod).
