---
name: php-explain
description: >
  Explain how parts of a PHP system work by tracing through actual code. Use this skill whenever
  the user asks to explain, trace, walk through, or understand how something works in their PHP
  codebase. Trigger on phrases like "explain how X works", "walk me through the payment flow",
  "how does authentication work here", "trace the request from controller to database", "what
  happens when a user registers", "how does this service work", "explain this part of the code",
  "what does this class do", "how are these connected". Also trigger when the user points at a
  specific file/class and asks "what is this", "what does this do", "why is this here". Works
  with any PHP codebase: Laravel, Symfony, plain PHP, or any other framework. The key difference
  from just reading code is that this skill follows the entire call chain, connects the pieces,
  explains the why behind design decisions, and presents it as a coherent narrative — not just
  a list of files.
---

# PHP System Explainer

Explain how parts of a PHP system work by tracing through actual code. Follows call chains,
connects components, explains design decisions, and presents a coherent narrative.

## References

- `references/explanation-patterns.md` — Templates for different explanation types (flow trace,
  component overview, data lifecycle, integration map, etc.). **Read before explaining** to
  pick the right structure.

## Configuration

Read `.php-explain.yaml` (or `.yml`) from the project root. If missing, use defaults.

**Key config fields and their defaults:**

| Field | Default | Purpose |
|---|---|---|
| `output.explanations_dir` | `./docs/explanations` | Where to save explanation files |
| `output.index_file` | `./docs/explanations/index.md` | Explanation index |
| `sources.include` | `[]` (whole project) | Directories to research |
| `sources.exclude` | `vendor`, `node_modules`, `.git`, `var/cache`, `storage/framework`, `tests` | Directories to skip |
| `framework` | `auto` | Framework hint: `auto`, `laravel`, `symfony`, `plain` |

## How This Skill Differs From Just Reading Code

Reading code shows what a file contains. This skill:
- **Follows the full chain** — from entry point to final output, across files and layers
- **Connects the pieces** — shows how controllers, services, models, events, and jobs relate
- **Explains the why** — not just what the code does, but why it's structured this way
- **Surfaces hidden behavior** — middleware, event listeners, observers, scheduled tasks,
  queue workers that affect the flow but aren't obvious from the main call chain
- **Identifies patterns** — names the design patterns in use (repository, strategy, observer,
  etc.) so the user can reason about the architecture

## Workflow

### Step 1: Understand what the user wants explained

The user's request falls into one of these categories:

| Request type | Example | What to do |
|---|---|---|
| **Flow** | "How does user registration work?" | Trace the full request → response chain |
| **Component** | "What does OrderService do?" | Explain a single class/service and its role |
| **Connection** | "How are orders and payments connected?" | Show how two parts of the system interact |
| **Data lifecycle** | "What happens to an order from creation to delivery?" | Trace an entity through all its state changes |
| **Integration** | "How does the Stripe integration work?" | Explain the external service connection |
| **Why** | "Why is there a separate queue for notifications?" | Explain design decisions and tradeoffs |
| **Specific code** | "What does this method do?" (pointing at code) | Explain a specific piece in context |

If the request is ambiguous, ask one clarifying question. Don't over-ask — start explaining
and the user will redirect if needed.

### Step 2: Ask for depth level

Ask the user how deep they want to go:

> How detailed should this explanation be?
> - **Overview** — what it does, key components, how they connect (2-3 minutes to read)
> - **Standard** — follows the call chain step by step, explains each layer (5-10 min read)
> - **Deep dive** — every method, every condition, every side effect, edge cases (15+ min read)

