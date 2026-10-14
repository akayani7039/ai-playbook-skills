---
name: story-setup
description: Pick up Jira stories and prepare one isolated workspace per story (branch, git worktree, feature folder, pinned Salesforce dev org). Use at the start of a sprint or when the user says "pick up my stories", "set up story", or "start BR-xxx". Pipeline steps 1-3.
---

# story-setup

Stage 01 — Pick up & prepare (steps 1-3).

## Tools
- Jira MCP (`searchJiraIssuesUsingJql`, `getJiraIssue`, `getTransitionsForJiraIssue`, `transitionJiraIssue`)
- `git worktree`
- Salesforce CLI (`sf config set target-org`)
- Official skill: `dx-org-switch`

## Setup
| Item | Value |
|---|---|
| App repo | `~/Desktop/Claude Projects/AIPlaybookDemo/app/ai-playbook-app` (stays on `main`) |
| Worktree root | `~/Desktop/Claude Projects/AIPlaybookDemo/app/ai-playbook-worktrees/BR014-S<n>` |
| Branch | `feature/BR014-S<n>` from `origin/develop` |
| Feature folder | `features/BR014-S<n>/`, copied from `features/_template/` |
| Dev org | `playbook-dev` for every story (one shared scratch org) |
| Story keys | BR014-S1 = SCRUM-6, S2 = SCRUM-7, S3 = SCRUM-8 |

## Steps

### 1. Pick up story
- JQL: `project = SCRUM AND assignee = currentUser() AND status = "To Do" AND issuetype = Story ORDER BY key`.
- Show them grouped by epic (BR-014 = SCRUM-5), one line each: key, story id, title. I pick the stories I'll take this sprint.
- Build order: S2 needs S1 in `develop`, S3 needs S2. Say so if I pick more than one.

### 2. Create branch
For each picked story, in the app repo:
- `git fetch --prune origin`
- Stop if the branch or worktree already exists (the last run wasn't reset; point to `pipeline-reset`).
- `git worktree add -b feature/BR014-S<n> "<worktree root>/BR014-S<n>" origin/develop`
- In the worktree, run `npm ci` (lint, Jest and Playwright need `node_modules`; a new worktree has none).
- Copy `features/_template/` to `features/BR014-S<n>/`. Fill the README from Jira in business language: title, Jira link, epic, branch, dev org, story, and the acceptance criteria numbered as in Jira. Leave `architecture.md` as the template.
- Don't commit. `pr-package` commits.

### 3. Pin sandbox
- In each worktree: `sf config set target-org playbook-dev` (local config, only that folder).
- Check it: `sf org display` in the worktree shows `playbook-dev` and status Active. The first call can fail with NoScratchInfo; retry once.
- Show the structure (worktree, branch, feature folder, org) and stop so I can check it in VS Code before design starts.

## Jira
- Site cloudId `21647882-f209-44e2-96aa-f5b9a20c2d31`, project `SCRUM`. Epic BR-014 = SCRUM-5. BR014-S1 = SCRUM-6, S2 = SCRUM-7, S3 = SCRUM-8.
- Statuses, in order: To Do → In Progress → In Review → In QA → QA Accepted → Ready for Release → Done.
- Move a story with `transitionJiraIssue`. Pick the transition by its target status (`getTransitionsForJiraIssue`). IDs: To Do `11`, In Progress `21`, In Review `2`, In QA `3`, QA Accepted `4`, Ready for Release `5`, Done `31`. Skip the move if the story is already there or further along.
- After step 3, move each picked story to **In Progress**. If the epic is still **To Do**, move it to **In Progress** too.
- Stories you didn't pick stay in **To Do**.

## Output
Each story has its own branch, worktree (with `node_modules`), pinned org and feature folder, ready for `architecture-brief`.
