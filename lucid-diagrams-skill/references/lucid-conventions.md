# Lucid Diagram Conventions & Guidelines

Read this before creating any diagram. Contains formatting suggestions, prompt templates, and
best practices for consistent diagrams.

**Important:** These are guidelines and suggestions, not guaranteed API formats. The Lucid MCP
server evolves — always verify against the actual tool schemas discovered at runtime. If a
convention here conflicts with what the discovered tool accepts, **the discovered schema wins.**

## Table of Contents

1. [Product Selection](#product-selection)
2. [Description Prompt Templates](#description-prompt-templates)
3. [Diagram Type Guidelines](#diagram-type-guidelines)
4. [Color Suggestions](#color-suggestions)
5. [Shape Suggestions](#shape-suggestions)
6. [Connection Suggestions](#connection-suggestions)

---

## Product Selection

| Product | Use for |
|---|---|
| **Lucidchart** | Formal technical diagrams: ERDs, sequence diagrams, flowcharts, network diagrams, swim lanes, class diagrams, state machines |
| **Lucidspark** | Collaborative boards: brainstorming, architecture proposals, mind maps, retrospectives |

When creating a diagram, specify the product explicitly. Default to Lucidchart for technical
diagrams.

## Description Prompt Templates

When using a description-based creation tool, write structured prompts. Here are templates
for each diagram type. Use real names from the codebase, not generic placeholders.

### Sequence Diagram

```
Create a sequence diagram in Lucidchart showing {what this flow does}.

Participants:
- {RealClassName1} ({role, e.g. "API entry point"})
- {RealClassName2} ({role})
- {Database/ExternalService}

Main flow:
1. {Actor} sends {HTTP method} {path} to {Controller}
2. {Controller} calls {Service}::{method}({params})
3. {Service} queries {table} via {Repository/Model}
4. {Service} dispatches {EventName} event
5. {Controller} returns {status code} with {response type}

Error handling:
- If validation fails at step 2: return 422 with validation errors
- If {ExternalService} is unavailable at step 3: throw {ExceptionClass}, return 503

Async operations:
- After step 4, {JobClass} is dispatched to queue "{queue_name}"
```

### Flowchart / Process Diagram

```
Create a flowchart in Lucidchart showing {what this process does}.

Start: {Entry point — e.g. "HTTP request arrives at /api/orders"}

Steps:
1. {Action} — {RealClassName}::{method}()
2. Decision: {condition from code}?
   - Yes: proceed to step 3
   - No: {what happens — e.g. "return 401 Unauthorized"}
3. {Action}
4. Decision: {another condition}?
   - Yes: {action}
   - No: {action}
5. End: {outcome — e.g. "return 201 with OrderResource"}

Error path:
- If exception at step 3: log error, return 500
```

### ERD (Entity Relationship Diagram)

```
Create an entity relationship diagram in Lucidchart showing {which part of the schema}.

Tables:
- {table_name}:
  - id (PK, int/bigint, auto-increment)
  - {column_name} ({type}, {nullable?}, {default?})
  - {foreign_key}_id (FK → {referenced_table}.id)
  - created_at (timestamp)
  - updated_at (timestamp)

- {another_table}:
  - ...

Relationships:
- {table_a} 1:N {table_b} via {table_b}.{fk_column}
- {table_c} N:M {table_d} via {pivot_table}
- {table_e} 1:1 {table_f} via {table_f}.{fk_column}
```

### Swim Lane Diagram

```
Create a swim lane diagram in Lucidchart showing {what this process does}.

Lanes (top to bottom):
1. {Actor/System — e.g. "User / Browser"}
2. {Actor/System — e.g. "API Gateway"}
3. {Actor/System — e.g. "OrderService"}
4. {Actor/System — e.g. "Database"}
5. {Actor/System — e.g. "Payment Provider (Stripe)"}

Flow:
1. [Lane 1] User submits order form
2. [Lane 2] API Gateway validates JWT token
3. [Lane 2] Decision: token valid?
   - No: [Lane 1] Show 401 error
   - Yes: continue
4. [Lane 3] OrderService::createOrder(data)
5. [Lane 4] INSERT INTO orders
6. [Lane 3] OrderService calls Stripe
7. [Lane 5] Stripe processes payment
8. [Lane 5] Stripe returns charge ID
9. [Lane 4] UPDATE orders SET status = 'paid'
10. [Lane 1] Show order confirmation
```

### State Diagram

```
Create a state diagram in Lucidchart showing the lifecycle of {entity}.

States:
- {StateName} — {when/why entity is in this state}
- {StateName} — {description}
- ...

Transitions:
- {FromState} → {ToState}: triggered by {event/action} (condition: {guard if any})
- ...

Initial state: {state}
Final states: {state(s)}
```

### Network / Architecture Diagram

```
Create an architecture diagram in Lucidchart showing {system/infrastructure}.

Components:
- {Tier/Group name}:
  - {Service name} ({type — e.g. "ECS container", "RDS instance"})
  - ...

Connections:
- {ServiceA} → {ServiceB}: {protocol/purpose — e.g. "HTTPS/REST", "TCP/5432"}
- ...

External services:
- {Name} ({what it does})
```

### Mind Map

```
Create a mind map in Lucidspark exploring {concept/system}.

Central topic: {main concept}

Branches:
- {Category 1}:
  - {Item}
  - {Item}
- {Category 2}:
  - {Item}
  - {Sub-item}
- ...
```

### User Flow

```
Create a user flow diagram in Lucidchart showing {what journey}.

Entry point: {page/screen/action}

Flow:
1. User sees {page/screen}
2. User clicks {element}
3. System {action}
4. Decision: {condition}?
   - Yes: User sees {page}
   - No: User sees {error/alternative}
5. ...

Success end: {outcome}
Failure end: {outcome}
```

## Diagram Type Guidelines

### When to use specification vs description

If the discovered MCP tools include both a specification-based and description-based creation
tool, choose based on:

| Situation | Use |
|---|---|
| ERD with many tables and relationships | Specification (precise layout) |
| Swim lane with 4+ actors | Specification (lane positioning) |
| Quick overview for discussion | Description (faster) |
| User said "just sketch it" | Description |
| Detailed flowchart with many branches | Specification |
| Mind map or brainstorm | Description |
| Anything where exact positioning matters | Specification |
| Anything where speed matters more than layout | Description |

### Level of detail per config setting

| Detail level | What to include | What to omit |
|---|---|---|
| `overview` | Major components, high-level arrows, group names | Method names, parameters, column types, middleware |
| `standard` | Class/method names, main flow + key errors, column names | Internal method chains, logging, minor validations |
| `detailed` | Everything: all methods, all errors, middleware, events, jobs, column types | Nothing — full documentation |

## Color Suggestions

Use consistent colors to make diagrams scannable. These are **suggestions** — adapt to your
team's preferences or Lucid theme.

| Element type | Suggested color | Hex |
|---|---|---|
| User / Client | Blue | `#4A90D9` |
| Frontend / UI | Purple | `#7B68EE` |
| API / Controller | Green | `#2ECC71` |
| Service / Handler | Orange | `#F39C12` |
| Database / Storage | Red | `#E74C3C` |
| External Service | Violet | `#9B59B6` |
| Queue / Worker | Teal | `#1ABC9C` |
| Error path | Red | `#E74C3C` |
| Happy path | Green | `#27AE60` |
| Async / Event | Teal dashed | `#1ABC9C` |

For swim lane fills, use the header color at 10-15% opacity.

## Shape Suggestions

| Element | Suggested shape | Notes |
|---|---|---|
| Process / Action | Rectangle | Default for most steps |
| Decision | Diamond | Yes/No branching |
| Start / End | Rounded rectangle or oval | Mark clearly |
| Database | Cylinder | Tables, caches, storage |
| External Service | Cloud | Third-party APIs |
| Queue / Async | Trapezoid | Message queues, background jobs |
| Note / Annotation | Sticky note | Clarifying info |

## Connection Suggestions

| Connection type | Suggested style | Notes |
|---|---|---|
| Sequential flow | Solid arrow → | Label with method name |
| Response / Return | Dashed arrow ← | Label with return value |
| Async / Event | Dotted arrow → | Label with event name |
| Error path | Red solid arrow → | Label with error type |
| Optional path | Gray dashed → | Label with condition |

**Labels on connections should use real names from the code:**
- Method calls: `createUser(data)` not "creates user"
- HTTP: `POST /api/users` not "sends request"
- Events: `UserRegistered` not "dispatches event"