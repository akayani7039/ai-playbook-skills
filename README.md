# AI Playbook Skills

Custom skills for the Salesforce AI development playbook.

## Skills

- [jira-story-setup](skills/jira-story-setup/SKILL.md)
- [story-setup](skills/story-setup/SKILL.md)
- [architecture-brief](skills/architecture-brief/SKILL.md)
- [sf-code-review](skills/sf-code-review/SKILL.md)
- [ui-test-playwright](skills/ui-test-playwright/SKILL.md)
- [pr-package](skills/pr-package/SKILL.md)
- [release-train](skills/release-train/SKILL.md)
- [prod-change](skills/prod-change/SKILL.md)

These workflows use installed Salesforce skills and MCP integrations as dependencies. Only custom playbook skills are included here.

## Use across computers

Clone this repository on each computer:

```sh
git clone https://github.com/akayani7039/ai-playbook-skills.git
```

Link each folder under `skills/` into `~/.agents/skills/` so Codex can discover it. Keep any existing folders until you have compared their contents; do not overwrite them. For Claude Code, use `~/.claude/skills/` instead.

Edit skills in this checkout, then commit and push your changes. On another computer, run `git pull --ff-only` from its checkout. Resolve local edits before pulling. Restart your agent session after updating skills.
