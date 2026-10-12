---
name: pipeline-reset
description: Reset the BR-014 pipeline demo to a clean start so it can be run again - back up story work, revert story merges out of develop with a PR, remove story components from the dev, QA and UAT orgs, remove story worktrees and branches, and move the Jira stories back to To Do. Keeps all GitHub setup (rulesets, workflows, environments, CI fixes). Use when the user says "reset the pipeline", "revert back to main", "start the demo again" or "clean up the stories". Run it again after the revert PR is merged to finish.
---

# pipeline-reset

Puts the demo back to "only `main`, every story in To Do" so the pipeline can be tested again from `story-setup`.

Two runs:
- **Run 1 (reset):** back up, open the revert PR, clean orgs, local branches and Jira.
- **Run 2 (finish):** after the revert PR is merged, remove the revert branch and sync `develop`.

Detect which run this is in step 1.

## Setup

| Item | Value |
|---|---|
| App repo | `~/Desktop/Claude Projects/AI Playbook/ai-playbook-app` (GitHub `akayani7039/ai-playbook-app`) |
| Worktree root | `~/Desktop/Claude Projects/AI Playbook/ai-playbook-worktrees/` |
| Story branches | `feature/BR014-S<n>` |
| Orgs to clean | `playbook-dev`, `playbook-qa`, `playbook-uat` |
| Jira | Site `asadkayani10-1791505782400.atlassian.net`, project `SCRUM`, epic SCRUM-5 and its child stories |

## Never touch
- `main` and `develop` as branches. Never delete them, force-push them or reset them on GitHub. Change `develop` only through a PR.
- GitHub rulesets, workflows (`.github/`), environments, secrets and variables.
- Commits on `develop` that are not story merges (CI fixes such as `fix/ci-*`). The revert must remove story changes only.
- `playbook-prod`. If any story reached `main` or production, **stop** and tell the user. Production needs its own reviewed rollback (`prod-change`).
- The `ai-playbook-skills` repo.

## Tools
- `git`, `gh`, Salesforce CLI with `sfdx-git-delta` (`sf sgd source delta`).
- Jira MCP: `searchJiraIssuesUsingJql`, `listJiraIssueTransitions`, `transitionJiraIssue`, `listJiraIssueComments`.
- `AskUserQuestion` for every decision.

## Steps

### 1. Take stock (read only)
Run `git fetch --prune origin` in the app repo, then gather:
- Local and remote branches, worktrees (`git worktree list`) and tags.
- Story PRs: `gh pr list --state all --json number,title,headRefName,baseRefName,state,mergeCommit`. Find merged `feature/BR014-*` PRs into `develop`, any `release/*` branches, and anything merged into `main`.
- Revert PRs into `develop` from any `revert/*` branch. If one is **merged** and its branch or worktree still exists, this is run 2. Go to step 8. If one is still **open**, don't open another; tell the user to merge it first.
- Components from each story in each org. For each story merge commit `M`, the story's changes are `M^1..M^2`.
- Jira: the epic's child stories, their status, and their comments.

Show a short table: what exists, what will be removed or reverted, and what stays. **Stop if anything reached `main` or `playbook-prod`.**

### 2. Confirm
Ask once, with these options:
- Reset everything shown (Recommended)
- Choose what to reset (then ask per area: develop, orgs, local, Jira)
- Cancel

### 3. Back up
For every story branch or story merge, tag the tip locally: `backup/<branch-name>-<YYYY-MM-DD>` (for example `backup/BR014-S1-2026-10-11`). Don't push tags.

### 4. Revert story merges out of develop
Skip this if no story is merged into `develop`.
1. Create worktree `ai-playbook-worktrees/revert-pipeline-reset` on a new branch `revert/pipeline-reset-<YYYY-MM-DD>` from `origin/develop`.
2. Revert each story merge, newest first: `git revert --no-edit -m 1 <merge>`. Squash them into one commit: `Revert BR-014 stories to reset the pipeline demo`.
3. Check the diff against `origin/develop`. It may only remove or restore story files. If it touches `.github/`, `scripts/ci/`, `tests/ci/` or any CI fix, stop and show the user.
4. Push and open a PR into `develop`. In the body, list the reverted PRs, the backup tags, and this note:
   > PR validation is expected to fail with "Deletion detected". The pipeline blocks destructive changes by design. Org cleanup is done separately by `pipeline-reset`. Merge with an admin bypass of the `develop` ruleset.
