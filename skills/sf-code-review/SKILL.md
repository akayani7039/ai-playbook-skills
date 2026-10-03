---
name: sf-code-review
description: AI code review of a Salesforce pull request for standards, security and bulk-safe code (Apex, LWC, OmniStudio, metadata). Use after pr-package opens a PR and CI has run, before the tech lead approves. Pipeline step 28.
---

# sf-code-review

Stage 04 — step 28. Runs before the tech lead's approval (step 29, HUMAN GATE).

## Tools
- GitHub MCP (read the PR diff, post comments)
- Official skills: `dx-apexguru-scan`, `platform-architecture-analyze`

## Steps
1. Read the PR diff, the PR summary and the story's `architecture.md`.
2. Check the code matches the approved design.
3. Review against the checklist below.
4. Post findings as PR comments, ranked by severity.
5. Give a summary: ready for tech lead, or needs fixes.

## Checklist
- **Security** — sharing keywords, CRUD/FLS, SOQL injection, LWS
- **Bulk-safe** — no SOQL/DML in loops, handles 200+ records
- **Governor limits**
- **Standards** — naming, structure, error handling
- **Tests** — coverage and meaningful asserts

## TODO
- [ ] Team coding standards
- [ ] Severity levels and what blocks the merge
- [ ] Comment format
