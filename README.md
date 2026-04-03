# Claude Code Skills

This is my public lab for custom [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills. Everything here is experimental and work-in-progress — some skills are serious attempts, some are just me testing things out, and some exist purely for fun.

Nothing is production-ready. Use at your own risk, break things, have fun.

PRs and issues are welcome if you want to discuss ideas, suggest improvements, or share your own experiments.

## Skills

| Skill | Description | State      |
|---|---|---|
| [php-diagrams](#php-diagrams) | Generate Mermaid diagrams from PHP codebases | draft |
| [php-regex](#php-regex) | Interactive PHP regex builder, debugger, and tester | draft |
| [lucid-diagrams](#lucid-diagrams) | Generate Lucidchart/Lucidspark diagrams from PHP codebases via MCP | draft |
| [drawio-diagrams](#drawio-diagrams) | Generate Draw.io diagrams from PHP codebases via MCP | draft |
| [php-explain](#php-explain) | Explain how parts of a PHP system work by tracing actual code | draft |

### php-diagrams

Generate Mermaid diagrams from PHP codebases by researching actual project code.

- Supports all 16 Mermaid diagram types (sequence, class, ERD, state, flowchart, and more)
- Works with Laravel, Symfony, plain PHP, or any other framework
- Configurable via `.mermaid-diagrams.yaml` in your project root
- Outputs organized Markdown files with an auto-maintained index
- Multi-repo support for cross-service diagram generation

**Usage:** Ask Claude Code to diagram any part of your PHP codebase — a flow, integration, database schema, or system architecture. The skill researches the actual code and generates accurate diagrams.

**Files:**
- `php-diagrams-skill/SKILL.md` — skill definition and workflow
- `php-diagrams-skill/.mermaid-diagrams.yaml` — configuration template with all available options
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

### lucid-diagrams

Create diagrams in Lucidchart and Lucidspark by researching actual PHP project code and generating them via the Lucid MCP server.

- Creates diagrams directly in Lucidchart or Lucidspark via MCP integration
- Auto-selects product: Lucidchart for formal diagrams, Lucidspark for collaborative ones
- Works with Laravel, Symfony, plain PHP, or any other framework
- Searches and updates existing Lucid diagrams across sessions
- Multi-repo support for cross-service diagram generation
- Configurable via `.lucid-diagrams.yaml` in your project root
- Falls back to php-diagrams skill (Mermaid) if Lucid is unavailable

**Usage:** Ask Claude Code to create a diagram in Lucidchart or Lucidspark for any part of your PHP codebase. Requires the Lucid MCP server to be connected and authenticated.

**Files:**
- `lucid-diagrams-skill/SKILL.md` — skill definition and workflow
- `lucid-diagrams-skill/.lucid-diagrams.yaml` — configuration template
- `lucid-diagrams-skill/references/lucid-conventions.md` — Lucid MCP tool usage and formatting conventions

### drawio-diagrams

Create diagrams in Draw.io by researching actual PHP project code and generating them via the Draw.io MCP server. Supports Mermaid (recommended default), native XML, and CSV input formats.

- Works with multiple Draw.io MCP server variants (official tool server, MCP app server, community server)
- Three input formats: Mermaid (recommended), XML (precise control), CSV (tabular data)
- Auto-discovers which MCP server variant is connected and adapts accordingly
- Works with Laravel, Symfony, plain PHP, or any other framework
- Falls back to Mermaid markdown if no Draw.io MCP server is available

**Usage:** Ask Claude Code to create a diagram in Draw.io for any part of your PHP codebase. Requires a Draw.io MCP server to be connected.

**Files:**
- `drawio-diagrams/SKILL.md` — skill definition and workflow
- `drawio-diagrams/references/drawio-formats.md` — Draw.io XML, Mermaid, and CSV format reference

### php-explain

Explain how parts of a PHP system work by tracing through actual code. Follows call chains, connects components, explains design decisions, and presents a coherent narrative.

- Traces entire call chains from entry point to database and back
- Explains the "why" behind design decisions, not just the "what"
- Supports multiple explanation types: flow traces, component overviews, data lifecycles, integration maps
- Works with Laravel, Symfony, plain PHP, or any other framework
- Saves explanations to a documented library with auto-maintained index
- Configurable via `.php-explain.yaml` in your project root

**Usage:** Ask Claude Code to explain how any part of your PHP codebase works — a flow, a service, a class, or how components connect. The skill traces through the actual code and presents a coherent narrative.

**Files:**
- `php-explain/SKILL.md` — skill definition and workflow
- `php-explain/references/explanation-patterns.md` — templates for different explanation types

## Installation

Copy the desired skill directory into your Claude Code skills configuration. Refer to the [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code) for details on installing custom skills.

## License

MIT
