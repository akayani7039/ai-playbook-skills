---
name: pr-package
description: Package a finished story as a pull request - commit with the story key, open the PR into develop, write the PR summary, link Jira, run CI checks and post results. Also merges QA-accepted stories and updates Jira and Slack. Pipeline steps 20-27 and 33-34.
---

# pr-package

Stage 04 — Commit, CI & review (steps 20-27) and Stage 05 — QA handoff (steps 33-34).

## Tools
- Git MCP, GitHub MCP, Jira MCP, Slack MCP
- GitHub Actions, sfdx-git-delta, Code Analyzer
- Official skills: `dx-code-analyzer-run`, `dx-code-analyzer-configure`, `experience-lwc-security-validate`

## Part A — Open the PR (steps 20-27)
20. **Create commit** — Put the story key in the message (e.g. `BR014-S1: ...`).
21. **Open pull request** — Into `develop`.
22. **Generate PR summary** — What changed, why, and test evidence.
23. **Link ticket** — Link the PR to the Jira story. Move it to **In Review**.
24. **Run CI validation** — Check-only deploy with tests (GitHub Actions).
25. **Run Code Analyzer** — Block the merge on any Sev 1 finding.
26. **Check deployment delta** — sfdx-git-delta, so CI deploys only what changed.
27. **Post CI feedback** — To the PR and Slack.

Then hand off to `sf-code-review` (step 28) and the tech lead (step 29, HUMAN GATE).

## Part B — After QA acceptance (steps 33-34)
33. **Merge to develop** — Only if the QA lead accepted the story.
34. **Update Jira + Slack** — Move the story to **Ready for Release**. Notify the team.

## TODO
- [ ] Commit message format
- [ ] PR template
- [ ] Jira status names and transitions
- [ ] Slack channel
- [ ] GitHub Actions workflow name
