---
name: architecture-brief
description: Design a Jira story against its whole epic and write architecture.md with options, recommendation, security/NFR checks and a build plan, then hold for architect approval. Use after story-setup and before any code is written. Pipeline steps 4-11.
---

# architecture-brief

Stage 02 — Architecture (steps 4-11). No code is written in this stage.

## Tools
- Jira MCP, Salesforce MCP
- Project docs in the app repo: `README.md`, `CLAUDE.md`, `docs/pipeline-setup.md`
- Official skills: `platform-metadata-retrieve`, `platform-docs-get`, `dx-org-analyze`, `omnistudio-dependencies-analyze`, `platform-architecture-analyze`, `external-diagram-mermaid-generate`

## Plugin hook
The salesforce-development plugin denies raw `sf project retrieve`, `sf data query`, `sf apex run test` and `sf project generate manifest` until its owning skill (`platform-metadata-retrieve`, `platform-soql-query`, `platform-apex-test-run`, `platform-manifest-generate`) has been dispatched in the same turn. Dispatch that skill first, then run the command shown here. Deploy commands only get a warning.

## Steps
4. **Generate architecture brief** — Read the full epic and its sibling stories so the design fits the whole feature. Work in the story's worktree. Read the old approved design if it exists (`git show dd9602e:features/BR014-S<n>/architecture.md` for S1/S2, `git show e941639:features/BR014-S3/architecture.md`) and reuse its decisions unless the story changed.
5. **Retrieve solution context** — Load the project docs and the current org metadata from `playbook-dev` (retrieve into a scratch folder, not `force-app`).
6. **Analyze impact + dependencies** — Map everything the change touches (objects, triggers, flows, OmniStudio).
7. **Generate design options** — Draft two or three options (e.g. Apex vs. Flow).
8. **Check security + NFRs** — Sharing, CRUD/FLS (including Admin profile read access for new fields), user vs. system mode, governor limits, bulk (200 records), data sensitivity.
9. **Recommend solution** — Pick one option and explain the trade-offs.
10. **Generate design + build plan** — Fill `features/BR014-S<n>/architecture.md` from the template: options, recommendation, build plan with components, tests (one per acceptance criterion), deploy order, and open questions in section 7. Status `Draft`.
11. **Architect approval** — HUMAN GATE. Stop and wait. The architect records the decision with `architecture-approve`.

## Jira
- Site cloudId `21647882-f209-44e2-96aa-f5b9a20c2d31`, project `SCRUM`. Epic BR-014 = SCRUM-5. BR014-S1 = SCRUM-6, S2 = SCRUM-7, S3 = SCRUM-8.
- Statuses, in order: To Do → In Progress → In Review → In QA → QA Accepted → Ready for Release → Done.
- Move a story with `transitionJiraIssue`. Pick the transition by its target status (`getTransitionsForJiraIssue`). IDs: To Do `11`, In Progress `21`, In Review `2`, In QA `3`, QA Accepted `4`, Ready for Release `5`, Done `31`. Skip the move if the story is already there or further along.
- If the story is still **To Do**, move it to **In Progress** at step 4.
- At step 11, add a comment: brief ready for the architect, the recommended option, and the path to `architecture.md`. The status stays **In Progress**.

## Output
`architecture.md` in the story's feature folder, approved by the architect.
