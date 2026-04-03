# Mermaid Syntax Reference

Read this file before generating any diagram. It contains syntax examples for every supported
diagram type. Use these as templates — adapt to the actual code you've researched.

## Table of Contents

1. [Sequence](#sequence)
2. [Class](#class)
3. [ERD](#erd)
4. [State](#state)
5. [Flowchart](#flowchart)
6. [Activity](#activity)
7. [Component](#component)
8. [Deployment](#deployment)
9. [Data Flow](#data-flow)
10. [Mind Map](#mind-map)
11. [Timeline](#timeline)
12. [Git Graph](#git-graph)
13. [Block](#block)
14. [Sankey](#sankey)
15. [Packet](#packet)
16. [Architecture](#architecture)

---

## Sequence

Shows call/message flow between components over time.

```mermaid
sequenceDiagram
    participant C as Client
    participant Ctrl as UserController
    participant Svc as UserService
    participant DB as Database

    C->>Ctrl: POST /register
    Ctrl->>Svc: createUser(data)
    Svc->>DB: INSERT INTO users
    DB-->>Svc: User model
    Svc-->>Ctrl: UserResource
    Ctrl-->>C: 201 Created
```

Tips: use `-->>` for responses, `alt`/`else` blocks for branching, `loop` for repetition,
`note over` for annotations, `activate`/`deactivate` for lifelines.

## Class

Shows classes, properties, methods, and relationships.

```mermaid
classDiagram
    class User {
        +int id
        +string name
        +string email
        +orders() Collection~Order~
        +hasRole(string role) bool
    }
    User "1" --> "*" Order : has many
```

Tips: use `+` public, `-` private, `#` protected, `~` package. Relationship arrows:
`<|--` inheritance, `*--` composition, `o--` aggregation, `-->` association, `..>` dependency.

## ERD

Shows database tables, columns, and relationships.

```mermaid
erDiagram
    USERS {
        int id PK
        string name
        string email
        timestamp created_at
    }
    ORDERS {
        int id PK
        int user_id FK
        decimal total
        string status
    }
    USERS ||--o{ ORDERS : "has many"
```

Tips: relationship notation — `||` exactly one, `o|` zero or one, `}|` one or more,
`}o` zero or more. Read left to right.

## State

Shows states an entity goes through and transitions.

```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Processing : payment_received
    Processing --> Completed : fulfilled
    Processing --> Failed : error
    Failed --> Pending : retry
    Completed --> [*]
```

Tips: use `state "Long Name" as s1` for aliases, `state fork_state <<fork>>` for parallel,
`state if_state <<choice>>` for decisions, nested states with `state ParentState { }`.

## Flowchart

Shows decision logic, branching, and process steps.

```mermaid
flowchart TD
    A[Request] --> B{Authenticated?}
    B -->|Yes| C[Process]
    B -->|No| D[401 Unauthorized]
    C --> E{Valid?}
    E -->|Yes| F[Save]
    E -->|No| G[422 Error]
```

Tips: `TD` top-down, `LR` left-right. Node shapes: `[rect]`, `(round)`, `{diamond}`,
`([stadium])`, `[[subroutine]]`, `[(cylinder)]`, `((circle))`, `>flag]`.

## Activity

Uses flowchart with swimlanes (subgraphs) to show parallel/branching workflows.

```mermaid
flowchart TD
    subgraph Client
        A[Submit Order]
    end
    subgraph API["API Layer"]
        B[Validate Request]
        C{Stock Available?}
    end
    subgraph Service["Order Service"]
        D[Reserve Stock]
        E[Create Order]
    end
    A --> B --> C
    C -->|Yes| D --> E
    C -->|No| F[Return 422]
```

## Component

High-level system architecture using flowchart with subgraphs.

```mermaid
flowchart LR
    subgraph Frontend
        SPA[React SPA]
    end
    subgraph Backend
        API[PHP API]
        Queue[Queue Worker]
    end
    subgraph External
        Stripe[Stripe API]
        Mail[Mailgun]
    end
    SPA -->|REST| API
    API -->|dispatch| Queue
    API -->|charge| Stripe
    Queue -->|send| Mail
```

## Deployment

Infrastructure and container layout using flowchart.

```mermaid
flowchart TD
    subgraph AWS["AWS eu-central-1"]
        subgraph ECS["ECS Cluster"]
            App1[App Container x3]
            Worker[Worker Container x2]
        end
        RDS[(PostgreSQL RDS)]
        Redis[(ElastiCache Redis)]
        S3[S3 Bucket]
    end
    CF[CloudFront CDN] --> App1
    App1 --> RDS
    App1 --> Redis
    Worker --> RDS
    Worker --> S3
```

## Data Flow

How data moves between processes and stores (uses flowchart).

```mermaid
flowchart LR
    User([User]) -->|HTTP Request| API[API Gateway]
    API -->|validate| Validator[Request Validator]
    Validator -->|write| DB[(Database)]
    Validator -->|publish| Queue[[Message Queue]]
    Queue -->|consume| Worker[Worker Process]
    Worker -->|write| Cache[(Redis Cache)]
    Worker -->|send| Email([Email Service])
```

## Mind Map

Hierarchical exploration of a concept or subsystem.

```mermaid
mindmap
    root((Notification System))
        Channels
            Email
            SMS
            Push
            Slack
        Triggers
            User Actions
            Scheduled Jobs
            System Events
        Storage
            notifications table
            notification_preferences
        Config
            config files
            .env credentials
```

## Timeline

Chronological view of events, phases, or lifecycle.

```mermaid
timeline
    title Order Lifecycle
    section Created
        Validation : Request validated
        Persisted : Order saved to DB
    section Processing
        Payment : Charge initiated
        Fulfillment : Stock reserved
    section Completed
        Shipped : Tracking number assigned
        Delivered : Delivery confirmed
```

## Git Graph

Branching strategies, release flows, merge patterns.

```mermaid
gitGraph
    commit id: "init"
    branch develop
    commit id: "feature-base"
    branch feature/payments
    commit id: "add-stripe"
    commit id: "add-webhooks"
    checkout develop
    merge feature/payments
    checkout main
    merge develop tag: "v1.2.0"
```

## Block

Nested grouping for architecture layers, modules, or bounded contexts.

```mermaid
block-beta
    columns 3
    Frontend["Frontend\nReact SPA"]:3
    space
    API["PHP API"]:1
    space
    Services["Domain Services"]:1
    Queue["Queue Workers"]:1
    Scheduler["Task Scheduler"]:1
    DB[("PostgreSQL")]:1
    Redis[("Redis")]:1
    S3[("S3 Storage")]:1
```

## Sankey

Weighted flow between nodes (request routing, pipeline throughput).

```mermaid
sankey-beta
    Incoming Requests,Auth Middleware,1000
    Auth Middleware,Authorized,850
    Auth Middleware,Rejected 401,150
    Authorized,Validation Pass,800
    Authorized,Validation Fail 422,50
    Validation Pass,Success 200,750
    Validation Pass,Server Error 500,50
```

## Packet

Protocol or message/payload structure breakdown.

```mermaid
packet-beta
    0-15: "Request ID"
    16-23: "Version"
    24-31: "Flags"
    32-63: "Payload Length"
    64-95: "User ID"
    96-127: "Timestamp"
    128-255: "JSON Payload Body"
```

## Architecture

Cloud/infra architecture (C4-style using flowchart).

```mermaid
flowchart TD
    subgraph Users
        Browser[Web Browser]
        Mobile[Mobile App]
    end
    subgraph LoadBalancer["Load Balancer"]
        Nginx[Nginx]
    end
    subgraph Application["Application Tier"]
        API1[API Server 1]
        API2[API Server 2]
    end
    subgraph Data["Data Tier"]
        PG[(PostgreSQL Primary)]
        PGR[(PostgreSQL Replica)]
        Redis[(Redis Cache)]
    end
    subgraph Queue["Queue Tier"]
        Supervisor[Process Manager]
        Workers[Queue Workers x4]
    end
    Browser --> Nginx
    Mobile --> Nginx
    Nginx --> API1 & API2
    API1 & API2 --> PG
    API1 & API2 --> Redis
    PG --> PGR
    API1 & API2 -->|dispatch| Supervisor
    Supervisor --> Workers
    Workers --> PG
```

## Applying Mermaid Theme

If config has `mermaid.theme` set to something other than `default`, prepend this frontmatter
to every diagram:

```
---
config:
  theme: {theme_value}
---
```

Place it right after the opening ` ```mermaid ` line, before the diagram type keyword.