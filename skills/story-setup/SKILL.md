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

## Output
Each story has its own branch, worktree, org and feature folder, ready for `architecture-brief`.

## TODO
- [ ] Jira project key, JQL filter and status names
- [ ] Worktree root path and folder naming
- [ ] Feature folder template
- [ ] Org alias to story mapping
