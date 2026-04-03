---
name: drawio-diagrams
description: >
  Generate diagrams in Draw.io from PHP codebases by researching actual project code and creating
  them via the Draw.io MCP server. Use this skill whenever the user asks to create, generate, or
  update diagrams using Draw.io, diagrams.net, or drawio for their PHP project. Trigger on phrases
  like "create a Draw.io diagram", "make a flowchart in drawio", "diagram this in draw.io",
  "open this in diagrams.net", or any request to create visual documentation using Draw.io.
  Also trigger when the user mentions ".drawio files", "drawio XML", or asks to export diagrams
  as PNG/SVG/PDF via Draw.io. This skill supports creating diagrams via XML, Mermaid, or CSV
  input — with Mermaid as the recommended default for reliability. Works with any PHP codebase:
  Laravel, Symfony, plain PHP, or any other framework.
---

# Draw.io Diagram Generator

Create diagrams in Draw.io by researching actual PHP project code and generating them via the
Draw.io MCP server. Supports three input formats: Mermaid (recommended default), native XML
(for precise control), and CSV (for tabular data).

## References

- `references/drawio-formats.md` — Draw.io XML format reference, Mermaid usage guide, CSV
  format, shape search, and formatting conventions. **Read before generating any diagram.**
  Note: always verify against discovered tool schemas at runtime.

## Prerequisites and MCP Discovery

This skill works with multiple Draw.io MCP server variants. Discover which one is connected.

**Step 1: Discover available tools**

List available MCP tools to find what the Draw.io server provides. Common variants:

| Server | Tools | Install |
|---|---|---|
| **Official tool server** (`@drawio/mcp`) | `open_drawio_xml`, `open_drawio_csv`, `open_drawio_mermaid` | `npx @drawio/mcp` |
| **MCP App Server** (`mcp.draw.io`) | `create_diagram`, `search_shapes` | Remote: `https://mcp.draw.io/mcp` |
| **Community server** (`drawio-mcp-server`) | CRUD tools for diagram elements | `npx drawio-mcp-server` |

Don't assume which variant is installed. Discover first, then adapt.

**Step 2: Verify the server is working**

Try a lightweight operation. If it fails:
- Tell the user: "The Draw.io MCP server isn't connected."
- Provide setup options:
    - Tool server: `npx @drawio/mcp` (opens diagrams in browser)
    - App server: add `https://mcp.draw.io/mcp` as remote MCP (renders inline in chat)
    - Community server: `npx drawio-mcp-server --editor` (CRUD operations)
- **Offer the Mermaid fallback** (see below)

**Step 3: Cache the tool inventory**

After discovery, remember for this session:
- Which input formats are supported (XML, Mermaid, CSV)
- Whether shape search is available
- Whether inline rendering or browser opening is used
- The exact parameter schemas

## Fallback Strategy

If the Draw.io MCP is unavailable or fails:

> The Draw.io MCP server isn't available. I can create this diagram as:
> 1. A Mermaid markdown file (view locally, importable to Draw.io later)
> 2. A `.drawio` XML file (open directly in Draw.io desktop app)
> 3. Skip for now

If MCP fails **mid-creation** (after code research), immediately save the diagram content:
- As Mermaid → save to `./docs/diagrams/{type}/{name}.md` (php-diagrams format)
- As `.drawio` XML → save to `./docs/diagrams/{type}/{name}.drawio`

Then offer to retry. Never lose completed research.

## Configuration

Read `.drawio-diagrams.yaml` (or `.yml`) from the project root. If missing, use defaults.

**Key config fields and their defaults:**

| Field | Default | Purpose |
|---|---|---|
| `output.diagrams_dir` | `./docs/diagrams` | Where to save local diagram files |
| `output.index_file` | `./docs/diagrams/drawio-index.md` | Local index of Draw.io diagrams |
| `sources.include` | `[]` (whole project) | Directories to research |
| `sources.exclude` | `vendor`, `node_modules`, `.git`, `var/cache`, `storage/framework`, `tests` | Directories to skip |
| `repositories` | `[]` | Additional local paths to research |
| `framework` | `auto` | Framework hint: `auto`, `laravel`, `symfony`, `plain` |
| `defaults.format` | `mermaid` | Preferred creation format: `mermaid`, `xml`, `auto` |
| `defaults.detail` | `standard` | Detail level: `overview`, `standard`, `detailed` |
| `defaults.include_error_paths` | `true` | Include error/edge-case paths |
| `defaults.dark_mode` | `false` | Use dark mode when opening diagrams |

