---
name: sf-code-review
description: AI code review of a Salesforce pull request for standards, security and bulk-safe code (Apex, LWC, OmniStudio, metadata). Use after pr-package opens a PR and CI has run, before the tech lead approves. Pipeline step 28.
---

# sf-code-review

Stage 04 — step 28. Runs before the tech lead's approval (step 29, HUMAN GATE).

## Tools
- `gh` (`gh pr diff <n>`, `gh pr view <n>`, `gh pr review <n> --comment --body-file <file>`)
- Official skills: `dx-apexguru-scan`, `platform-architecture-analyze`

## Steps
1. Read the PR diff, the PR summary and the story's `architecture.md`.
2. Check the code matches the approved design.
3. Review against the checklist below.
4. Post one PR review as a **comment** (`gh pr review --comment`), findings ranked by severity: Blocker, Major, Minor. Never `--approve` or `--request-changes`: approval is the tech lead's (gate 2).
5. Give a summary: ready for tech lead, or needs fixes. Blockers go back to `story-build`.

## Checklist
- **Security** — sharing keywords, CRUD/FLS, SOQL injection, LWS. Code Analyzer doesn't flag `AccessLevel.SYSTEM_MODE` or `without sharing` DML: check each one by hand and confirm `architecture.md` approves it.
- **Design** — every build plan row is in the diff, nothing extra; new fields have `Sales_Rep_Base` and Admin profile access
- **Bulk-safe** — no SOQL/DML in loops, handles 200+ records
- **Governor limits**
- **Standards** — naming, structure, error handling
- **Tests** — coverage and meaningful asserts
