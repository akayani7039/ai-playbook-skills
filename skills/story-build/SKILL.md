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
- Get the story's dev org from the feature `README.md` (`playbook-dev`, shared by all story worktrees). Confirm it is active.
- Read the old approved design for this story if it exists, to reuse its decisions: `git show dd9602e:features/BR014-S<n>/architecture.md` (S1, S2) or `git show e941639:features/BR014-S3/architecture.md`.

## Plugin hook
The salesforce-development plugin denies raw `sf project retrieve`, `sf data query`, `sf apex run test` and `sf project generate manifest` until its owning skill (`platform-metadata-retrieve`, `platform-soql-query`, `platform-apex-test-run`, `platform-manifest-generate`) has been dispatched in the same turn. Dispatch that skill first, then run the command shown here. Deploy commands only get a warning.

## Steps
12. **Build change** — Follow section 6 (Design and build plan) row by row, in the deploy order given. Write each component and its tests. Do not add components the plan does not list. If the plan is wrong or incomplete, stop and ask. Do not redesign.
13. **Retrieve metadata** — Retrieve any org metadata the plan changes (layouts, permission sets, flows, OmniStudio) before editing it, so you edit the org's current version. Retrieve by name (`-m Layout:Lead-Lead Layout`), into a scratch folder if you only need to compare. Never retrieve all of `CustomObject:Lead` into `force-app`: it pulls every standard field.
14. **Run Jest tests** — `npm run lint && npm run test:unit`. Skip only if the plan says the story has no LWC.
14b. **Run Code Analyzer on the story's files** — Scan only the files this story created or changed (the build plan rows, including test classes and LWC bundles). Keep the workspace as `force-app` so cross-file rules still work:
    ```sh
    sf code-analyzer run --workspace force-app \
      --target <file-or-folder> --target <file-or-folder> ... \
      --severity-threshold 1 --view table
    ```
    The project's `code-analyzer.yml` applies. Zero Sev 1 is required: fix every Sev 1 in a story file before deploying. Fix Sev 2 if the fix is small and stays inside the plan. Otherwise list it with a reason. Report Sev 3+ counts only. Don't fix findings in files the story didn't touch. Note them for the developer instead.
15. **Deploy to dev** — `playbook-dev` is shared by all story worktrees, so source tracking reports conflicts. First compare the org's copy of each changed component with git (retrieve into a scratch folder and diff). If the org only has another story's or an older version, deploy with:
    `sf project deploy start --source-dir force-app -o playbook-dev --ignore-conflicts -w 30`
    Never deploy from tracking (`sf project deploy start` with no source) after retrieving `CustomObject:Lead`: it tries to delete standard fields. If the org has a change git doesn't, stop and ask.
16. **Run Apex tests** — Run the story's test classes with coverage: `sf apex run test -o <dev-org> -n <TestClass> -c -r human -w 30`. New classes need 90% or more.
17. **Run UI tests** — Hand off to `ui-test-playwright` in `story` mode. Skip only if the plan says the story has no UI.
18. **Fix and re-test** — On any failure, read the error, fix the cause, and re-run from the failed step. Fix the code, not the test, unless the test is wrong against the acceptance criteria. Say which one you changed and why.
19. **Build complete** — HUMAN GATE. Show the summary below. Stop and wait for the developer to confirm every test is green.

## Rules
- Bulk-safe code. No SOQL or DML in loops. Tests insert 200 records.
- Queries in a selector `WITH USER_MODE`. DML in user mode.
- An exception approved in `architecture.md` overrides these generic rules (for example S1 runs on demand, not from the trigger; S3 saves in system mode). Follow the approved design and name the exception in the summary.
- New custom fields: add read (or edit, per the plan) access in `Sales_Rep_Base` **and** read-only FLS on the Admin profile, and put them on `Lead-Lead Layout`. Metadata deploys give no profile access by default.
- Lead logic goes in a service class, called from `LeadTriggerHandler`. No logic in triggers.
- Reuse `LeadDataStandardizer`, `LeadSelector` and `TestDataFactory`. Don't rebuild what exists.
- Every acceptance criterion maps to at least one test. Use the plan's test table.
- Demo data only. No `System.debug` of lead data.
- Never deploy to QA, UAT or production. Dev org only.
- No deploy to dev with an open Sev 1 in a story file (step 14b).
- Do not commit or open a PR. That is `pr-package` (step 20).

## Jira
- Site cloudId `21647882-f209-44e2-96aa-f5b9a20c2d31`, project `SCRUM`. Epic BR-014 = SCRUM-5. BR014-S1 = SCRUM-6, S2 = SCRUM-7, S3 = SCRUM-8.
- Statuses, in order: To Do → In Progress → In Review → In QA → QA Accepted → Ready for Release → Done.
- Move a story with `transitionJiraIssue`. Pick the transition by its target status (`getTransitionsForJiraIssue`). IDs: To Do `11`, In Progress `21`, In Review `2`, In QA `3`, QA Accepted `4`, Ready for Release `5`, Done `31`. Skip the move if the story is already there or further along.
- If the story is still **To Do**, move it to **In Progress** before step 12.
- At step 19, after the developer confirms green, add a comment with the summary table below. The status stays **In Progress**. `pr-package` moves it to **In Review**.

## Output
Update the feature `README.md` "Components changed" table. Then report:

| Check | Result |
|---|---|
| Components built | List vs. the build plan |
| Lint + Jest | Pass/fail, or skipped (why) |
| Code Analyzer | Files scanned; Sev 1 / Sev 2 / Sev 3+ counts; Sev 2 left open (why) |
| Dev deploy | Pass/fail, deploy ID |
| Apex tests | Pass count, coverage per new class |
| UI test | Pass/fail, or skipped (why) |
| AC coverage | Each AC → test name |
| Fixes made | What failed, what changed |

Hands off to `pr-package` (step 20).
