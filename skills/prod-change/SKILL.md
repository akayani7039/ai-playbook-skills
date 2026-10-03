---
name: prod-change
description: Take a signed-off release to production - validate against prod, write the runbook and rollback plan, send the approver brief, quick deploy, smoke test, merge and tag main, announce and close tickets. Use after UAT sign-off. Pipeline steps 42-50.
---

# prod-change

Stage 07 — Production change (steps 42-50).

## Tools
- Salesforce CLI (`sf project deploy validate`, `sf project deploy quick`)
- Git MCP, Slack MCP, Jira MCP
- `ui-test-playwright` (prod-smoke mode)
- Official skill: `platform-metadata-deploy`

## Steps
42. **Validate production** — Check-only deploy with tests against prod. Save the validation ID.
43. **Generate runbook + rollback plan** — Deploy steps and how to roll back.
44. **Send approver brief** — Test results, UAT sign-off, runbook and rollback plan.
45. **Authorize production** — HUMAN GATE. Stop and wait for the change approver.
46. **Quick deploy** — Tech lead runs the quick deploy using the validation ID. Tests are not re-run.
47. **Run production smoke test** — `ui-test-playwright` prod-smoke.
48. **Merge + tag** — Merge the release branch into `main`. Tag `prod-<yyyy.mm>`.
49. **Publish + announce** — Publish release notes. Tell stakeholders it's live.
50. **Close tickets** — Move every story in the release to **Done**.

## Rules
- Never deploy to production without approval at step 45.
- The tech lead runs step 46, not the agent.

## TODO
- [ ] Runbook and rollback templates
- [ ] Approver channel and brief format
- [ ] Tag naming
- [ ] Rollback method (destructive changes, backup retrieve)
