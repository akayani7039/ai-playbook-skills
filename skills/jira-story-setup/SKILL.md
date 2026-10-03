---
name: jira-story-setup
description: Pull my open Jira stories, let me pick which to work on, then set up feature folders, git worktrees, and a Salesforce target org per worktree.
---

# Jira story setup

Sets up local work for Jira stories assigned to me. Ask first, act second.
Do not create, change, or delete anything until Step 4 is confirmed.

## Step 1 — Read my stories (Jira MCP server)

All Jira access goes through the Jira MCP server. Don't use curl, the REST API
directly, or any CLI.

1. **Check the server is there.** Look for the Jira MCP tools (names start with
   `mcp__atlassian__` or similar). If none are available, stop and tell me the
   Jira MCP server isn't connected. Don't guess stories.
2. **Find the Jira site.** Call `getAccessibleAtlassianResources` and take the
   `cloudId` for the Jira site. If there's more than one site, ask me which one
   with AskUserQuestion.
3. **Search with JQL.** Call `searchJiraIssuesUsingJql` with that `cloudId` and:

   ```
   assignee = currentUser()
   AND issuetype = Story
   AND statusCategory != Done
   ORDER BY priority DESC, updated DESC
   ```

   Request fields: summary, status, priority, sprint, story points
   (story points and sprint are custom fields, so match them by name).
   Limit to 50 results.
4. **Get details later, not now.** Only call `getJiraIssue` for the stories I
   select in Step 2, to pull description and acceptance criteria for the
   feature README.

If a tool name differs on this server, use the one that does the same job
(site lookup, JQL search, single-issue read). This skill only reads Jira.

Show a short table of the results (key, summary, status, points).

## Step 2 — Ask which stories to work on

Use AskUserQuestion with `multiSelect: true`.

- Each option: label = story key, description = summary + status.
- A card holds at most 4 options. If there are more than 4 stories, show the
  top 4 by priority and tell me I can type other keys in "Other"
  (e.g. `AFRS-1412, AFRS-1420`).
- If I pick nothing, stop.

## Step 3 — Ask how to set up

Before asking, gather what the options need:
- Active and next sprint numbers from the selected stories' Jira sprint field.
- Authenticated orgs from `sf org list --json` (use aliases; skip expired ones).

Then ask these in ONE AskUserQuestion call:

1. **Sprint number** (single select)
   - Current sprint (e.g. "Sprint 14")
   - Next sprint (e.g. "Sprint 15")
   - I can type another number in "Other"

2. **Create feature folders?** (single select)
   - Yes — `features/sprint-<N>/<KEY>/` with a README
   - No

3. **Create git worktrees?** (single select)
   - Yes — one worktree per story
   - No — just create branches in the current checkout
   - No branches or worktrees

4. **Target org for the worktrees** (single select, only if worktrees = Yes)
   - Up to 4 org aliases from `sf org list`
   - "Pick per story" — ask again for each story in a follow-up question

If "Pick per story" was chosen, ask one follow-up question per story
(up to 4 stories per AskUserQuestion call) with the same org options.

## Step 4 — Confirm the plan

Show exactly what will happen, for example:

```
Sprint 14 · base branch origin/develop

AFRS-1412  Recruiter event calendar filters
  folder    features/sprint-14/AFRS-1412/
  branch    feature/sprint-14/AFRS-1412-calendar-filters
  worktree  ../wt/AFRS-1412
  org       afrs-dev2

AFRS-1420  Lead intake OmniScript validation
  ...
```

Ask: "Go ahead?" (Yes / Change something / Cancel). Only continue on Yes.

## Step 5 — Execute

Run from the repo root. Stop on the first error and report it.

1. `git fetch origin`
2. For each story:
   - **Branch name:** `feature/sprint-<N>/<KEY>-<short-slug>` (slug from summary,
     lowercase, dashes, max 5 words).
   - **Skip, don't overwrite:** if the branch or worktree path already exists,
     skip that story and note it.
   - **Worktree (if Yes):**
     `git worktree add ../wt/<KEY> -b <branch> origin/develop`
   - **Branch only (if chosen):**
     `git branch <branch> origin/develop`
   - **Feature folder (if Yes):** create `features/sprint-<N>/<KEY>/README.md`
     inside the worktree (or repo root if no worktree) with:
     - Jira key, link, summary, points
     - Description and acceptance criteria from `getJiraIssue`
     - Empty "Notes" and "Components changed" sections
   - **Target org (if worktrees = Yes):** inside the worktree run
     `sf config set target-org <alias>`
     This writes local `.sf/config.json`, so each worktree deploys to its own org.
     Confirm `.sf/` and `.sfdx/` are in `.gitignore`; warn if not.

## Step 6 — Summarize

Report per story: branch, worktree path, org, folder, and anything skipped.
End with the command to open each worktree, e.g. `code ../wt/AFRS-1412`.

## Rules

- Never delete branches, worktrees, or folders.
- Never push to remote.
- Never change Jira (no status or sprint updates) unless I ask.
- Base branch is `origin/develop` unless I say otherwise.
