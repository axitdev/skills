# Draw.io Format Reference

Covers XML format, Mermaid usage, CSV format, shape search, and key conventions.

**Important:** Always verify against discovered tool schemas at runtime. This reference
documents common patterns, but tool parameters may vary between MCP server variants.

## Table of Contents

1. [Format Selection](#format-selection)
2. [Mermaid Format](#mermaid-format)
3. [XML Format](#xml-format)
4. [CSV Format](#csv-format)
5. [Shape Search](#shape-search)
6. [Color and Style Conventions](#color-and-style-conventions)

---

## Format Selection

| Format | Strengths | Weaknesses | Use when |
|---|---|---|---|
| **Mermaid** | Reliable, concise, readable | No custom positioning, limited shapes | Default for most diagrams |
| **XML** | Full control, all 10,000+ shapes | Verbose, harder to write | Precise layouts, specialized shapes |
| **CSV** | Good for tabular data | Processing can fail, limited styling | Org charts from structured data |

**Decision flow:**
1. Can Mermaid express this diagram type? → Use Mermaid
2. Need specialized shapes (AWS, BPMN, UML)? → Use XML + shape search
3. Need exact positioning or complex layouts? → Use XML
4. Data is naturally tabular (org chart, hierarchy)? → Try Mermaid first, CSV as fallback

## Mermaid Format

Draw.io renders Mermaid natively. Use the same syntax as standard Mermaid.js.

**Supported diagram types in Mermaid:**
- `flowchart` / `graph` — flowcharts with decision logic
- `sequenceDiagram` — sequence diagrams
- `classDiagram` — class diagrams
- `erDiagram` — entity relationship diagrams
- `stateDiagram-v2` — state machine diagrams
- `mindmap` — mind maps
- `gitGraph` — git branching diagrams
- `timeline` — timeline diagrams

Refer to the **php-diagrams** skill's `references/mermaid-syntax.md` for full syntax examples.
The same Mermaid code works in Draw.io.

**Draw.io-specific Mermaid notes:**
- Draw.io converts Mermaid to its native format, making it fully editable after opening
- The rendered layout may differ slightly from other Mermaid renderers
- Complex Mermaid diagrams (50+ nodes) may render slowly

## XML Format

Draw.io uses mxGraph XML. The root structure is:

```xml
<mxGraphModel>
    <root>
        <mxCell id="0"/>
        <mxCell id="1" parent="0"/>
        <!-- your shapes and connections here, all with parent="1" -->
    </root>
</mxGraphModel>
```

**Every diagram must have cells 0 and 1.** Cell 0 is the root, cell 1 is the default layer.

### Shapes (vertices)

```xml
<mxCell id="2" value="UserController" style="rounded=1;whiteSpace=wrap;"
        vertex="1" parent="1">
    <mxGeometry x="100" y="100" width="160" height="60" as="geometry"/>
</mxCell>
```

Key attributes:
- `id` — unique identifier (use incrementing numbers or descriptive strings)
- `value` — text displayed in the shape (supports HTML: `<b>`, `<br>`, `<font>`)
- `style` — semicolon-separated style properties
- `vertex="1"` — marks this as a shape (not a connection)
- `parent="1"` — parent cell (1 = default layer, or a container ID)

### Connections (edges)

```xml
<mxCell id="10" value="createUser()" style="endArrow=classic;"
        edge="1" parent="1" source="2" target="3">
    <mxGeometry relative="1" as="geometry"/>
</mxCell>
```

Key attributes:
- `edge="1"` — marks this as a connection
- `source` / `target` — IDs of connected shapes
- `value` — label on the connection

### Containers (grouping)

For architecture diagrams with nested elements, use proper parent-child containment:

```xml
<!-- Container -->
<mxCell id="100" value="Backend Services" style="rounded=1;container=1;
        collapsible=0;pointerEvents=0;fillColor=#dae8fc;strokeColor=#6c8ebf;"
        vertex="1" parent="1">
    <mxGeometry x="50" y="50" width="400" height="300" as="geometry"/>
</mxCell>

        <!-- Child inside container — note parent="100" and relative coordinates -->
<mxCell id="101" value="API Server" style="rounded=1;whiteSpace=wrap;"
        vertex="1" parent="100">
<mxGeometry x="20" y="40" width="120" height="60" as="geometry"/>
</mxCell>
```

**Critical container rules:**
- Set `parent="containerId"` on children — don't just place shapes visually on top
- Children use coordinates **relative to the container**
- Add `pointerEvents=0;` to container styles so child connections aren't captured
- Only omit `pointerEvents=0` when the container itself needs to be connectable (use
  `swimlane` style in that case)
- Set `container=1;collapsible=0;` on the container cell

### Common style properties

| Property | Values | Purpose |
|---|---|---|
| `rounded=1` | 0 or 1 | Rounded corners |
| `whiteSpace=wrap` | wrap | Enable text wrapping |
| `fillColor=#hex` | Hex color | Background color |
| `strokeColor=#hex` | Hex color | Border color |
| `fontColor=#hex` | Hex color | Text color |
| `fontSize=14` | Number | Font size |
| `fontStyle=1` | 0=normal, 1=bold, 2=italic, 3=bold+italic | Font style |
| `dashed=1` | 0 or 1 | Dashed border/line |
| `endArrow=classic` | classic, block, open, none | Arrow head style |
| `startArrow=none` | Same as endArrow | Arrow tail style |
| `edgeStyle=orthogonalEdgeStyle` | orthogonal, elbowEdgeStyle | Connection routing |
| `curved=1` | 0 or 1 | Curved connections |
| `shape=mxgraph.aws4.xxx` | Shape identifier | Specialized shapes |
| `swimlane` | Style keyword | Swim lane container |
| `container=1` | 0 or 1 | Mark as container |
| `collapsible=0` | 0 or 1 | Prevent collapsing |
| `pointerEvents=0` | 0 or 1 | Transparent to mouse events |

### Swim lanes in XML

```xml
<!-- Swim lane container -->
<mxCell id="200" value="User" style="swimlane;startSize=30;fillColor=#4A90D9;
        fontColor=#ffffff;swimlaneLine=1;" vertex="1" parent="1">
    <mxGeometry x="0" y="0" width="800" height="150" as="geometry"/>
</mxCell>

        <!-- Shape inside swim lane -->
<mxCell id="201" value="Submit Form" style="rounded=1;whiteSpace=wrap;"
        vertex="1" parent="200">
<mxGeometry x="20" y="40" width="120" height="40" as="geometry"/>
</mxCell>
```

### Dark mode

Add `adaptiveColors="auto"` to the root element for automatic dark mode support:
```xml
<mxGraphModel adaptiveColors="auto">
```

## CSV Format

CSV creates diagrams from tabular data. **Warning: CSV processing can fail.** Prefer Mermaid.

Basic format:
```csv
## label: %name%
## style: shape=mxgraph.basic.rect;rounded=1;
## parentstyle: swimlane;
## identity: id
## parent: manager
## left: spouse
##
id,name,manager,spouse
1,CEO,,
2,CTO,1,
3,CFO,1,
4,Engineer,2,
```

**Avoid `%column%` placeholders in style attributes** (like `fillColor=%color%`) — this causes
"URI malformed" errors.

## Shape Search

If the MCP server provides `search_shapes` (available in the MCP App Server), use it to find
shapes from Draw.io's 10,000+ shape libraries before building XML.

**Libraries include:** AWS, Azure, GCP, Kubernetes, Cisco, UML, BPMN, P&ID, electrical,
flowchart, network, mockups, and many more.

**Usage:** search by keyword (e.g. "EC2 instance", "database cylinder", "kubernetes pod") and
use the returned style string directly in your XML `style` attribute.

Example workflow:
1. Search: "AWS Lambda"
2. Get style: `shape=mxgraph.aws4.lambda_function;...`
3. Use in XML: `<mxCell style="shape=mxgraph.aws4.lambda_function;..." .../>`

**If shape search is not available**, use generic shapes and add text labels. The diagram is
still accurate — just without vendor-specific icons.

## Color and Style Conventions

Suggested colors for consistent diagrams (same as lucid-diagrams skill):

| Element type | Suggested fill | Suggested stroke |
|---|---|---|
| User / Client | `#dae8fc` | `#6c8ebf` |
| API / Controller | `#d5e8d4` | `#82b366` |
| Service / Handler | `#fff2cc` | `#d6b656` |
| Database / Storage | `#f8cecc` | `#b85450` |
| External Service | `#e1d5e7` | `#9673a6` |
| Queue / Worker | `#d5e8d4` | `#82b366` |
| Error path | `#f8cecc` | `#b85450` |

**Connection styles:**

| Type | Style |
|---|---|
| Normal flow | `endArrow=classic;` |
| Response | `endArrow=classic;dashed=1;` |
| Async / Event | `endArrow=classic;dashed=1;dashPattern=4 4;` |
| Error | `endArrow=classic;strokeColor=#b85450;` |

**Layout tips:**
- Space shapes at least 120px apart
- Align on a grid (increments of 40px)
- Use `edgeStyle=orthogonalEdgeStyle` for clean right-angle connections
- Place labels on connections, not just shapes