**Format selection logic** (when `defaults.format` is `auto`):
- Most diagram types → **Mermaid** (reliable, handles flowcharts, sequences, ERDs, etc.)
- Need exact positioning, custom colors, complex layouts → **XML**
- Tabular/hierarchical data (org charts) → **CSV** (but prefer Mermaid when possible)

## Workflow

Follow these steps in order. At every stage, if you have questions — ask.

### Step 0: MCP discovery and config

1. Discover available Draw.io MCP tools (see Prerequisites)
2. Verify the server works
3. If unavailable, offer fallback
4. Read config if it exists, merge with defaults

### Step 1: Check for existing diagrams

Check the local index file (default: `./docs/diagrams/drawio-index.md`) for existing diagrams.
Also search for `.drawio` files in the project:

```bash
find . -name "*.drawio" -not -path "*/vendor/*" -not -path "*/node_modules/*"
```

If the MCP App Server is connected and has `search_shapes` or similar search tools, use those
too.

If matches found:
> I found existing Draw.io diagrams:
> - `./docs/diagrams/sequence/user-registration.drawio`
>
> Would you like to:
> 1. Update this existing diagram
> 2. Create a new, separate diagram

### Step 2: Ask what to diagram

Ask the user what part of the codebase they want to diagram:

> What would you like me to diagram? For example:
> - A specific flow (e.g. "User Registration", "Payment processing")
> - An integration (e.g. "Stripe integration", "API authentication")
> - A system or subsystem (e.g. "Notification system", "Queue workers")
> - Database relationships (e.g. "User-related models", "Order schema")

If already specified, skip.

### Step 3: Ask for diagram type

> What type of diagram should I create?
> - **Sequence** — flow of calls/messages between components
> - **Flowchart** — decision logic, branching, process steps
> - **ERD** — database tables and relationships
> - **Class** — classes, properties, methods, relationships
> - **State** — entity lifecycle and transitions
> - **Architecture** — infrastructure, services, connections
> - **Mind Map** — hierarchical concept exploration
> - **BPMN** — business process model (Draw.io has native BPMN shapes)
> - **Network** — network topology with device shapes
> - **UML** — any UML diagram type (Draw.io has full UML shape libraries)

If already specified, skip. If unsure, recommend:

| What they want | Recommended type | Best format |
|---|---|---|
| Flow / process | Flowchart or Sequence | Mermaid |
| Integration with external API | Sequence | Mermaid |
| Database / models | ERD | Mermaid |
| State machine / status | State | Mermaid |
| System architecture | Architecture | XML (for cloud shapes) |
| Infrastructure / network | Network | XML (for Cisco/AWS shapes) |
| Business process | BPMN | XML (for BPMN shapes) |
| UML diagrams | UML | XML (for UML shapes) |
| Feature exploration | Mind Map | Mermaid |
| Org chart / hierarchy | Org chart | CSV or Mermaid |

**Key insight:** if the diagram needs specialized shapes (AWS, Azure, GCP, Cisco, Kubernetes,
BPMN, electrical, P&ID), use XML format and search for shapes using `search_shapes` if
available. Mermaid can't access Draw.io's 10,000+ shape libraries.

### Step 4: Research the code

Thoroughly research the codebase before creating any diagram.

**Scope:** Only search within `sources.include` (or entire project). Skip `sources.exclude`.

**How to research:**

1. **Detect framework** — `artisan` → Laravel, `bin/console` → Symfony, else Plain PHP
2. **Start broad** — find relevant directories per framework
3. **Go deep** — read files, follow call chains, note signatures, relationships, events
4. **If nothing found** — tell user, offer: point to files / create draft / diagram something else
5. **Ask if unclear** — ambiguous code? Ask, don't guess.

