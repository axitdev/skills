---
name: lucid-diagrams
description: >
  Generate diagrams in Lucidchart or Lucidspark from PHP codebases by researching actual project
  code and creating them via the Lucid MCP server. Use this skill whenever the user asks to create,
  generate, or update diagrams in Lucidchart or Lucidspark for their PHP project. Also trigger when
  the user says things like "create a Lucid diagram", "make a flowchart in Lucidchart", "diagram
  this in Lucid", "visualize in Lucidspark", or any request to create visual documentation using
  Lucid. Trigger even if the user just says "Lucid" in the context of diagramming — e.g. "put
  this flow in Lucid", "create a Lucid for the auth system". This skill handles creating new
  diagrams, searching existing ones, sharing them, and keeping them in sync with code changes.
  Works with any PHP codebase: Laravel, Symfony, plain PHP, or any other framework.
---

# Lucid Diagram Generator

Create diagrams in Lucidchart and Lucidspark by researching actual PHP project code and generating
them via the Lucid MCP server.

## References

- `references/lucid-conventions.md` — Diagram formatting suggestions, shape and color
  conventions, description-mode prompt templates. **Read before creating any diagram.**
  Note: these are guidelines, not guaranteed API formats. Always verify against discovered
  tool schemas.

## Prerequisites and MCP Discovery

This skill requires the **Lucid MCP server** to be connected and authenticated.

**Step 1: Discover available tools**

Before doing anything else, list the available MCP tools to discover what the Lucid MCP server
actually provides. In Claude Code, use `mcp__lucid__*` or equivalent tool listing to see:
- Which tools are available (names and descriptions)
- What parameters each tool accepts (schemas)
- What responses they return

This is critical because the Lucid MCP server's API evolves — tool names, parameters, and
capabilities change between versions. **Never assume a fixed API.** Always discover first.

**Step 2: Verify authentication**

Try a lightweight read operation (like searching for documents) to confirm auth is working.
If it fails:
- Tell the user clearly: "The Lucid MCP server isn't connected or authentication has expired."
- Provide setup instructions:
    1. Enable the Lucid MCP server in Lucid Admin Panel
    2. Connect it in AI tool's MCP settings (URL: `https://mcp.lucid.co/mcp`)
    3. Re-authenticate
- **Offer the Mermaid fallback** (see Fallback Strategy below)

**Step 3: Cache the tool inventory**

After discovery, remember for this session:
- Which creation tools are available (description-based, specification-based, or both)
- Which read tools exist (search, fetch, list)
- Which sharing tools exist
- The exact parameter schemas for each

Use these discovered schemas throughout the rest of the workflow. If a tool from
`references/lucid-conventions.md` doesn't exist in the discovered set, don't use it — adapt.

## Fallback Strategy

If the Lucid MCP server is unavailable, fails during creation, or the user doesn't have a Lucid
account, **always offer to generate as Mermaid instead:**

> The Lucid MCP server isn't available right now. I can create this diagram as a Mermaid
> markdown file instead — you can view it locally and import it to Lucid later.
>
> Should I:
> 1. Try reconnecting to Lucid
> 2. Generate as Mermaid (saved to `./docs/diagrams/`)
> 3. Skip for now

If MCP fails **mid-creation** (after code research is done), don't lose the work. Immediately
generate the Mermaid version so the user has something. Then offer to retry Lucid.

The Mermaid fallback follows the same file structure as the **php-diagrams** skill — save to
`{diagrams_dir}/{type}/{name}.md` with the standard template (title, context, diagram, key
files, notes). Update the php-diagrams index if it exists.

## Configuration

Read `.lucid-diagrams.yaml` (or `.yml`) from the project root. If it doesn't exist, use defaults.

**Key config fields and their defaults:**

| Field | Default | Purpose |
|---|---|---|
| `output.index_file` | `./docs/diagrams/lucid-index.md` | Local index of Lucid diagrams |
| `sources.include` | `[]` (whole project) | Directories to research |
| `sources.exclude` | `vendor`, `node_modules`, `.git`, `var/cache`, `storage/framework`, `tests` | Directories to skip |
| `repositories` | `[]` | Additional local paths to research |
| `framework` | `auto` | Framework hint: `auto`, `laravel`, `symfony`, `plain` |
| `lucid.product` | `auto` | Which Lucid product: `auto`, `lucidchart`, `lucidspark` |
| `lucid.default_folder` | `""` | Default Lucid folder for new diagrams |
| `defaults.detail` | `standard` | Detail level: `overview`, `standard`, `detailed` |
| `defaults.include_error_paths` | `true` | Include error/edge-case paths |

