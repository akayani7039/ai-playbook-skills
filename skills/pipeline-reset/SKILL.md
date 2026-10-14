---
name: pipeline-reset
description: Reset the BR-014 pipeline demo to a clean start (every story in To Do, only the base project everywhere) from any stage, including after UAT and production - back up story and release work, revert story merges out of develop and the release merge out of main with PRs, remove story components from the dev, QA, UAT and demo production orgs, delete the release branch and freeze/prod tags, remove story worktrees and branches, and move the stories and epic back to To Do. Keeps all GitHub setup (rulesets, workflows, environments, secrets, CI fixes). Use when the user says "reset the pipeline", "revert back to main", "start the demo again" or "clean up the stories". Run it again after each PR it opens is merged, until it reports clean.
---

# pipeline-reset

Puts the demo back to "every story in To Do, only the base project in every org and on `main` and `develop`" so the pipeline can be run again from `story-setup`. Works from any stage: build, QA, release train, UAT or production.

The reset changes `develop` and `main` only through PRs the user merges, so it takes more than one run. **Every run starts by taking stock and does only what the current state needs.** Run it again after each merge until step 1 reports clean.

| State found in step 1 | This run does |
|---|---|
| Story work anywhere, no revert PRs yet | Steps 2–8: back up, open revert PRs, clean orgs, release refs, local work and Jira |
| A revert PR is open | Nothing new. Tell the user which PR to merge |
| Revert PRs merged, but `main` isn't inside `develop` | Step 9: open the sync PR from `main` into `develop` |
| Everything merged and in sync | Step 10: finish and report clean |

## Setup

| Item | Value |
|---|---|
| App repo | `~/Desktop/Claude Projects/AIPlaybookDemo/app/ai-playbook-app` (GitHub `akayani7039/ai-playbook-app`) |
| Worktree root | `~/Desktop/Claude Projects/AIPlaybookDemo/app/ai-playbook-worktrees/` |
| Story branches | `feature/BR014-S<n>` |
| Release refs | Branch `release/<yyyy.mm>`, tags `freeze-<yyyy.mm>` and `prod-<yyyy.mm>` |
| Orgs to clean | `playbook-dev`, `playbook-qa`, `playbook-uat`, `playbook-prod` (demo production: a scratch org from Dev Hub `devhub2`) |
| Jira | Site `asadkayani10-1791505782400.atlassian.net`, project `SCRUM`, epic SCRUM-5 and its child stories, fix version `<yyyy.mm>` |

## Never touch
- `main` and `develop` as branches. Never delete them, force-push them or reset them on GitHub. Change them only through PRs the user merges.
- GitHub rulesets, workflows (`.github/`), environments, secrets, variables and repo settings.
- Commits that are not story work: CI fixes (`fix/ci-*`), pipeline syncs (`sync/*`, `fix/ci-sync-main`). Reverts must remove story changes only.
- The base project in any org: the components that exist on `main` before the release merge.
- Sample data (`scripts/data`) in any org. Story fields are deleted with their data; base Lead records stay.
- A **real** production org. `playbook-prod` must be a scratch org (`sf org display -o playbook-prod --json` shows an expiration date). If it isn't, **stop**: real production needs its own reviewed rollback through `prod-change`.
- The `ai-playbook-skills` repo and the Dev Hubs (`devhub`, `devhub2`).

## Tools
- `git`, `gh`, Salesforce CLI with `sfdx-git-delta` (`sf sgd source delta`).
- Jira MCP: `searchJiraIssuesUsingJql`, `getJiraIssue` (comments: `fields: ["comment"]`), `getTransitionsForJiraIssue`, `transitionJiraIssue`. It has no tools for versions or deleting comments; those are done by hand in the Jira UI.
- Plugin hook: the salesforce-development plugin denies raw `sf data query` and `sf project retrieve` until `platform-soql-query` / `platform-metadata-retrieve` has been dispatched in the same turn. Dispatch it first, then run the command.
- `AskUserQuestion` for every decision.

## Permission blocks
Claude Code may block deleting branches or tags on GitHub and deploying to an org named `playbook-prod`, even after the user agrees. When that happens, don't work around it. Show the exact command and ask the user to run it in the prompt with the `!` prefix, then check the result.

## Steps

### 1. Take stock (read only)
Run `git fetch --prune --tags origin` in the app repo, then gather:
- Local and remote branches, worktrees (`git worktree list`) and tags, including `release/*`, `freeze-*`, `prod-*` and `backup/*`.
- PRs: `gh pr list --state all --limit 100 --json number,title,headRefName,baseRefName,state,mergeCommit,mergedAt`.
  - **Story merges into `develop`:** merged `feature/BR014-*` PRs.
  - **Release merges into `main`:** merged PRs from `release/*` into `main`. Open ones too.
  - **Revert PRs:** from `revert/*` branches into `develop` or `main`. Open ones mean "wait"; merged ones mean a later run.