(Same research approach as php-diagrams and lucid-diagrams skills.)

### Step 5: Create the diagram

**Read `references/drawio-formats.md`** before generating any content.

Choose the format based on config `defaults.format`, diagram type, and what the discovered
MCP tools support:

**Mermaid (recommended default):**
- Most reliable — Draw.io renders Mermaid natively
- Covers: flowchart, sequence, class, ERD, state, mindmap, gitgraph
- Use the same Mermaid syntax as the php-diagrams skill
- Send via the Mermaid tool (e.g. `open_drawio_mermaid`) if available

**XML (for precision and specialized shapes):**
- Full control over positioning, styling, shapes
- Access to all Draw.io shape libraries (AWS, Azure, BPMN, UML, etc.)
- Use `search_shapes` to find correct shape styles before building XML
- See `references/drawio-formats.md` for XML format reference
- Send via the XML tool (e.g. `open_drawio_xml` or `create_diagram`)

**CSV (for tabular data):**
- Good for org charts, simple hierarchies from structured data
- Unreliable — Draw.io's CSV processing can fail
- Prefer Mermaid for org charts when possible
- Only use CSV when data is naturally tabular

**Quality standards:**
- Every element must correspond to something real in the code
- Use actual class/method/table names
- Include error paths if `defaults.include_error_paths` is true

**Respect detail level:**
- `overview` — high-level boxes, no method names
- `standard` — class/method names, happy path + key errors
- `detailed` — everything: all methods, errors, middleware, events

**If creation fails:**
1. Save the content locally (Mermaid as `.md`, XML as `.drawio`)
2. Tell the user what happened
3. Offer to retry or use the local file directly

### Step 6: Confirm and save locally

After creating:

1. Tell the user what was created and how to access it
    - Tool server: "Click the link to open in Draw.io editor"
    - App server: "The diagram is rendered inline above"
2. **Save a local copy** as `.drawio` file in `{diagrams_dir}/{type}/{name}.drawio`
   (if XML was used) or as `.md` with Mermaid (if Mermaid was used)
3. Ask for feedback — adjustments, more detail, less detail?

### Step 7: Update local index

Create or update the index file (default: `./docs/diagrams/drawio-index.md`):

```markdown
# Draw.io Diagrams Index

> Tracks Draw.io diagrams generated from this codebase. Last updated: {YYYY-MM-DD}

| Diagram | Type | Format | Local File | Created |
|---|---|---|---|---|
| {Title} | {type} | {mermaid/xml/csv} | `{path}` | {date} |
```

**Rules:**
- Preserve existing entries
- Update date if regenerated
- Mark drafts: `[DRAFT] {Title}`

## Updating Existing Diagrams

1. Read the existing `.drawio` or `.md` file
2. Re-research the code to find changes
3. **Always ask before overwriting:**
   > Here's what changed:
   > - {changes, or "no changes detected"}
       > Should I update?
4. If confirmed, regenerate and open via MCP
5. If update fails, save locally

## Important Principles

1. **Default to Mermaid** — it handles most diagram types reliably. Use XML only when you
   need precise positioning, custom colors, or specialized shapes (AWS, BPMN, UML, etc.).
   Avoid CSV for critical diagrams.

2. **Discover, don't assume** — introspect MCP tools before using them. Multiple Draw.io
   MCP server variants exist with different tool names and capabilities.

3. **Never lose work** — if MCP fails after research, save locally immediately.

4. **Save local copies** — always keep a `.drawio` or `.md` file in the project. Draw.io
   diagrams opened via URL are ephemeral; the local file is the persistent record.

5. **Use shape search** — if `search_shapes` is available and you need specialized shapes
   (cloud providers, network devices, BPMN), search first to get correct style strings.

6. **Accuracy over aesthetics** — real code names, real table columns, real method signatures.

7. **Research thoroughly** — more time reading code than writing diagram content.

8. **Graceful degradation** — MCP unavailable → save as local file. Shape search unavailable
   → use Mermaid. CSV fails → use Mermaid. Never silently fail.