If the user already indicated depth (e.g. "give me a quick overview" or "I want to understand
every detail"), skip this question.

### Step 3: Research the code

This is where the skill earns its value. Research thoroughly based on depth level.

**Scope:** Only search within directories from `sources.include` (or entire project if not set).
Skip everything in `sources.exclude`. If config has `framework` set, use that instead of
auto-detecting.

**For all depth levels:**

1. **Find the entry point** — where does this flow/component start?
    - Routes (`routes/web.php`, `routes/api.php`, `config/routes.yaml`)
    - Console commands
    - Queue workers
    - Scheduled tasks
    - Event listeners

2. **Detect the framework** — Laravel, Symfony, or plain PHP (same detection as other skills)

3. **Follow the call chain** — from entry point through every layer:
    - Middleware / security (what runs before the main logic?)
    - Controller / command handler (entry point logic)
    - Form validation / request validation
    - Service / action layer (business logic)
    - Repository / model layer (data access)
    - Events dispatched and their listeners
    - Queue jobs triggered
    - Notifications / emails sent
    - External API calls

4. **Map the data flow** — what data goes in, how it transforms, what comes out

5. **Identify hidden actors** — things that affect the flow but aren't in the main chain:
    - Model observers / lifecycle hooks
    - Global middleware
    - Event subscribers that listen to dispatched events
    - Scheduled tasks that process the data later
    - Cache layers
    - Database triggers (if any)

**Additional research by depth:**

| Depth | Also research |
|---|---|
| Overview | Skip — just the main components and their relationships |
| Standard | Validation rules, key conditionals, error handling, main events |
| Deep dive | Every method body, all edge cases, configuration values, env dependencies, rate limits, retry logic, logging, all event listeners/subscribers, database indexes used, cache TTLs |

### Step 4: Explain

Read `references/explanation-patterns.md` to pick the right structure for the explanation type.

**General rules for all explanations:**

1. **Start with the one-sentence summary** — what does this thing do, in plain language?

2. **Use real names from the code** — `UserController::register()`, not "the registration
   endpoint". `users.email`, not "the email field". Real names let the user find things.

3. **Show the call chain visually** — even in text, show the flow:
   ```
   Request → AuthMiddleware → UserController::register()
           → RegisterRequest (validation)
           → UserService::createUser()
           → User::create() (Eloquent)
           → UserRegistered event → SendWelcomeEmail listener
           → Response 201 with UserResource
   ```

4. **Explain why, not just what** — "This uses a separate service layer instead of putting
   logic in the controller because..." / "The event is dispatched here rather than in the
   model because..."

5. **Flag important details** — things the user should know:
    - ⚠️ "This runs inside a database transaction — if the email dispatch fails, the user
      is still created because the event is dispatched after commit"
    - 🔒 "This endpoint requires the `admin` guard — regular users can't access it"
    - ⏱️ "This job is dispatched to the `notifications` queue with a 5-minute delay"
    - 💾 "Results are cached for 1 hour in Redis with key `user:{id}:profile`"

6. **Reference files with paths** — always include file paths so the user can jump to the code:
   "In `app/Services/UserService.php` (line ~45), the `createUser` method..."

7. **Name the patterns** — if the code uses recognizable patterns, name them:
   "This follows the **Repository pattern** — `UserRepository` abstracts database access..."
   "This is a **Pipeline** — the request passes through a chain of handlers..."

8. **Match the depth level:**
    - **Overview**: 3-5 paragraphs max. Major components and how they connect. No method-level
      detail. Think "explaining to a new team member on their first day."
    - **Standard**: follow the call chain step by step. Explain each layer, key decisions,
      error handling. Think "PR review explanation for a mid-level developer."
    - **Deep dive**: every method, every condition, every side effect. Config values, env
      variables, cache TTLs, queue names, retry policies. Think "writing internal documentation
      for the on-call engineer who'll debug this at 3am."

9. **End with a summary** — for Standard and Deep dive, end with:
    - Key files involved (with paths)
    - Important configuration that affects behavior
    - Known edge cases or potential issues you spotted
    - Questions you couldn't answer from the code alone

### Step 5: Offer follow-up

After explaining, offer natural next steps:

> Want me to:
> - Dive deeper into any specific part?
> - Explain how a related system connects to this one?
> - Save this explanation to docs? (as a markdown file for the team)
> - Generate a diagram of this flow? (if a diagram skill is available)

Only offer the diagram option if the user previously expressed interest or if the flow is
complex enough that a visual would genuinely help. Don't push it.

### Step 6: Save to docs (if requested)

When the user asks to save (either via the follow-up offer, or by saying "save this", "write
this to docs", "document this"), save the explanation as a markdown file.

**File path:** `{explanations_dir}/{type}/{name}.md`

- `{explanations_dir}` from config (default: `./docs/explanations`)
- `{type}` — the explanation type: `flow`, `component`, `connection`, `data-lifecycle`,
  `integration`, `design-decision`, `code`
- `{name}` — kebab-case, descriptive (e.g. `user-registration`, `stripe-integration`)

**File format:**

```markdown
# {Title}

> {One-line summary}

*Depth: {overview/standard/deep-dive} | Generated: {YYYY-MM-DD} | Code state: current*

{The full explanation content — same as what was shown in chat, but lightly cleaned up:
remove conversational elements like "Let me trace through this...", keep the substance}

## Key Files

- `path/to/file.php` — {role}

## Open Questions

- {Anything that couldn't be determined from code alone}
```

**When saving, adjust the tone slightly:**
- Chat explanation: conversational, "you" and "your", can reference the conversation
- Saved file: neutral, third-person, stands alone without conversation context
- Don't rewrite the whole thing — just clean up the framing so it reads as documentation

**Create directories as needed** (`mkdir -p`).

**Update the index** at `{explanations_dir}/index.md`:

```markdown
# Code Explanations

> Documentation generated from codebase analysis. Last updated: {YYYY-MM-DD}

### Flows
- [{Title}](./flow/{name}.md) — {short description}

### Components
- [{Title}](./component/{name}.md) — {short description}

### Integrations
- [{Title}](./integration/{name}.md) — {short description}
```

**Index rules:**
- Only include section headers for types that have explanations
- Keep entries sorted alphabetically
- Preserve existing entries
- Add `(outdated)` marker if the explanation references code that has since changed

## Handling Edge Cases

**Code is too complex for the requested depth:**
If the user asked for an overview but the system is deeply interconnected, give the overview
and flag the complexity: "This is a simplified view — the actual flow has several additional
branches around [X]. Want me to go deeper on any part?"

**Code is unclear or seems wrong:**
Don't silently skip confusing parts. Flag them:
"I notice `OrderService::process()` calls `PaymentService::charge()` inside a try/catch but
swallows the exception silently (line 87). This might be intentional for idempotency, or it
might be a bug. Worth checking."

**Multiple possible flows:**
If the same entry point leads to different flows based on conditions, explain the branching:
"This endpoint behaves differently depending on the user's role:
- For `admin`: [flow A]
- For `user`: [flow B]
- For `guest`: returns 401"

**Can't find the entry point:**
Ask the user: "I can see `PaymentService` but I'm not sure how it's triggered. Is it called
from a controller, a console command, or a queue worker? Point me to the entry point and I'll
trace from there."

**Dead code:**
If you find code that appears unreachable, mention it: "Note: `LegacyPaymentHandler` exists
in the codebase but I can't find any reference to it. It may be dead code."

## Important Principles

1. **Follow the chain, don't summarize file headers** — the value is in connecting pieces
   across files, not describing what each file says it does in its docblock.

2. **Real names always** — use actual class names, method names, table columns, route paths.
   The user needs to find things in the codebase, not in an abstract description.

3. **Explain the why** — anyone can read what code does. Explaining why it's structured
   this way is the real value.

4. **Surface hidden behavior** — middleware, observers, event listeners, cron jobs. These
   are the things that bite developers who only read the obvious call chain.

5. **Be honest about unknowns** — if you can't determine something from the code, say so.
   "I can't tell from the code alone whether this cache key is invalidated elsewhere —
   you'd need to search for `Cache::forget('user:{id}')` to verify."

6. **Inline first, save on request** — always explain in chat first. Only save to a file when
   the user asks. When saving, clean up conversational tone but don't rewrite — the content
   should be the same explanation, just framed as documentation.

7. **One question max** — ask at most one clarifying question before starting. The user
   came to understand something — start explaining and let them redirect.