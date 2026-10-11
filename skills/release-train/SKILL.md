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
35. **Blocker summary** — Summarize open defects and blockers for go/no-go.
36. **Cut + freeze branch** — Cut `release/<yyyy.mm>` from `develop` at code freeze. Tech lead confirms.
37. **Deploy to UAT** — Deploy the release branch (GitHub Actions).
38. **Full regression** — Run full Apex, Jest and Playwright suites.
39. **Generate release notes** — From Jira and the merged PRs.
40. **Announce release** — Tell stakeholders in `#br-014-release` what's in it and when UAT opens.
41. **Business sign-off** — HUMAN GATE. Stop and wait. The business owner accepts all stories in the requirement together.

## Output
A frozen release branch, tested in UAT, with release notes and business sign-off.

## TODO
- [ ] Release branch naming
- [ ] Release notes template
- [ ] How sign-off is recorded
