---
name: architecture-approve
description: Record the architect's decision on a story's architecture.md - show the options and recommendation, ask for the decision, the architect's name and an answer to every open question, then write the approval into architecture.md. Use when the user says "approve the architecture", "approve BR014-S2", or after architecture-brief holds at step 11. Pipeline step 11.
---

# architecture-approve

Stage 02 gate — Architect approval (step 11). No code is written in this stage.

## Tools
- `AskUserQuestion` for every decision. Give options. Put the recommended one first.
- Jira MCP (optional comment, only if the architect asks)

## Before you start
- Find the story. Use the argument (e.g. `BR014-S2`) or the current branch `feature/BR014-S<n>`.
- Read `features/BR014-S<n>/architecture.md` and `README.md`.
- **Stop if `architecture.md` is missing** or has no options, recommendation or build plan. Point to `architecture-brief`.
- **If it is already `Approved`**, say who approved it and when. Ask whether to re-open it. Don't overwrite silently.

## Steps
1. **Show the brief** — In a few short lines: the options (one line each), the recommendation and why, the build plan rows, the dependencies on sibling stories, and the open questions. Don't paste the whole file.
2. **Ask for the decision** — One question with these options:
   - Approve the recommended option (Recommended)
   - Approve a different option (then ask which one, listing the options from section 3)
   - Return for changes (ask what to change)
   - Reject
3. **Ask the open questions** — Ask every question in section 7 ("Open questions"). Up to 4 per `AskUserQuestion` call. For each one, offer the design's assumption as the first option, marked (Recommended), plus the real alternatives. Never skip one. Never answer one yourself.
4. **Check the impact of the answers** — If an answer, or a choice of a different option, changes the build plan (a new or removed component, a different permission, a changed test), tell the architect what changes.
   - A small change (one row or one test): ask whether to update section 6 now and approve.
   - A bigger change: don't approve. Record it as "Returned for changes" and hand back to `architecture-brief`.
5. **Ask for the architect's name** — Offer `git config user.name` as the first option if it is set. Otherwise ask for it. The date is today (YYYY-MM-DD). Never guess a name.
6. **Ask for conditions** — "Any conditions or notes for the build?" Options: None (Recommended), Add a note.
7. **Show the changes and confirm** — Show the new status lines and section 8 before you write them. Ask: Write it (Recommended), Change something, Cancel.
8. **Write to `architecture.md`** — See "What to write" below.
9. **Optional Jira comment** — Ask whether to post the decision as a comment on the story's Jira issue. Default is no. Post only on a yes.

## What to write

The header must match what `story-build` checks for:

```markdown
**Status:** Approved
**Architect:** <name> — **Decision date:** <YYYY-MM-DD>
```

For other decisions, use `Returned for changes` or `Rejected` as the status. Keep the architect and date lines.

In section 7, add the answer under each question: `**Answer:** <answer>`.

If the plan changed in step 4, update section 6 and note it in section 8.

Add a new section at the end:

```markdown
## 8. Approval

- **Decision:** Approved: Option A (LWC card + Apex controller)
- **Architect:** <name>, <YYYY-MM-DD>
- **Open questions:** all answered in section 7
- **Plan changes:** none / <what changed in section 6>
- **Conditions:** none / <note>
- **Blockers before build:** <sibling stories not yet in develop, or none>
```

For blockers, check whether every component the plan says comes from a sibling story is in `develop` (`git ls-tree -r --name-only develop`). List each one that is missing.

## Rules
- The architect decides. You present and record. Don't approve on anyone's behalf.
- Every open question gets an answer before the status becomes `Approved`.
- Only edit `architecture.md` (and the README only if an answer changes the acceptance criteria). No code, no metadata.
- Don't commit, push or deploy. Commits happen in `pr-package`.
- Don't post to Jira or Slack unless the architect says yes.

## Output
Report:

| Item | Result |
|---|---|
| Decision | Approved / Returned / Rejected, and which option |
| Architect, date | Name, YYYY-MM-DD |
| Open questions | Each question → answer |
| Plan changes | None, or the rows changed |
| Blockers | Sibling stories still missing from develop |
| Next step | `story-build` if approved and unblocked; `architecture-brief` if returned |
