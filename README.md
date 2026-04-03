# Claude Code Skills

A collection of work-in-progress custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Skills

### php-diagrams

Generate Mermaid diagrams from PHP codebases by researching actual project code.

- Supports all 16 Mermaid diagram types (sequence, class, ERD, state, flowchart, and more)
- Works with Laravel, Symfony, plain PHP, or any other framework
- Configurable via `.diagrams.yaml` in your project root
- Outputs organized Markdown files with an auto-maintained index
- Multi-repo support for cross-service diagram generation

**Usage:** Ask Claude Code to diagram any part of your PHP codebase — a flow, integration, database schema, or system architecture. The skill researches the actual code and generates accurate diagrams.

**Files:**
- `php-diagrams-skill/SKILL.md` — skill definition and workflow
- `php-diagrams-skill/.diagrams.yaml` — configuration template with all available options
- `php-diagrams-skill/references/mermaid-syntax.md` — Mermaid syntax reference for all 16 diagram types

## Installation

Copy the desired skill directory into your Claude Code skills configuration. Refer to the [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code) for details on installing custom skills.

## License

MIT
