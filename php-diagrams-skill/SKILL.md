---
name: php-diagrams
description: >
  Generate Mermaid diagrams from PHP codebases by researching actual project code. Use this skill
  whenever the user asks to create, generate, or update diagrams, flowcharts, sequence diagrams,
  class diagrams, state diagrams, ER diagrams, or any visual documentation for their PHP project.
  Also trigger when the user says things like "diagram the payment flow", "show me how registration
  works", "visualize the database schema", "map out the Stripe integration", "document the
  architecture", or any request to understand or document code visually. Trigger even if the user
  doesn't explicitly say "diagram" — phrases like "how does X flow work" or "show me the
  relationship between these models" should also activate this skill. Works with any PHP codebase:
  Laravel, Symfony, plain PHP, or any other framework.
---

# PHP Diagram Generator

Generate Mermaid diagrams by researching actual PHP project code and saving them as organized
Markdown files.

## References

- `references/mermaid-syntax.md` — Mermaid syntax examples for all 16 diagram types.
  **Read this before generating any diagram.** Do not generate from memory.

## Configuration

Before starting the workflow, look for `.diagrams.yaml` (or `.diagrams.yml`) in the project root.
This file lets the user customize paths, sources, and behavior. If the file doesn't exist, use
defaults. See the bundled `.diagrams.yaml` for all available options with documentation.

**Key config fields and their defaults:**

| Field | Default | Purpose |
|---|---|---|
| `output.diagrams_dir` | `./docs/diagrams` | Where diagram files are stored |
| `output.index_file` | `./docs/diagrams/index.md` | Path to the diagram index |
| `sources.include` | `[]` (whole project) | Directories to research |
| `sources.exclude` | `vendor`, `node_modules`, `.git`, `var/cache`, `storage/framework`, `tests`, `test` | Directories to skip |
| `repositories` | `[]` | Additional local paths or git URLs to research |
| `framework` | `auto` | Framework hint: `auto`, `laravel`, `symfony`, `plain` |
| `defaults.types` | `[]` | Pre-selected diagram types |
| `defaults.detail` | `standard` | Detail level: `overview`, `standard`, `detailed` |
| `defaults.include_error_paths` | `true` | Include error/edge-case paths |
| `mermaid.theme` | `default` | Mermaid theme: `default`, `dark`, `forest`, `neutral` |

**Config loading rules:**

1. Read `.diagrams.yaml` / `.diagrams.yml` from project root
2. If it doesn't exist, use all defaults — don't ask the user to create one
3. Any missing field uses its default value
4. If `sources.include` is empty or not set, research the entire project (minus excludes)
5. If `repositories` has git URLs, attempt to clone them to a temp directory. If network is
   unavailable (common in sandboxed environments), warn the user and skip remote repos. Local
   paths always work.
6. Paths are always relative to the project root

Throughout this skill, "index file" and "diagrams directory" refer to config values (or defaults).

## Workflow

Follow these steps in order. At every stage, if you have questions or need clarification — ask the
user. Do not guess or assume.

### Step 0: Load configuration

1. Check for `.diagrams.yaml` or `.diagrams.yml` in the project root
2. If found, parse it and merge with defaults
3. If `repositories` has entries, resolve them:
   - Local paths: verify they exist
   - Git URLs: attempt clone to a temp dir. If clone fails (network unavailable, auth error),
     warn the user and continue without that repo
4. Remember all resolved paths for subsequent steps

### Step 1: Check for existing diagrams index

Look for the index file (default: `./docs/diagrams/index.md`).

- If it exists, read it and keep its contents in mind for the next steps.
- If it doesn't exist, that's fine — you'll create it later.

### Step 2: Ask what to diagram

Ask the user what part of the codebase they want to diagram. Be conversational:

> What would you like me to diagram? For example:
> - A specific flow (e.g. "User Registration", "Payment processing")
> - An integration (e.g. "Stripe integration", "API authentication")
> - A system or subsystem (e.g. "Notification system", "Queue workers")
> - Database relationships (e.g. "User-related models", "Order schema")

If the user already specified what they want in their initial message, skip this question.

**Check for duplicates:** If the index exists and contains a diagram that matches or closely
matches what the user is asking for, tell them:

> I found an existing diagram that might be what you're looking for:
> - [Diagram Name](./path/to/diagram.md) — short description
>
> Would you like to:
> 1. View this existing diagram
> 2. Update/regenerate it (the code may have changed)
> 3. Create a new, separate diagram

Wait for the user's response before continuing.

### Step 3: Ask for diagram type(s)

Ask the user what type(s) of diagram they want. They can pick multiple:

> What type(s) of diagram should I create? You can choose multiple:
> - **Sequence** — flow of calls/messages between components over time
> - **Class** — classes, their properties, methods, and relationships
> - **State** — states an entity goes through and transitions between them
> - **ERD** (Entity Relationship) — database tables, columns, and their relationships
> - **Flowchart** — decision logic, branching, and process steps
> - **Activity** — parallel and branching workflows (swimlanes)
> - **Component** — high-level system architecture, services, packages, and dependencies
> - **Deployment** — infrastructure, servers, containers, and how services are deployed
> - **Data Flow (DFD)** — how data moves through the system between processes and stores
> - **Mind Map** — hierarchical exploration of a concept, feature, or subsystem
> - **Timeline** — chronological view of events, migrations, or lifecycle phases
> - **Git Graph** — branching strategies, release flows, or merge patterns
> - **Block** — nested grouping for architecture layers, modules, or bounded contexts
> - **Sankey** — weighted flow between nodes (e.g. request routing, pipeline throughput)
> - **Packet** — protocol or message structure (e.g. API payload breakdown)
> - **Architecture** — cloud/infra architecture with services, databases, queues, caches

If the user already specified the type in their initial message, skip this question.

If config has `defaults.types` set, pre-suggest those but still ask for confirmation.

If the user isn't sure, recommend based on what they want to diagram:

| What they want | Recommended types |
|---|---|
| Flow / process | Sequence + Flowchart |
| Integration with external API | Sequence + Component |
| Database / models | ERD + Class |
| State machine / status field | State |
| Business logic with branching | Flowchart or Activity |
| System overview / architecture | Component + Deployment |
| How data moves through the app | Data Flow |
| Feature planning or exploration | Mind Map |
| Migration or rollout plan | Timeline |
| Infrastructure / cloud setup | Architecture + Deployment |
| API request/response structure | Packet |
| CI/CD or branching strategy | Git Graph |
| Module or layer organization | Block |

### Step 4: Research the code

This is the most important step. Thoroughly research the actual codebase before writing any diagram.

**Scope:** Only search within directories from `sources.include` (or the entire project if not
set). Skip everything in `sources.exclude`. If `repositories` are configured, research those too
as additional read-only source trees.

**How to research:**

1. **Detect the framework** — if config `framework` is not `auto`, use that. Otherwise check:
   - `artisan` in root, `app/Http/` → **Laravel**
   - `bin/console`, `config/bundles.php`, `src/Kernel.php` → **Symfony**
   - `composer.json` with no framework dependency → **Plain PHP** or micro-framework
   - Check `composer.json` for additional clues (packages, autoload paths)

2. **Start broad** — look at the project structure, find relevant directories.

   **Laravel:** `app/Http/Controllers/`, `app/Models/`, `app/Services/`, `app/Actions/`,
   `app/Events/`, `app/Listeners/`, `app/Jobs/`, `routes/`, `database/migrations/`, `config/`

   **Symfony:** `src/Controller/`, `src/Entity/`, `src/Repository/`, `src/Service/`,
   `src/Handler/`, `src/EventSubscriber/`, `src/EventListener/`, `src/Message/`,
   `src/MessageHandler/`, `src/Command/`, `config/routes/`, `migrations/`, `config/packages/`

   **Plain PHP:** read `composer.json` autoload for source directories, look for
   `public/index.php`, check `src/`, `lib/`, `classes/`, look for SQL files or schema dumps

3. **Go deep** — read the actual files, not just names. Follow the call chain:
   - For a flow: route → controller → service/handler → entity/model → events
   - For an integration: client class, API calls, webhooks, service configuration
   - For DB relationships: migrations/entity annotations AND relationship definitions
     - Laravel: migration files + model methods (`hasMany`, `belongsTo`, etc.)
     - Symfony: Doctrine attributes (`#[OneToMany]`, `#[ManyToOne]`, etc.)

4. **Note everything relevant:** method signatures, return types, middleware/security voters/
   guards/firewalls, database columns and types, event dispatching, async jobs/message handlers,
   external API calls, error handling and edge cases.