- **Pipeline state:** is `origin/main` an ancestor of `origin/develop` (`git merge-base --is-ancestor origin/main origin/develop`)? Do `main` and `develop` have the same `.github` and `scripts/ci` files?
- **Org state:** for each org, which story components exist. For each story merge `M` into `develop` the story's changes are `M^1..M^2`; for the release merge `R` into `main` the production changes are `R^1..R^2`. Query the org for the added components (classes, LWC bundles, custom fields).
- **Jira:** the epic, its child stories, their status and comments, and whether fix version `<yyyy.mm>` (id `10000`) is marked released (`getJiraIssue` returns `fixVersions[].released`). Also list stray issues in the project that aren't under the epic (for example SCRUM-4) for the user to delete by hand.

Pick the row from the state table above. Show a short table: what exists, what this run will remove or revert, and what stays.

### 2. Confirm
Ask once, with these options:
- Reset everything shown (Recommended)
- Choose what to reset (then ask per area: develop, main, release refs, orgs, local, Jira)
- Cancel

Deleting custom fields removes their data; name them in the question. Say which steps need the user to run a command (see Permission blocks).

### 3. Back up
Tag every tip you're about to remove, locally: `backup/<name>-<YYYY-MM-DD>`, for example `backup/BR014-S1-2026-10-13`, `backup/release-2026.10-2026-10-13`, `backup/prod-2026.10-2026-10-13`. Don't push backup tags.

### 4. Revert story merges out of develop
Skip if no story merge is in `develop`.
1. Create worktree `ai-playbook-worktrees/revert-develop` on a new branch `revert/develop-reset-<YYYY-MM-DD>` from `origin/develop`.
2. Revert each story merge, newest first: `git revert --no-edit -m 1 <merge>`. Squash into one commit: `Revert BR-014 stories to reset the pipeline demo`.
3. Check the diff against `origin/develop`. It may only remove or restore story files (including `features/BR014-S*`). If it touches `.github/`, `scripts/ci/`, `tests/ci/` or any CI fix, stop and show the user.
4. Push and open a PR into `develop`. In the body, list the reverted PRs, the backup tags, and this note:
   > PR validation is expected to fail with "Deletion detected". The pipeline blocks destructive changes by design. Org cleanup is done separately by `pipeline-reset`. Merge with an admin bypass of the `develop` ruleset.
5. Don't merge it. The user merges it.

### 5. Revert the release out of main
Skip if no release PR was merged into `main`. If a release PR into `main` is still **open**, ask whether to close it instead (`gh pr close`), and skip the revert.
1. Create worktree `ai-playbook-worktrees/revert-main` on a new branch `revert/main-reset-<YYYY-MM-DD>` from `origin/main`.
2. For each merged release PR, newest first: `git revert --no-edit -m 1 <release merge>`.
3. Check the diff against `origin/main`. It may only remove or restore story files. `main` must end with the same content as before the first release merge (`git diff <first release merge>^1 HEAD` is empty except CI fixes made on `main` since). Stop on anything else.
4. Push and open a PR into `main` titled `Revert release <yyyy.mm> to reset the pipeline demo`. Say that `main` must match production, which step 6 also resets.
5. Don't merge it. The user merges it.

### 6. Clean the orgs
For each org that has story components:

| Org | Story changes come from |
|---|---|
| `playbook-dev`, `playbook-qa`, `playbook-uat` | Each story merge into `develop` (`M^1..M^2`), or each unmerged story branch against `origin/develop` |
| `playbook-prod` | Each release merge into `main` (`R^1..R^2`). If the release reached production but wasn't merged into `main`, use `origin/main..prod-<yyyy.mm>` |