5. Don't merge it. The user merges it.

### 5. Clean the orgs
For each org in `playbook-dev`, `playbook-qa` and `playbook-uat`, only if it has story components:
1. Build the story delta for each merge, or for each unmerged story branch against `origin/develop`:
   `sf sgd source delta --from <M^1> --to <M^2> --output-dir <scratch>/delta-<story>`
   - **Added** components (in `package/package.xml` and new in the diff) → put them in `destructiveChanges.xml`.
   - **Changed** components that existed before (a layout, permission set or flexipage) → redeploy them from the pre-story commit `M^1`. Don't delete them.
2. **Custom fields and objects:** deleting them removes their data and leaves them in the org's recycle bin. Ask before deleting them, and say which ones.
3. Deploy with an empty `package.xml` and `--post-destructive-changes`. Run tests only if the org requires them.
4. Check with a query, for example `SELECT Name FROM ApexClass WHERE Name IN (...)`. Report 0 left per org.

Do this from a scratch folder for the manifests. Never edit or delete files in the app repo for org cleanup.

### 6. Remove local story work
1. For each story worktree and branch (`feature/BR014-S*`, `release/*` from this demo): check `git status --short`. If there are uncommitted changes, show them and ask before removing.
2. `git worktree remove <path>` and `git branch -D <branch>`. Only after the backup tag exists.
3. Delete **merged** `feature/BR014-*` branches on GitHub (`git push origin --delete <branch>`). Ask first about unmerged ones.
4. Remove merged `fix/*` worktrees and local branches only if the user asks. Check with `git merge-base --is-ancestor <branch> origin/develop` first.

### 7. Reset Jira
1. Move each story under the epic to **To Do**. Find the transition by its target status with `listJiraIssueTransitions`.
2. Don't change the epic, the acceptance criteria, the fix version or the sprint.
3. The Jira tools can't delete comments. List each pipeline comment (issue key, date, first line) so the user can delete them by hand.

Then report (see Output) and stop. Tell the user to merge the revert PR and run `pipeline-reset` again.

### 8. Finish (run 2, after the revert PR is merged)
1. Check that `origin/develop` has no story files left: no `features/BR014-S*` folders and none of the story components from step 5.
2. `git worktree remove` the revert worktree. Delete the merged `revert/*` branch locally and on GitHub.
3. Update local `develop`: `git fetch origin && git branch -f develop origin/develop` (only if `develop` is not checked out anywhere; otherwise `git pull --ff-only` in that checkout).
4. Leave `main` checked out in the app repo.
5. Re-check the orgs: the QA deployment that runs on the merge does not delete metadata, so confirm the story components are still gone.

## Rules
- Back up before deleting anything. Never delete a branch without a backup tag.
- Destructive steps happen only after the user confirms in step 2.
- Never use `--force` on a push, `reset --hard` on a shared branch, or a ruleset bypass yourself.
- If something doesn't match what you expected (an unknown branch, a non-story commit in the revert diff, a component you can't place), stop and ask.
- Report facts. If a step failed or was skipped, say so.

## Output

| Area | Result |
|---|---|
| Backups | Tags created |
| develop | Revert PR link and expected CI result, or "nothing merged" |
| Orgs | Components removed and restored per org, with the 0-left check |
| Local | Worktrees and branches removed; remote branches deleted |
| Jira | Stories moved to To Do; comments to delete by hand |
| Kept | GitHub setup, CI fixes, `main`, `develop`, prod |
| Next step | Merge the revert PR with admin bypass, then run `pipeline-reset` again. After run 2: `story-setup`. |
