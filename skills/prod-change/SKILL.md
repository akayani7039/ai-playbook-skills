---
name: prod-change
description: Take a signed-off release to production - validate against prod, write the runbook and rollback plan, send the approver brief, quick deploy, smoke test, merge and tag main, announce and close tickets. Use after UAT sign-off. Pipeline steps 42-50.
---

# prod-change

Stage 07 — Production change (steps 42-50).

## Tools
- Salesforce CLI (`sf project deploy validate`, `sf project deploy quick`)
- Git MCP, Slack MCP, Jira MCP
- Slack channel: `#br-014-release` (C0C966P3VB8). Post every Slack message there.
- `ui-test-playwright` (prod-smoke mode)
- Official skill: `platform-metadata-deploy`

## Steps
42. **Validate production** — Check-only deploy with tests against prod. Save the validation ID.
43. **Generate runbook + rollback plan** — Deploy steps and how to roll back.
44. **Send approver brief** — Test results, UAT sign-off, runbook and rollback plan, posted to `#br-014-release`.
45. **Authorize production** — HUMAN GATE. Stop and wait for the change approver.
46. **Quick deploy** — Tech lead runs the quick deploy using the validation ID. Tests are not re-run.
47. **Run production smoke test** — `ui-test-playwright` prod-smoke.
48. **Merge + tag** — Merge the release branch into `main`. Tag `prod-<yyyy.mm>`.
49. **Publish + announce** — Publish release notes. Tell stakeholders in `#br-014-release` it's live.
50. **Close tickets** — Move every story in the release from **Ready for Release** to **Done**, with a comment: production tag, deploy time, smoke result. When every story in the epic is **Done**, move the epic to **Done**.

## Jira
- Site cloudId `21647882-f209-44e2-96aa-f5b9a20c2d31`, project `SCRUM`. Epic BR-014 = SCRUM-5. BR014-S1 = SCRUM-6, S2 = SCRUM-7, S3 = SCRUM-8.
- Statuses, in order: To Do → In Progress → In Review → In QA → QA Accepted → Ready for Release → Done.
- Move a story with `transitionJiraIssue`. Pick the transition by its target status (`listJiraIssueTransitions`). Skip the move if the story is already there or further along.
- Before step 42, check every story in the release is **Ready for Release**. Stop if one isn't.
- Step 45: add the change approver's decision as a comment on each story.
- Don't move anything to **Done** before the production smoke test (step 47) passes.

## Rules
- Never deploy to production without approval at step 45.
- The tech lead runs step 46, not the agent.

## TODO
- [ ] Runbook and rollback templates
- [ ] Approver channel and brief format
- [ ] Tag naming
- [ ] Rollback method (destructive changes, backup retrieve)
