# Claude Code Skills

A collection of work-in-progress custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Skills

| Skill | Description | State |
|---|---|---|
| [php-diagrams](#php-diagrams) | Generate Mermaid diagrams from PHP codebases | alpha test |
| [php-regex](#php-regex) | Interactive PHP regex builder, debugger, and tester | alpha test |

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

### php-regex

Interactive PHP regex builder, debugger, and tester. Builds patterns from natural language descriptions, explains existing patterns, generates test suites, and catches common pitfalls.

- Build regex from natural language descriptions with incremental refinement
- Explain and visually break down existing patterns
- Debug failing patterns with step-by-step diagnosis
- Optimize patterns for performance (backtracking, possessive quantifiers, anchoring)
- Generate Pest or PHPUnit test cases for any pattern
- Save patterns to a documented library with auto-maintained index
- Searches your codebase for real sample data and existing patterns before building
- Configurable via `.regex-workshop.yaml` in your project root

**Usage:** Ask Claude Code to build, explain, debug, optimize, or test any PHP regex. The skill searches the project for real data and existing patterns, builds incrementally, and always explains what each part does.

**Files:**
- `php-regex-skill/SKILL.md` — skill definition and workflow
- `php-regex-skill/.regex.yaml` — configuration template
- `php-regex-skill/references/php-regex-cheatsheet.md` — PHP regex syntax reference and PCRE features
- `php-regex-skill/references/common-patterns.md` — ready-to-use patterns for common formats

## Installation

Copy the desired skill directory into your Claude Code skills configuration. Refer to the [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code) for details on installing custom skills.

## License

MIT
