---
name: story-build
description: Build a Jira story from its approved architecture.md - write the code and tests from the build plan, retrieve metadata, run Jest, deploy to the dev org, run Apex tests, fix and re-test until green, then hold for the developer. Use after architecture-brief is approved, or when the user says "build BR014-S1 from architecture.md". Pipeline steps 12-19.
---

# story-build

Stage 03 — Build & validate (steps 12-19). Builds only what the approved `architecture.md` says.

## Tools
- Salesforce CLI, Salesforce MCP
- Jest (`npm run test:unit`), ESLint (`npm run lint`)
- Official skills: `platform-metadata-retrieve`, `dx-code-analyzer-run`, `experience-lwc-security-validate`
- `ui-test-playwright` (step 17)

## Before you start
- Work in the story's worktree on `feature/BR014-S<n>`. Never on `main` or `develop`.
- Read `features/BR014-S<n>/architecture.md` and `README.md`.
- **Stop if `architecture.md` is not approved.** The status line must say `Approved` with an architect and date. Without it, no code (step 11 gate).
- Read the project `CLAUDE.md`. Its Apex, LWC and metadata rules apply to every file.
- Get the story's dev org from the feature `README.md`. Confirm it is active.

## Steps
12. **Build change** — Follow section 6 (Design and build plan) row by row, in the deploy order given. Write each component and its tests. Do not add components the plan does not list. If the plan is wrong or incomplete, stop and ask. Do not redesign.
13. **Retrieve metadata** — Retrieve any org metadata the plan changes (layouts, permission sets, flows, OmniStudio) before editing it, so you edit the org's current version.
14. **Run Jest tests** — `npm run lint && npm run test:unit`. Skip only if the plan says the story has no LWC.
15. **Deploy to dev** — `sf project deploy start --source-dir force-app -o <dev-org> -w 30`.
16. **Run Apex tests** — Run the story's test classes with coverage: `sf apex run test -o <dev-org> -n <TestClass> -c -r human -w 30`. New classes need 90% or more.
17. **Run UI tests** — Hand off to `ui-test-playwright` in `story` mode. Skip only if the plan says the story has no UI.
18. **Fix and re-test** — On any failure, read the error, fix the cause, and re-run from the failed step. Fix the code, not the test, unless the test is wrong against the acceptance criteria. Say which one you changed and why.
19. **Build complete** — HUMAN GATE. Show the summary below. Stop and wait for the developer to confirm every test is green.

## Rules
- Bulk-safe code. No SOQL or DML in loops. Tests insert 200 records.
- Queries in a selector `WITH USER_MODE`. DML in user mode.
- Lead logic goes in a service class, called from `LeadTriggerHandler`. No logic in triggers.
- Reuse `LeadDataStandardizer`, `LeadSelector` and `TestDataFactory`. Don't rebuild what exists.
- Every acceptance criterion maps to at least one test. Use the plan's test table.
- Demo data only. No `System.debug` of lead data.
- Never deploy to QA, UAT or production. Dev org only.
- Do not commit or open a PR. That is `pr-package` (step 20).

## Output
Update the feature `README.md` "Components changed" table. Then report:

| Check | Result |
|---|---|
| Components built | List vs. the build plan |
| Lint + Jest | Pass/fail, or skipped (why) |
| Dev deploy | Pass/fail, deploy ID |
| Apex tests | Pass count, coverage per new class |
| UI test | Pass/fail, or skipped (why) |
| AC coverage | Each AC → test name |
| Fixes made | What failed, what changed |

Hands off to `pr-package` (step 20).

## TODO
- [ ] Checkpoint tags (`demo/stage-*`) for the live demo
- [ ] Whether to run Code Analyzer here or only in CI (step 25)
