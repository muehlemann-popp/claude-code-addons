# Claude Code Skills by m+p

Shared [Agent Skills](https://github.com/vercel-labs/skills) for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), maintained by [m+p](https://github.com/muehlemann-popp).

## Installation

```bash
npx skills add muehlemann-popp/claude-code-addons/skills
```

This installs all available skills into your Claude Code configuration. You can also install a single skill:

```bash
npx skills add muehlemann-popp/claude-code-addons/skills --skill review-codebase
```

After installation, use the skill as a slash command in any Claude Code session (e.g. `/review-codebase`).

## Available Skills

| Skill | Description |
|-------|-------------|
| `review-codebase` | Comprehensive tech due diligence review with parallelized analysis using Serena and sub-agents |

## Contributing

To add a new skill, create a folder under `skills/` with a `SKILL.md` file containing YAML frontmatter (`name`, `description`) and the agent instructions. See any existing skill for reference.

---

*Last updated: 2026-03-12 | c0c4d75*
