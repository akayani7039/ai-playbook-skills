---
name: story-setup
description: Pick up Jira stories and prepare one isolated workspace per story (branch, git worktree, feature folder, pinned Salesforce dev org). Use at the start of a sprint or when the user says "pick up my stories", "set up story", or "start BR-xxx". Pipeline steps 1-3.
---

# story-setup

Stage 01 — Pick up & prepare (steps 1-3).

## Tools
- Jira MCP
- Git MCP / `git worktree`
- Salesforce CLI (`sf config set target-org`)
- Official skill: `dx-org-switch`

## Steps

### 1. Pick up story
- Pull every Jira story assigned to me with status **To Do**.
- Show them grouped by epic. I pick the stories I'll take this sprint.

### 2. Create branch
- Cut one `feature/<EPIC>-S<n>` branch per story from `develop` (e.g. `feature/BR014-S1`).
- Create a git worktree per branch so stories never collide.
- Scaffold a feature folder with the story and acceptance criteria.

### 3. Pin sandbox
- Assign a Salesforce dev org to each worktree.
- Set it as that folder's default org (`sf config set target-org <alias>`).
- Stop so I can check the structure in VS Code before design starts.

## Jira
- Site cloudId `21647882-f209-44e2-96aa-f5b9a20c2d31`, project `SCRUM`. Epic BR-014 = SCRUM-5. BR014-S1 = SCRUM-6, S2 = SCRUM-7, S3 = SCRUM-8.
- Statuses, in order: To Do → In Progress → In Review → In QA → QA Accepted → Ready for Release → Done.
- Move a story with `transitionJiraIssue`. Pick the transition by its target status (`listJiraIssueTransitions`). Skip the move if the story is already there or further along.
- After step 3, move each picked story to **In Progress**. If the epic is still **To Do**, move it to **In Progress** too.
- Stories you didn't pick stay in **To Do**.

## Output
Each story has its own branch, worktree, org and feature folder, ready for `architecture-brief`.

## TODO
- [ ] Jira project key, JQL filter and status names
- [ ] Worktree root path and folder naming
- [ ] Feature folder template
- [ ] Org alias to story mapping
