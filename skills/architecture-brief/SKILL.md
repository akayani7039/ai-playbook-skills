---
name: architecture-brief
description: Design a Jira story against its whole epic and write architecture.md with options, recommendation, security/NFR checks and a build plan, then hold for architect approval. Use after story-setup and before any code is written. Pipeline steps 4-11.
---

# architecture-brief

Stage 02 — Architecture (steps 4-11). No code is written in this stage.

## Tools
- Jira MCP, Salesforce MCP
- Docling (project docs converted to Markdown)
- Official skills: `platform-metadata-retrieve`, `platform-docs-get`, `dx-org-analyze`, `omnistudio-dependencies-analyze`, `platform-architecture-analyze`, `external-diagram-mermaid-generate`

## Steps
4. **Generate architecture brief** — Read the full epic and its sibling stories so the design fits the whole feature.
5. **Retrieve solution context** — Load project docs and current org metadata.
6. **Analyze impact + dependencies** — Map everything the change touches (objects, triggers, flows, OmniStudio).
7. **Generate design options** — Draft two or three options (e.g. Apex vs. Flow).
8. **Check security + NFRs** — Sharing, CRUD/FLS, governor limits, IL4/IL5 data rules.
9. **Recommend solution** — Pick one option and explain the trade-offs.
10. **Generate design + build plan** — Write `architecture.md`: components, tests, deploy order.
11. **Architect approval** — HUMAN GATE. Stop and wait. The architect approves or returns the design.

## Jira
- Site cloudId `21647882-f209-44e2-96aa-f5b9a20c2d31`, project `SCRUM`. Epic BR-014 = SCRUM-5. BR014-S1 = SCRUM-6, S2 = SCRUM-7, S3 = SCRUM-8.
- Statuses, in order: To Do → In Progress → In Review → In QA → QA Accepted → Ready for Release → Done.
- Move a story with `transitionJiraIssue`. Pick the transition by its target status (`listJiraIssueTransitions`). Skip the move if the story is already there or further along.
- If the story is still **To Do**, move it to **In Progress** at step 4.
- At step 11, add a comment: brief ready for the architect, the recommended option, and the path to `architecture.md`. The status stays **In Progress**.

## Output
`architecture.md` in the story's feature folder, approved by the architect.

## TODO
- [ ] architecture.md template
- [ ] Location of project docs / Docling output
- [ ] Security and NFR checklist
- [ ] How approval is recorded (Jira comment, status, PR?)
