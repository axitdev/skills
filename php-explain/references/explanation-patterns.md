# Explanation Patterns

Pick the right structure based on what the user wants explained. Each pattern is a template —
adapt to the actual code, don't force content into sections that don't apply.

## Table of Contents

1. [Flow Trace](#flow-trace)
2. [Component Overview](#component-overview)
3. [Connection Map](#connection-map)
4. [Data Lifecycle](#data-lifecycle)
5. [Integration Explainer](#integration-explainer)
6. [Why Explainer](#why-explainer)
7. [Specific Code Explainer](#specific-code-explainer)

---

## Flow Trace

**Use when:** "How does X work?", "Walk me through the Y flow", "What happens when Z?"

**Structure:**

```
One-sentence summary of what this flow does.

Entry point: {route/command/listener} → {file:line}

The chain:

1. **{Layer}** — `ClassName::method()` (`path/to/file.php`)
   {What happens here and why}
   {Key data transformations}

2. **{Next layer}** — `ClassName::method()` (`path/to/file.php`)
   {What happens here}

   ⚠️ {Important caveat if any}

3. ...

Hidden actors:
- {Observer/listener/middleware that affects this flow}

Error handling:
- If {condition}: {what happens}

Key files:
- `path/to/file.php` — {role}
```

**Example flow trace (overview depth):**

> **User Registration** creates a new account, sends a welcome email, and logs the user in.
>
> `POST /api/register` → `AuthController::register()` → validates input via `RegisterRequest`
> → calls `UserService::createUser()` → inserts into `users` table → dispatches
> `UserRegistered` event → `SendWelcomeEmail` listener queues the email → returns 201 with
> auth token.
>
> Hidden: `UserObserver::created()` also fires and logs to the audit table.

**Example flow trace (standard depth):**

> **User Registration** — creates a new user account with email verification.
>
> **1. Route & Middleware** — `POST /api/register` (`routes/api.php:34`)
> Hits the `throttle:5,1` middleware first (max 5 attempts per minute per IP), then
> `AuthController::register()`.
>
> **2. Validation** — `RegisterRequest` (`app/Http/Requests/RegisterRequest.php`)
> Validates: email (required, unique:users), password (min:8, confirmed), name (required,
> max:255). If validation fails, Laravel auto-returns 422 with error details.
>
> **3. Service** — `UserService::createUser()` (`app/Services/UserService.php:28`)
> Wraps everything in a DB transaction. Hashes the password via `Hash::make()`, creates the
> `User` model, assigns the default `user` role via Spatie permissions.
>
> ⚠️ The transaction covers user creation + role assignment. If role assignment fails, the
> user is rolled back too.
>
> **4. Events** — `UserRegistered` event dispatched after commit
> Listeners: `SendWelcomeEmail` (queued, `notifications` queue), `CreateDefaultSettings`
> (sync, creates user preferences row).
>
> **5. Response** — returns 201 with `UserResource` + Sanctum token
>
> **Hidden actors:**
> - `UserObserver::created()` logs to `audit_log` table
> - `EnsureEmailIsVerified` middleware on other routes will block this user until they
    >   click the verification link in the welcome email

## Component Overview

**Use when:** "What does X do?", "Explain this service/class", "What is this for?"

**Structure:**

```
One-sentence summary: what this component does and why it exists.

Role in the system:
- Where it sits in the architecture (controller/service/repository/etc.)
- What depends on it and what it depends on

Public interface:
- `method1(params)` — what it does
- `method2(params)` — what it does

How it works internally:
{Key implementation details at the requested depth}

Used by:
- {Who calls this component and when}

Dependencies:
- {What this component needs: other services, config, external APIs}

Design notes:
- {Why it's built this way, patterns used}
```

## Connection Map

**Use when:** "How are X and Y connected?", "How does X talk to Y?"

**Structure:**

```
One-sentence summary of the relationship.

Connection type: {direct call / event / queue / shared database / API / etc.}

Direction: {A → B, B → A, or bidirectional}

How they connect:
1. {A does X, which triggers Y in B}
2. {B responds with Z}

Data exchanged:
- A sends: {what data}
- B returns: {what data}

Coupling assessment:
- {Tightly coupled (direct dependency) / loosely coupled (via events/queue) / etc.}
- {Implications for testing, deployment, modification}
```

## Data Lifecycle

**Use when:** "What happens to X from start to end?", "Trace the lifecycle of an order"

**Structure:**

```
One-sentence summary: the entity and its lifecycle.

States: {list all states/statuses the entity goes through}

Timeline:

1. **Created** — {how, where, by whom}
   - Initial data: {key fields}
   - Triggers: {events, jobs}

2. **{Next state}** — {what causes the transition}
   - Changes: {what fields update}
   - Side effects: {notifications, external calls}

3. ...

Where it's stored:
- Table: `{table_name}` with key columns: {list}
- Cache: {if cached, key pattern and TTL}
- Related tables: {relationships}

Who touches it:
- {List of services/controllers/jobs that read or modify this entity}
```

## Integration Explainer

**Use when:** "How does the Stripe/Twilio/S3 integration work?"

**Structure:**

```
One-sentence summary: what the integration does.

Service: {external service name}
Client class: `{ClassName}` (`{file path}`)
Config: `{config file or env vars}`

Authentication:
- {How auth works: API key, OAuth, webhook secret}
- {Where credentials are stored}

Outbound calls (our code → external):
1. `method()` — calls {endpoint}, sends {data}, expects {response}
2. ...

Inbound webhooks (external → our code):
1. `{route}` → `{controller}` — handles {event type}
   - Validates: {signature verification, etc.}
   - Actions: {what it does with the webhook data}

Error handling:
- {Retries, timeouts, fallbacks}

Testing:
- {How to test: mock class, sandbox env, etc.}
```

## Why Explainer

**Use when:** "Why is it done this way?", "Why separate X from Y?"

**Structure:**

```
The decision: {what was done}

The likely reasoning:
1. {Reason 1 — based on code evidence}
2. {Reason 2}

The tradeoffs:
- Benefit: {what this approach gains}
- Cost: {what it sacrifices or complicates}

Alternatives that weren't chosen:
- {What else could have been done and why it probably wasn't}

Evidence from the code:
- {Specific code patterns that support this interpretation}
```

**Important:** be honest about the difference between what you can determine from code and
what's speculation. "Based on the code, it appears..." vs "I'm guessing this is because..."

## Specific Code Explainer

**Use when:** "What does this method do?", "Explain this code" (pointing at specific code)

**Structure:**

```
Summary: {what this code does in one sentence}

Context: {where this sits in the system — who calls it and why}

Line by line (for deep dive) or block by block (for standard):
- {code section} — {what it does and why}

Inputs: {parameters, their types, what they represent}
Outputs: {return value, side effects}

Edge cases:
- {What happens with null/empty/invalid input}
- {Boundary conditions}

Potential issues:
- {Anything that looks risky, unclear, or could break}
```

---

## Formatting Tips

For all patterns:

- **Use inline code** for class names, methods, file paths, table names, config keys
- **Use bold** for section names and emphasis on important concepts
- **Use emoji markers** sparingly for important callouts:
    - ⚠️ Warning / gotcha
    - 🔒 Security-related
    - ⏱️ Performance / timing
    - 💾 Data / storage
    - 🔄 Async / queued
- **Show the chain visually** even in text:
  `A → B → C` reads faster than "A calls B which calls C"
- **Keep it scannable** — someone should be able to skim the bold text and get the gist