---
name: release-train
description: Run the release train - summarize blockers, cut and freeze the release branch, deploy to UAT, run full regression, write release notes and announce UAT, then hold for business owner sign-off. Use at code freeze. Pipeline steps 35-41.
---

# release-train

Stage 06 — Release train + UAT (steps 35-41).

## Tools
- Jira MCP, Git MCP, GitHub Actions, Slack MCP
- Slack channel: `#br-014-release` (C0C97HAD3S4). Post every Slack message there.
- `ui-test-playwright` (regression mode)
- Official skill: `platform-metadata-deploy`

## Steps
35. **Blocker summary** — Summarize open defects and blockers for go/no-go. Every story in the release must be **Ready for Release** in Jira. List any that aren't as blockers.
36. **Cut + freeze branch** — Cut `release/<yyyy.mm>` from `develop` at code freeze. Tech lead confirms.
37. **Deploy to UAT** — Deploy the release branch (GitHub Actions).
38. **Full regression** — Run full Apex, Jest and Playwright suites.
39. **Generate release notes** — From Jira and the merged PRs.
40. **Announce release** — Tell stakeholders in `#br-014-release` what's in it and when UAT opens.
41. **Business sign-off** — HUMAN GATE. Stop and wait. The business owner accepts all stories in the requirement together.

## Jira
- Site cloudId `21647882-f209-44e2-96aa-f5b9a20c2d31`, project `SCRUM`. Epic BR-014 = SCRUM-5. BR014-S1 = SCRUM-6, S2 = SCRUM-7, S3 = SCRUM-8.
- Statuses, in order: To Do → In Progress → In Review → In QA → QA Accepted → Ready for Release → Done.
- Move a story with `transitionJiraIssue`. Pick the transition by its target status (`listJiraIssueTransitions`). Skip the move if the story is already there or further along.
- The release takes only stories in **Ready for Release**. Don't move stories forward to make them fit.
- Step 39: build the release notes from the **Ready for Release** stories and their linked PRs.
- Step 41: after the business owner signs off, add a sign-off comment (who, when, release branch) to each story. They stay **Ready for Release** until production.
- If the business owner rejects a story, move it back to **In Progress** and comment why.

## Output
A frozen release branch, tested in UAT, with release notes and business sign-off.

## TODO
- [ ] Release branch naming
- [ ] Release notes template
- [ ] How sign-off is recorded