5. **If research finds nothing** — the requested flow/feature may not exist in the code. Don't
   invent or guess. Instead, tell the user what you found (or didn't find) and ask how to proceed:

   > I searched the codebase but couldn't find an implementation for "{what user asked for}".
   > Here's what I found related to this area: {list what you did find, if anything}.
   >
   > Would you like me to:
   > 1. Look again — point me to the right files or directories
   > 2. Create a draft/planned diagram showing a proposed design (marked as DRAFT)
   > 3. Diagram something else instead

   If the user chooses option 2, mark the diagram clearly as a draft — both in the title
   (`# [DRAFT] Proposed: {Title}`) and in the index entry. Draft diagrams document intent,
   not existing code.

6. **Ask if unclear** — if you find ambiguous code, multiple possible flows, or can't determine
   the intended behavior, ask the user rather than guessing.

### Step 5: Generate the Mermaid diagram(s)

**Before generating, read `references/mermaid-syntax.md`** for syntax examples of the diagram
type(s) you need. Do not rely on memory — the reference file has correct, tested syntax.

Create accurate Mermaid diagrams based on your code research.

**Quality standards:**

- Every element must correspond to something real in the code (unless it's a DRAFT diagram)
- Use actual class names, method names, and table/column names from the codebase
- Include error paths and edge cases if `defaults.include_error_paths` is true
- Add notes/comments for non-obvious behavior
- Keep diagrams readable — if too complex, split into sub-diagrams and link them

**Respect the detail level** from config `defaults.detail`:

- `overview` — high-level boxes, no method names, major flows only
- `standard` — class/method names, happy path + key error paths
- `detailed` — every method call, all error/edge-case paths, middleware, events, queue jobs

### Step 6: Save the diagram(s)

Save each diagram as a Markdown file:

**Path:** `{diagrams_dir}/{diagram-type}/{diagram-name}.md`

- `{diagrams_dir}` from config (default: `./docs/diagrams`)
- `{diagram-type}` — lowercase: `sequence`, `class`, `state`, `erd`, `flowchart`, `activity`,
  `component`, `deployment`, `data-flow`, `mind-map`, `timeline`, `git-graph`, `block`,
  `sankey`, `packet`, `architecture`
- `{diagram-name}` — kebab-case, descriptive (e.g. `user-registration`, `stripe-payment-flow`)

**File template:**

```markdown
# {Diagram Title}

> {One-line description of what this diagram shows}

## Context

{Brief explanation of the flow/system, including which files/classes are involved.}

## Diagram

{open a mermaid code block here}
{mermaid code}
{close the code block}

## Key Files

- `path/to/Controller.php` — entry point
- `path/to/Service.php` — core logic
{use actual paths from the project; prefix with repo label for multi-repo setups}

## Notes

- {Caveats, simplifications, or things to watch out for}
```

For **DRAFT diagrams**, use this title format: `# [DRAFT] Proposed: {Title}` and add a note
at the top: `> ⚠️ This diagram represents a proposed design, not existing code.`

Create directories as needed (`mkdir -p`).

### Step 7: Update the index

Create or update the index file (default: `./docs/diagrams/index.md`).

**Build the index dynamically** — only include sections for diagram types that actually have
diagrams. Do not use a static template with all 16 types.

```markdown
# Project Diagrams

> Diagram index. Last updated: {YYYY-MM-DD}

### {Diagram Type}
- [{Title}](./{type}/{name}.md) — {short description}
```

**Rules:**
- Only include section headers for types that have at least one diagram
- Keep entries sorted alphabetically within each section
- Preserve existing entries — only add or update, never remove unless the user asks
- Use relative paths from the index file's location
- Mark DRAFT diagrams: `- [DRAFT: {Title}](./{type}/{name}.md) — {description}`

### Step 8: Confirm with the user

After saving, tell the user:
- What diagrams were created and where they're saved
- A brief summary of what each diagram shows
- Ask if they want any adjustments (more detail, less detail, different scope, corrections)

## Updating Existing Diagrams

When the user asks to update, regenerate, or refresh an existing diagram:

1. Read the existing diagram file to understand what it documents
2. Re-research the relevant code (Step 4) to find what has changed
3. **Always ask the user before overwriting:**

   > I've re-analyzed the code for "{diagram title}". Here's what changed:
   > - {list of changes found, or "no significant changes detected"}
   >
   > Should I regenerate the diagram with these updates?

4. If the user confirms, regenerate and overwrite the file. Update the `Last updated` date in
   the index.
5. If no changes were found, tell the user — don't regenerate unnecessarily.

## Important Principles

1. **Accuracy over aesthetics** — a correct but plain diagram beats a pretty but wrong one.
   Every arrow, every label should reflect real code.

2. **Ask, don't assume** — when in doubt, ask the user. Especially about:
   - Which flow variant to document (happy path only vs. error handling too?)
   - Level of detail (high-level overview vs. detailed with every method call?)
   - Scope boundaries (just the controller+service, or include middleware, events, queues?)

3. **Research thoroughly** — spend more time reading code than writing Mermaid. The diagram
   quality is directly proportional to how well you understand the code.

4. **Incremental is fine** — the user can start with one diagram and add more later. The index
   file tracks everything.

5. **Multi-repo awareness** — when `repositories` are configured, label components in diagrams
   with their repo label so it's clear which codebase each piece lives in. In the Key Files
   section, prefix paths with the repo label (e.g. `[Common SDK] src/Client/StripeClient.php`).

6. **Respect detail level** — follow `defaults.detail` from config:
   - `overview` — high-level boxes, no method names, major flows only
   - `standard` — class/method names, main happy path + key error paths
   - `detailed` — every method call, all error paths, middleware, events, queue jobs

7. **Network may be unavailable** — if git clone fails for remote repositories in config, warn
   the user and continue with what's locally available. Never block the entire workflow because
   one remote repo is unreachable.