1. Build the delta in a scratch folder: `sf sgd source delta --from <before> --to <after> --output-dir <scratch>/delta-<org>`.
   - **Added** components → `destructiveChanges.xml`.
   - **Changed** components that existed before (layout, permission set, flexipage, the Lead object's history setting) → redeploy them from the "before" commit.
   - **Changed** components that don't exist in the "before" commit (for example a profile file holding only story field permissions) → skip. Deleting the fields removes their permissions.
2. **Custom fields and objects:** deleting them removes their data and field history, and leaves them in the org's recycle bin. Confirmed in step 2.
3. Deploy the restored "before" versions and the deletions **in one deploy**: `sf project deploy start --manifest <restored package.xml> --post-destructive-changes <destructiveChanges.xml> -o <org>`. Restoring the layout, page and permission set first removes their references, so the fields, classes and LWC can be deleted. Run tests only if the org requires them.
4. For `playbook-prod`, check first that it's the demo scratch org (see Never touch). The deploy may need the user to run it (see Permission blocks).
5. Check with queries per org: no story classes (`SELECT Name FROM ApexClass WHERE Name IN (...)`), no story LWC, no story fields (Tooling `CustomField` on Lead), and the base classes still there. Report 0 story components left per org.

Never edit or delete files in the app repo for org cleanup.

### 7. Remove release refs and local story work
1. **Release refs on GitHub:** after the backup tags exist, delete `release/<yyyy.mm>`, `freeze-<yyyy.mm>` and `prod-<yyyy.mm>`: `git push origin --delete refs/heads/release/<yyyy.mm> refs/tags/freeze-<yyyy.mm> refs/tags/prod-<yyyy.mm>`. Workflow 03 refuses to freeze a month whose branch or tag already exists. Delete the same tags locally.
2. **Story worktrees and branches** (`feature/BR014-S*`): check `git status --short` in each. If there are uncommitted changes, show them and ask before removing. Then `git worktree remove <path>` and `git branch -D <branch>`.
3. Delete **merged** `feature/BR014-*` branches on GitHub. Ask first about unmerged ones.
4. Remove merged `fix/*` and `sync/*` worktrees and branches only if the user asks. Check with `git merge-base --is-ancestor <branch> origin/develop` first.

### 8. Reset Jira
1. Move each story under the epic to **To Do**, from whatever status it's in (including **Done**). Find the transition by its target status with `getTransitionsForJiraIssue` (To Do = `11`).
2. Move the epic to **To Do** if it has moved.
3. If fix version `<yyyy.mm>` is marked released, tell the user to mark it unreleased in the Jira UI (Project settings → Releases). The MCP can't.
4. Don't change the acceptance criteria, the fix version on the stories or the sprint.
5. The Jira tools can't delete comments. List each pipeline comment (issue key, date, first line) so the user can delete them by hand.

Then report (see Output) and stop. Tell the user which revert PRs to merge, and to run `pipeline-reset` again.

### 9. Sync main into develop (later run)
When the revert PRs are merged but `origin/main` isn't an ancestor of `origin/develop`:
1. Push a branch `sync/main-to-develop-<YYYY-MM-DD>` from `origin/main`. Don't open the PR from `main` itself: the ruleset needs an approval from someone other than the last pusher.
2. Open a PR into `develop`. Both branches already have the same files, so it only joins the history. Without it, the next `release-train` freeze fails its "main is inside develop" check.
3. Don't merge it. Tell the user to merge it and run `pipeline-reset` again.

### 10. Finish (last run)
When every revert and sync PR is merged:
1. Check `origin/develop` and `origin/main` have no story files: no `features/BR014-S*` folders and none of the story components.
2. Check `origin/main` is an ancestor of `origin/develop`, and their `.github` and `scripts/ci` files match.
3. Remove the `revert-*` worktrees. Delete the merged `revert/*` and `sync/*` branches locally and on GitHub.
4. Update local `develop`: `git fetch origin && git branch -f develop origin/develop` (only if `develop` isn't checked out anywhere; otherwise `git pull --ff-only` in that checkout). Leave `main` checked out in the app repo.
5. Re-check all four orgs. The QA deployment that runs on the `develop` merge doesn't delete metadata, so confirm the story components are still gone everywhere.
6. Check Jira: every story and the epic in **To Do**.

## Rules
- Back up before deleting anything. Never delete a branch or tag without a backup tag.
- Destructive steps happen only after the user confirms in step 2.
- Never use `--force` on a push, `reset --hard` on a shared branch, or a ruleset bypass yourself.
- Never deploy to `playbook-prod` unless step 1 confirmed it's the demo scratch org.
- If something doesn't match what you expected (an unknown branch, a non-story commit in a revert diff, a component you can't place), stop and ask.
- Report facts. If a step failed, was skipped or was handed to the user, say so.

## Output

| Area | Result |
|---|---|
| Backups | Tags created |
| develop | Revert PR link and expected CI result, or "nothing merged" |
| main | Release revert PR link, closed release PR, or "no release in main" |
| Orgs | Per org (dev, QA, UAT, prod): components removed and restored, with the 0-left check |
| Release refs | `release/*`, `freeze-*`, `prod-*` deleted, or the command handed to the user |
| Local | Worktrees and branches removed; remote branches deleted |
| Jira | Stories and epic moved to To Do; version unreleased if asked; comments to delete by hand |
| Kept | GitHub setup, CI fixes, base project, sample data, Dev Hubs |
| Next step | Which PRs to merge, then run `pipeline-reset` again. When it reports clean: `story-setup`. |