**Product selection logic** (when `lucid.product` is `auto`):
- Formal technical diagrams (ERD, sequence, class, flowchart, network) → **Lucidchart**
- Collaborative/exploratory (brainstorming, architecture proposals, retrospectives) → **Lucidspark**

## Workflow

Follow these steps in order. At every stage, if you have questions — ask the user.

### Step 0: MCP discovery and config

1. Discover available Lucid MCP tools (see Prerequisites above)
2. Verify authentication with a test read operation
3. If MCP is unavailable, offer fallback (see Fallback Strategy)
4. Read `.lucid-diagrams.yaml` if it exists, merge with defaults
5. If `repositories` has local paths, verify they exist

### Step 1: Search for existing diagrams

Before creating anything, search Lucid for existing diagrams. Use whatever search/list tool
was discovered in Step 0 (don't hardcode tool names).

If matches are found:

> I found existing diagrams in Lucid that might be related:
> - "{Document Title}" — {brief summary if fetchable}
>
> Would you like to:
> 1. View/fetch this existing diagram
> 2. Update it with current code changes
> 3. Create a new, separate diagram

Also check the local index file if it exists.

### Step 2: Ask what to diagram

Ask the user what part of the codebase they want to diagram:

> What would you like me to diagram in Lucid? For example:
> - A specific flow (e.g. "User Registration", "Payment processing")
> - An integration (e.g. "Stripe integration", "API authentication")
> - A system or subsystem (e.g. "Notification system", "Queue workers")
> - Database relationships (e.g. "User-related models", "Order schema")

If the user already specified what they want, skip this.

### Step 3: Ask for diagram type

Ask the user what type of diagram they want:

> What type of diagram should I create?
> - **Sequence** — flow of calls/messages between components over time
> - **Flowchart / Process** — decision logic, branching, and process steps
> - **ERD** — database tables and their relationships
> - **Class / Object** — classes, properties, methods, and relationships
> - **State** — states an entity goes through and transitions
> - **Swim Lane** — process flow divided by actor/system responsibility
> - **Network / Architecture** — infrastructure, services, and connections
> - **Mind Map** — hierarchical exploration (Lucidspark)
> - **User Flow** — step-by-step user journey through the system

If the user already specified, skip. If unsure, recommend:

| What they want | Recommended type | Product |
|---|---|---|
| Flow / process | Flowchart or Swim Lane | Lucidchart |
| Integration with external API | Sequence | Lucidchart |
| Database / models | ERD | Lucidchart |
| State machine / status field | State | Lucidchart |
| System overview | Network / Architecture | Lucidchart |
| Feature exploration | Mind Map | Lucidspark |
| User journey | User Flow | Lucidchart |

### Step 4: Research the code

This is the most important step. Thoroughly research the actual codebase before creating any
diagram. **Read `references/lucid-conventions.md`** for formatting guidelines.

**Scope:** Only search within directories from `sources.include` (or entire project if not set).
Skip `sources.exclude`.

**How to research:**

1. **Detect the framework** — if config `framework` is not `auto`, use that. Otherwise check:
    - `artisan` in root, `app/Http/` → **Laravel**
    - `bin/console`, `config/bundles.php` → **Symfony**
    - Otherwise → **Plain PHP**

2. **Start broad** — find relevant directories:

   **Laravel:** `app/Http/Controllers/`, `app/Models/`, `app/Services/`, `app/Events/`,
   `app/Jobs/`, `routes/`, `database/migrations/`, `config/`

   **Symfony:** `src/Controller/`, `src/Entity/`, `src/Repository/`, `src/Service/`,
   `src/EventSubscriber/`, `src/Message/`, `config/routes/`, `migrations/`

   **Plain PHP:** read `composer.json` autoload, check `src/`, `public/index.php`

3. **Go deep** — read files, follow call chains, note method signatures, relationships,
   middleware, events, error handling.

4. **If research finds nothing** — tell the user what you found (or didn't) and ask:
   > Would you like me to:
   > 1. Look again — point me to the right files
   > 2. Create a draft diagram showing a proposed design
   > 3. Diagram something else

5. **Ask if unclear** — ambiguous code, multiple flows? Ask rather than guess.

### Step 5: Create the diagram in Lucid

**Use the tools discovered in Step 0.** Don't hardcode tool names. Adapt to what's available.

Based on common Lucid MCP capabilities, there are typically two creation approaches:

**Description-based creation** — provide a natural language description of the diagram. Good for
quick conceptual diagrams. Use when:
- The user said "just sketch it" or wants something quick
- The diagram is exploratory or high-level
- Exact positioning doesn't matter

Write a structured description following the templates in `references/lucid-conventions.md`.

**Specification-based creation** — provide structured data with shapes, connections, and
positions. Good for precise technical diagrams. Use when:
- ERD, swim lane, or detailed flowchart
- Exact layout matters
- Multiple actors with specific relationships

When using specification-based creation, check the discovered tool schema for the exact format.
The reference file has suggested conventions, but **the discovered schema is authoritative.**

**Quality standards:**
- Every element must correspond to something real in the code
- Use actual class/method/table names from the codebase
- Include error paths if `defaults.include_error_paths` is true

**Respect detail level** from config:
- `overview` — high-level boxes, no method names, major flows only
- `standard` — class/method names, happy path + key error paths
- `detailed` — every method call, all error paths, middleware, events, queue jobs

**If creation fails** (MCP error, timeout, auth expiry):
1. Don't discard the research — immediately generate a Mermaid fallback
2. Tell the user what happened and provide the Mermaid file
3. Offer to retry Lucid after they re-authenticate

### Step 6: Share and confirm

After creating the diagram:

1. Use whatever sharing tool was discovered to get a link
2. Share the link with the user so they can view and edit
3. Tell them what was created — diagram type, which Lucid product, brief summary
4. Ask for feedback — adjustments, more detail, less detail?

### Step 7: Update local index

Create or update the local index file (default: `./docs/diagrams/lucid-index.md`).

```markdown
# Lucid Diagrams Index

> Tracks Lucid documents generated from this codebase. Last updated: {YYYY-MM-DD}

| Diagram | Type | Lucid Link | Code Area | Created |
|---|---|---|---|---|
| {Title} | {type} | [{doc ID}]({link}) | `src/Controller/...` | {date} |
```

**Rules:**
- Preserve existing entries
- Update link/date if regenerated
- Mark drafts: `[DRAFT] {Title}`

## Updating Existing Diagrams

When the user asks to update an existing Lucid diagram:

1. Fetch the current document using whatever read tool is available
2. Re-research the relevant code to find changes
3. **Always ask before overwriting:**
   > Here's what changed since the diagram was created:
   > - {list changes, or "no significant changes detected"}
   >
   > Should I update the Lucid diagram?
4. If confirmed, update the document (or create a new version if the MCP doesn't support edits)
5. If the update fails, generate a Mermaid fallback with the updated content
6. Update the local index

## Important Principles

1. **Discover, don't assume** — always introspect MCP tools before using them. Tool names,
   parameters, and capabilities change between Lucid MCP versions. The discovered schema is
   the source of truth, not this skill file.

2. **Never lose work** — if Lucid MCP fails after code research, immediately produce a Mermaid
   fallback. The user's time researching and deciding what to diagram is valuable.

3. **Accuracy over aesthetics** — every element must reflect real code. Lucid handles visual
   polish; your job is correct content.

4. **Ask, don't assume** — when in doubt, ask about scope, detail level, and flow.

5. **Research thoroughly** — spend more time reading code than writing diagram descriptions.

6. **Check before creating** — always search Lucid first to avoid duplicating existing work.

7. **Keep the local index updated** — this tracks what exists across sessions.

8. **Graceful degradation** — MCP unavailable → Mermaid fallback. Auth expired → explain and
   offer alternatives. Tool missing → adapt to what's available. Never just fail silently.