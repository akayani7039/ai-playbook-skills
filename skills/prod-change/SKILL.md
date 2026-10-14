---
name: prod-change
description: Take a signed-off release to production - validate against prod, write the runbook and rollback plan, send the approver brief, quick deploy, smoke test, merge and tag main, announce and close tickets. Use after UAT sign-off. Pipeline steps 42-50.
---

# prod-change

Stage 07 — Production change (steps 42-50).

## Tools
- `gh` (GitHub Actions), Git, Slack MCP, Jira MCP
- Slack channel: `#br-014-release` (C0C966P3VB8). Post every Slack message there.
- `ui-test-playwright` (prod-smoke mode)
- Official skill: `platform-metadata-deploy`

## Setup
- Repo `akayani7039/ai-playbook-app`. Production workflows run **from `main` only** (`--ref main`).
  - `05-production-validation.yml`: environment `production-validation` (release owner `akayani7039`). Check-only deploy to `playbook-prod` with `RunLocalTests`. The validation is good for quick deploy for 9 days.
  - `06-production-deployment.yml`: environment `production` (change approver `techlead10`). Quick deploy, prod smoke, then job `finalize` pushes tag `prod-<yyyy.mm>` and opens the PR `release/<yyyy.mm>` → `main`.
- `playbook-prod` is the demo production scratch org (Dev Hub `devhub2`). Never touch a real production org.

## Steps
42. **Validate production** — Check the UAT run first: the latest `UAT deployment` run for the release is green and its `uat-signoff` job was approved. Then:
    `gh workflow run 05-production-validation.yml --ref main -f release_branch=release/<yyyy.mm> -f uat_run_id=<UAT run id>`
    `akayani7039` approves `production-validation`. Wait (`gh run watch`), then save the validation ID from the run summary or `evidence/` artifact.
43. **Generate runbook + rollback plan** — Deploy steps (workflow 06, quick deploy of the validation ID, smoke, tag, release PR). Rollback: deploy the previous `main` content with the story components in `destructiveChanges.xml`, built with `sf sgd source delta` from `origin/main` to the release commit; restore changed layouts, pages and permission sets from `origin/main`. Custom field data is lost on rollback; say so.
44. **Send approver brief** — Post to `#br-014-release`: release, stories, UAT and validation run links, test counts and coverage, UAT sign-off (who, when), runbook and rollback plan, validation expiry date.
45. **Authorize production** — HUMAN GATE. Stop and wait for the change approver (`techlead10`).
46. **Quick deploy** — The change approver runs it, not the agent. Show the command for them:
    `gh workflow run 06-production-deployment.yml --ref main -f release_branch=release/<yyyy.mm> -f validation_run_id=<workflow 05 run id>`
    and they approve the `production` environment. Tests are not re-run. Then watch the run.
47. **Run production smoke test** — Workflow 06 runs `tests/ui/prod-smoke`. Report it, and run `ui-test-playwright` prod-smoke locally only if asked.
48. **Merge + tag** — Workflow 06 pushed `prod-<yyyy.mm>` and opened the release PR into `main`. Check both exist. The user merges the PR (never the agent). After it's merged, `main` isn't inside `develop` until a sync PR (`sync/main-to-develop-<YYYY-MM-DD>` → `develop`); open it and say so.
49. **Publish + announce** — Post the release notes and "live in production" to `#br-014-release`: tag, deploy time, smoke result. Fix version `<yyyy.mm>` is marked released by hand in the Jira UI (the MCP can't).
50. **Close tickets** — Move every story in the release from **Ready for Release** to **Done**, with a comment: production tag, deploy time, smoke result. When every story in the epic is **Done**, move the epic to **Done**.

## Jira
- Site cloudId `21647882-f209-44e2-96aa-f5b9a20c2d31`, project `SCRUM`. Epic BR-014 = SCRUM-5. BR014-S1 = SCRUM-6, S2 = SCRUM-7, S3 = SCRUM-8.
- Statuses, in order: To Do → In Progress → In Review → In QA → QA Accepted → Ready for Release → Done.
- Move a story with `transitionJiraIssue`. Pick the transition by its target status (`getTransitionsForJiraIssue`). IDs: To Do `11`, In Progress `21`, In Review `2`, In QA `3`, QA Accepted `4`, Ready for Release `5`, Done `31`. Skip the move if the story is already there or further along.
- Before step 42, check every story in the release is **Ready for Release**. Stop if one isn't.
- Step 45: add the change approver's decision as a comment on each story.
- Don't move anything to **Done** before the production smoke test (step 47) passes.

## Rules
- Never deploy to production without approval at step 45.
- The change approver runs step 46, not the agent. Never approve `production-validation` or `production` yourself.
- Claude Code may block a deploy to `playbook-prod` or pushing tags. Don't work around it: show the command and ask the user to run it with the `!` prefix.
- Never merge the release PR into `main` yourself.
