---
name: pr-package
description: Package a finished story as a pull request - commit with the story key, open the PR into develop, write the PR summary, link Jira, run CI checks and post results. Also merges QA-accepted stories and updates Jira and Slack. Pipeline steps 20-27 and 33-34.
---

# pr-package

Stage 04 — Commit, CI & review (steps 20-27) and Stage 05 — QA handoff (steps 33-34).

## Tools
- Git, GitHub CLI (`gh`), Jira MCP, Slack MCP
- Slack channel: `#br-014-release` (C0C97HAD3S4). Post every Slack message there.
- GitHub Actions, sfdx-git-delta, Code Analyzer
- Official skills: `dx-code-analyzer-run`, `dx-code-analyzer-configure`, `experience-lwc-security-validate`

## Setup
- Repo: `akayani7039/ai-playbook-app`. PRs go into `develop`.
- Tech lead on GitHub: `techlead10`.
- CI workflow: `PR validation` (`.github/workflows/01-pr-validation.yml`). Required check: `validate`.
- `develop` ruleset: 1 approval after the last push, `validate` must pass, no force push.
- Jira site cloudId: `21647882-f209-44e2-96aa-f5b9a20c2d31`. Project `SCRUM`.
- Jira statuses, in order: To Do → In Progress → In Review → In QA → QA Accepted → Ready for Release → Done.
- Story keys: BR014-S1 = SCRUM-6, BR014-S2 = SCRUM-7, BR014-S3 = SCRUM-8. Check the feature `README.md` if unsure.

## Pick the part
- No open PR for `feature/BR014-S<n>` → Part A.
- PR open and CI green, tech lead approved, story **QA Accepted** in Jira → Part B.
- PR merged, story deployed to QA, Jira not yet **In QA** → QA handoff (below).
- Anything else → report where the story is and stop.

## Part A — Open the PR (steps 20-27)

### Before you start
- Work in the story's worktree on `feature/BR014-S<n>`.
- `architecture.md` says `Approved`, and `story-build` finished (step 19).
- Run the checks from the project `CLAUDE.md`. Stop on any failure:
  ```sh
  npm run lint && npm run test:unit
  sf project deploy start --dry-run --source-dir force-app -o <dev-org> --test-level RunLocalTests -w 30
  sf code-analyzer run --workspace force-app --target <story files> --severity-threshold 1
  ```

### Steps
20. **Create commit** — Message: `BR014-S<n>: <what changed>`. Skip if the tree is clean and the story's commits already exist.
21. **Open pull request** — Push the branch. `gh pr create --base develop --reviewer techlead10`. Title: `BR014-S<n>: <story title>`.
22. **Generate PR summary** — Fill `.github/pull_request_template.md`: story and Jira link, what changed and why, design match, test evidence table, one row per acceptance criterion with its test method name, security, deploy order. Use real numbers from the checks above.
23. **Link ticket** — Add the PR as a web link on the Jira story (`createJiraIssueRemoteIssueLink`). Check existing links first; don't add it twice. Move the story to **In Review**.
24. **Run CI validation** — Wait for the `PR validation` run on the PR (`gh run watch`).
25. **Run Code Analyzer** — CI runs it. Zero Sev 1 is required.
26. **Check deployment delta** — CI builds `build/package.xml` with sfdx-git-delta. Check it lists only the story's components.
27. **Post CI feedback** — One PR comment and one Slack post: pass/fail per check, test count, coverage of new classes, Sev 1/Sev 2 counts, validation ID.

### When CI fails
- Download the evidence: `gh run download <run-id>`. Read `evidence/pr-validation.json` and `evidence/code-analyzer.json`.
- **Code failure** (compile error, test failure, Sev 1 in a story file): report it. Fix only if it's inside the build plan, then push and re-check.
- **CI setup failure** (auth, missing folder, tracking, empty QA org): don't touch the story branch. Fix CI on its own branch from `develop` and its own PR. Merge `develop` into the story branch after it lands, so CI re-runs.
- An auth secret is fixed by the user. Show the command; never read or print the value:
  `sf org auth show-sfdx-auth-url -o <alias> --json | jq -r '.result.sfdxAuthUrl' | gh secret set SFDX_AUTH_URL --env <env> -R akayani7039/ai-playbook-app`

Then hand off to `sf-code-review` (step 28) and the tech lead (step 29, HUMAN GATE).

## QA handoff (steps 30-31)
When the `QA deployment` workflow is green on a `develop` commit that contains the story:
- Move the story to **In QA**.
- Add a Jira comment: the commit, the QA deployment run, the smoke result, and the org (`playbook-qa`).
- Post the same to `#br-014-release`, and ask the QA lead to test.
The QA lead then moves it to **QA Accepted** (gate 3), or logs a Bug and moves it back to **In Progress**.

## Part B — After QA acceptance (steps 33-34)

### Check the gates. All must pass, or stop and say which one failed.
1. **Gate 2, tech lead** — Latest review from `techlead10` is `APPROVED`, and it's after the `sf-code-review` review (step 28).
2. **CI** — `validate` is green on the PR's current head.
3. **Gate 3, QA lead** — The Jira story is in **QA Accepted**. Read the changelog (`listJiraIssueChangelogs`): the move to QA Accepted must be made by the QA lead, not the developer or the agent. If it isn't, stop.

### Steps
33. **Merge to develop** — `gh pr merge <n> --merge`. Keep the branch; the worktree uses it. The push starts the `QA deployment` workflow. Wait for it and report the result. If the story wasn't in QA yet, run the QA handoff.
34. **Update Jira + Slack** — Move the story to **Ready for Release**. Add a Jira comment: merge commit, QA deployment run and result. Post the same to `#br-014-release`.

### If the PR is already merged
Don't merge again. Report who merged it and when, and whether gates 2 and 3 were in place at that time. Record it in a Jira comment and Slack. Move to **Ready for Release** only if the story is QA Accepted.

## Rules
- Never approve a PR, never accept a story, never move a story to QA Accepted. Those are human gates.
- Never merge without all three gate checks.
- Never print, log or commit auth URLs or tokens.
- Don't post to Slack or Jira for steps you didn't run.

## TODO
- [ ] QA order: the `QA deployment` workflow deploys `develop`, so a story reaches QA only after step 33. Decide: merge first then QA, or add a pre-merge QA deploy of the PR branch.
- [ ] QA lead Jira account, so gate 3 can be checked by name.
