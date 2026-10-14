---
name: release-train
description: Run the release train - summarize blockers, cut and freeze the release branch, deploy to UAT, run full regression, write release notes and announce UAT, then hold for business owner sign-off. Use at code freeze. Pipeline steps 35-41.
---

# release-train

Stage 06 — Release train + UAT (steps 35-41).

## Tools
- Jira MCP, Git MCP, GitHub Actions, Slack MCP
- Slack channel: `#br-014-release` (C0C966P3VB8). Post every Slack message there.
- `ui-test-playwright` (regression mode)
- Official skill: `platform-metadata-deploy`

## Setup
- Repo `akayani7039/ai-playbook-app`. Release workflows run **from `main` only** (`--ref main`); every other ref skips the job.
  - `03-release-preparation.yml` (freeze): environment `release-freeze`, approved by `techlead10`.
  - `04-uat-deployment.yml` (UAT + sign-off): environment `uat` (`akayani7039`), then `uat-signoff` (business owner `akayani7039`, gate 4).
- Release branch `release/<yyyy.mm>`, freeze tag `freeze-<yyyy.mm>`. Workflow 03 creates both on the tested `develop` commit.

## Pipeline checks (before step 36)
Run `git fetch --prune --tags origin`, then check all of these. Stop on any failure, say which one, and how to fix it:
1. **`main` is inside `develop`:** `git merge-base --is-ancestor origin/main origin/develop`. UAT (workflow 04) checks that `origin/main` is an ancestor of the frozen commit, so a freeze without it fails in UAT. Fix: push `sync/main-to-develop-<YYYY-MM-DD>` from `origin/main` and open a PR into `develop`. The user merges it, then QA re-runs on the new `develop` head.
2. **Same CI on both branches:** `git diff --quiet origin/main origin/develop -- .github scripts/ci tests/ci`. If they differ, a CI fix reached only one branch. Show the diff. Get it into both (see `pr-package`, "CI fixes reach main too").
3. **QA run matches `develop`:** the latest successful `QA deployment` run (`gh run list -w 02-qa-deployment.yml -b develop -s success -L 1 --json databaseId,headSha`) has `headSha` = `origin/develop`. Otherwise workflow 03 fails with "develop advanced". Wait for, or re-run, QA.
4. **Month is free:** no `release/<yyyy.mm>` branch and no `freeze-<yyyy.mm>` tag on origin (`git ls-remote --heads --tags origin`). If one exists, the last run wasn't reset. Point to `pipeline-reset`.
5. **No open PRs into `develop` or `main`** that belong in this release (`gh pr list --base develop`, `--base main`).

## Steps
35. **Blocker summary** — Summarize open defects and blockers for go/no-go. Every story in the release must be **Ready for Release** in Jira. List any that aren't as blockers. Include the pipeline check results.
36. **Cut + freeze branch** — Run workflow 03 from `main`:
    `gh workflow run 03-release-preparation.yml --ref main -f release_month=<yyyy.mm> -f qa_run_id=<id from check 3> -f jira_keys="SCRUM-6 SCRUM-7 SCRUM-8" -f blocker_review=<link to the step 35 Slack post>`
    The run waits on `release-freeze`. Tell the tech lead (`techlead10`) to approve it in GitHub. Wait, then confirm `release/<yyyy.mm>` and `freeze-<yyyy.mm>` exist on the same commit.
37. **Deploy to UAT** — Run workflow 04 from `main`: `gh workflow run 04-uat-deployment.yml --ref main -f release_branch=release/<yyyy.mm>`. `akayani7039` approves the `uat` environment. The job deploys, runs Apex (`RunLocalTests`), Jest and the UI regression.
38. **Full regression** — Report workflow 04's results. Its UI regression runs only the base suite (`tests/ui/regression`), so also run the story UI tests against UAT by hand: `ui-test-playwright` with `tests/ui/story` and `TARGET_ORG=playbook-uat`. UAT needs sample data first: check for the 12 sample leads, and load them with `npm run data:load -- --target-org playbook-uat` if they're missing.
39. **Generate release notes** — From Jira and the merged PRs.
40. **Announce release** — Tell stakeholders in `#br-014-release` what's in it and when UAT opens.
41. **Business sign-off** — HUMAN GATE. Stop and wait. The business owner accepts all stories in the requirement together by approving the `uat-signoff` environment on workflow 04's run. Never approve it yourself.

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

## Rules
- Never freeze with a failed pipeline check.
- Never approve `release-freeze`, `uat` or `uat-signoff` yourself. Those are human gates.
- Never push to `main`, `develop` or `release/*` directly.

## TODO
- [ ] Release notes template
