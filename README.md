# Claude Code Skills

A shared collection of custom slash commands (skills) for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), maintained by [m+p](https://github.com/muehlemann-popp).

Use these skills across all your projects to standardize workflows like code reviews, due diligence, and more.

## Installation

Symlink the commands you want into your Claude Code config directory:

```bash
# All skills at once
for f in /path/to/this/repo/commands/*.md; do
  ln -s "$f" ~/.claude/commands/"$(basename "$f")"
done

# Or individual skills
ln -s /path/to/this/repo/commands/review-codebase.md ~/.claude/commands/review-codebase.md
```

Then use them as `/command-name` in any Claude Code session.

## Available Skills

### `/review-codebase` — Tech Due Diligence Review

A comprehensive, parallelized code review that produces a structured due diligence report. Designed for evaluating codebases during acquisitions, investment decisions, or internal audits.

**How it works:**
1. **Discovery** — Uses Serena (semantic code intelligence) and static analysis tools to understand the codebase
2. **Parallel analysis** — Launches 6 specialized agents simultaneously, each covering a different domain
3. **Aggregation** — Synthesizes findings into a single report with management summary
4. **PDF export** — Renders architecture diagrams and outputs a PDF

**What it analyzes:**

| Agent | Key checks |
|-------|-----------|
| **Code Quality** | Cyclomatic complexity (measured via radon/eslint/gocyclo), dead code detection, type coverage, code duplication, hardcoded values & magic numbers, code smells |
| **Security** | Auth mechanism, hardcoded secrets, input validation (SQLi/XSS/command injection), security anti-patterns, dependency vulnerabilities |
| **Architecture** | Architecture pattern identification, component mapping, data flow analysis, **layer boundary violation detection** (e.g. business logic in views), circular dependencies |
| **Dependencies** | Inventory, outdated/deprecated packages, vulnerability scanning, license analysis, vendor lock-in assessment |
| **DevOps** | CI/CD pipeline analysis, container best practices, IaC coverage, observability stack (logging/monitoring/tracing/alerting), reliability patterns |
| **Maintainability** | Complexity hotspots with root cause classification, technical debt indicators, bus factor, performance concerns (N+1 queries, missing pagination), refactoring opportunities |

**Output:** A structured Markdown + PDF report with:
- Management summary with 1-5 star rating
- Per-domain assessments with file paths and line numbers
- Risk register with impact/likelihood ratings
- Prioritized recommendations (critical / high / medium / low)
- Architecture diagrams (Mermaid, rendered to PNG for PDF)

**Requirements:**
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Serena MCP server](https://github.com/oramasearch/serena) (for semantic code analysis)
- Node.js (for jscpd, mermaid-cli, md-to-pdf — installed automatically if npm is available)
- Language-specific tools are installed automatically as needed (radon, vulture, complexity-report, etc.)

## Contributing

To add a new skill, create a Markdown file in `commands/` and submit a pull request. Each skill should include clear instructions for the agent, expected output format, and any tool requirements.
