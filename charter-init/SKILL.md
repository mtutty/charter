---
name: charter-init
description: Use this skill when a developer wants to start a new application project using the Charter framework. Triggers include: "start a new app", "create a new project", "I want to build an app", "let's scaffold a new application", or any request to initialize a Charter project. This skill conducts a conversational interview to produce a project charter — the durable artifact that defines what the application is, what capabilities it has, and the context an LLM needs to generate and iterate on it reliably.
---

You are conducting a Charter project initialization. Your job is to interview the developer, understand what they're building, and produce a complete project charter — the durable artifact that defines this application and guides all future LLM-assisted development.

The charter is the primary source of truth for this project. Code is downstream of it. Every future LLM interaction with this codebase should read the charter first.

## Mandatory First Step — Fetch Framework Reference Data

Before starting the interview, fetch the following files from the Charter framework repository. You will need them to propose stacks, select libraries, and generate capability scaffolding.

```
https://raw.githubusercontent.com/mtutty/charter/main/stacks.md
https://raw.githubusercontent.com/mtutty/charter/main/libraries.md
```

Then fetch the capability spec for each capability you end up selecting during the interview. Capability specs live at:

```
https://raw.githubusercontent.com/mtutty/charter/main/capabilities/{capability-name}.md
```

For example: `capabilities/auth.md`, `capabilities/billing.md`, `capabilities/mailer.md`.

Do not proceed with stack proposal, library selection, or code generation without this data. If a fetch fails, tell the developer which URL failed and ask them to check their network connection or whether the Charter repository is accessible.

## Your Responsibilities

1. Conduct a focused conversational interview
2. Propose an appropriate capability set
3. Configure each capability with sensible defaults
4. Produce the complete charter artifact and meta-document scaffold
5. Generate the initial capability scaffolding

## The Interview

### Phase 1 — App Identity (always ask these)

Ask these questions conversationally, not as a list. Adapt based on answers. You need:

- **What are you building?** One or two sentences. What does it do, who uses it.
- **Who is the primary user?** Consumer, SMB, enterprise, developers, internal team.
- **What's the business model?** Free, subscription, usage-based, enterprise contract, internal tool.
- **What stage?** New project, MVP, replacing something existing.

Do not ask about specific features or implementation details at this stage. You will propose the capability set and stack. They will determine the domain.

### Phase 1b — Stack and Infrastructure Posture

After app identity, establish the technical foundation. Be opinionated — propose a stack and only ask if the developer has a strong reason to deviate.

Default stack proposal for a web SaaS:
> "I'll set you up with Node.js + TypeScript, Express, React + Vite, PostgreSQL with Prisma, and pnpm. Does anything there conflict with your team's preferences or existing infrastructure?"

If they have preferences, record them. If they don't push back, proceed with defaults.

Then ask one infrastructure posture question:
> "Do you expect to run multiple backend instances (for availability or load), or are you starting with a single server?"

This answer determines:
- Whether `in-process` event delivery is safe (single instance only)
- Whether the caching provider needs to be Redis vs. in-memory
- Whether the real-time transport needs Redis pub/sub coordination

Do not ask about CI/CD provider, deployment target, or secret management at this stage — propose sensible defaults and let the developer override post-init.

### Phase 2 — Capability Proposal

Based on the app description, propose a capability set. Be opinionated. Don't list every possible capability and ask them to choose — propose what makes sense and explain why.

A typical proposal sounds like:

> "Based on what you've described, here's what I'd suggest as your foundation:
> Auth, because you need user accounts. Billing with subscription tiers, because you mentioned paid plans. Multi-tenancy, because teams need their own workspaces. Notifications, because you'll want to reach users when things happen.
> I'd leave Audit Log, API Access, and Impersonation out of the first version — those are enterprise concerns you can add when you need them.
> Does this feel right, or am I missing something central to how the product works?"

Wait for confirmation or correction before proceeding.

### Phase 3 — Capability Configuration

For each confirmed capability, work through the meaningful decisions. Not every option — only the ones where the default would genuinely be wrong for this app.

**What to ask vs. what to assume:**

ASSUME sensible defaults for implementation details. Do not ask:
- Which JWT library to use
- Password hashing algorithm
- Specific table names
- Email template styling

ASK only about genuine business decisions:
- Auth: which social providers, whether MFA is optional or required, open or invite-only registration
- Billing: tier names and what distinguishes them, trial policy, whether enterprise is self-serve or sales-assisted
- Multi-tenancy: what the "org" concept is called in their domain (team, workspace, company, account)
- Permissions: what roles exist and what they mean in this domain
- Onboarding: what steps genuinely need to happen before a user can get value

Keep this phase brief. Two or three questions per capability maximum. The rest is post-generation configuration.

### Phase 4 — Integration Points

Before closing the interview, ask one question:

> "Are there any places where these capabilities need to hand off to each other that aren't obvious? For example, does completing onboarding unlock billing, or does a permission change need to trigger a notification?"

Most developers won't think of everything here. That's fine. Integration points can be added later. The goal is capturing the ones they already know about.

## The Charter Artifact

Produce the following files:

### `.charter/project-brief.yaml`

```yaml
charter: 1.0
generated: {ISO timestamp}

app:
  name: {app name}
  description: >
    {2-4 sentence description capturing domain, users, and core value}
  market: {consumer | smb | enterprise | internal | developer}
  business-model: {free | subscription | usage-based | enterprise | internal}
  stage: {mvp | growth | established}
  domain-context: >
    {Key domain concepts, terminology, and workflows specific to this
    application that an LLM needs to make good implementation decisions.
    This is not a feature list — it's the semantic context that
    distinguishes this app from a generic app in the same category.}

stack:
  backend:
    runtime: node              # node | python | go | java
    framework: express         # express | fastapi | gin | spring
    language: typescript       # typescript | javascript | python | go
  frontend:
    framework: react           # react | vue | svelte | none
    meta-framework: vite       # vite | next | nuxt | none
    language: typescript
  api:
    style: rest                # rest | graphql | trpc
  database:
    primary: postgres          # postgres | mysql | sqlite | mongodb
    orm: prisma                # prisma | drizzle | typeorm | none
  package-manager: pnpm        # npm | yarn | pnpm | bun

infrastructure:
  scalability:
    model: stateless           # stateless | stateful-with-shared-store
    horizontal: false          # true = multiple instances expected; constrains event delivery and caching choices

  secrets:
    provider: env-files        # env-files | aws-secrets-manager | vault | doppler
    validation: strict         # strict (fail on missing required) | warn | none

  migrations:
    strategy: forward-only     # forward-only | up-down
    run: on-startup            # on-startup | manual
    seed: true

  caching:
    provider: redis            # redis | memcached | in-memory
    # note: in-memory is incompatible with horizontal: true

  background-jobs:
    transport: database        # in-process | database | redis | broker
    provider: pg-boss          # resolved from transport: pg-boss | bullmq | rabbitmq | sqs | nats
    # note: in-process is incompatible with horizontal: true and with separate worker containers
    coordination:
      mutex: true              # at-most-once delivery for sensitive operations
      deduplication-window: 60s

  real-time-transport:
    enabled: false             # set to true if Presence or Messaging capabilities are included
    protocol: websocket        # websocket | sse
    # multi-instance pub/sub via Redis is auto-generated when horizontal: true

  observability:
    logging:
      format: json
      level: info              # debug | info | warn | error — overridable via LOG_LEVEL env var
    error-tracking: none       # none | sentry | rollbar
    apm: none                  # none | opentelemetry | datadog | newrelic
    health-checks: true

  containerization:
    enabled: true
    orchestration: compose     # compose | kubernetes

  ci-cd:
    provider: github-actions   # github-actions | gitlab-ci | circleci | none
    environments:
      - staging
      - production
    deploy-target: fly         # fly | railway | aws-ecs | kubernetes | ssh

capabilities:
  {capability-name}:
    {capability YAML config}
    context: >
      {Why this capability is configured this way for this specific app.
      What's non-obvious. What decisions were made and why.}
```

### `.charter/capabilities/{name}.md`

One file per capability. Write these as a future developer (or LLM) would want to read them:

```markdown
# {Capability Name}

## What it does in this app
{2-3 sentences. Not a generic description of the capability — 
a description of what it does *here*, for *these users*.}

## Configuration decisions
{Each non-default configuration choice, and why it was made.
Future developers need to understand the reasoning, not just the value.}

## Integration points
{How this capability connects to others. What triggers it,
what it triggers, what data it shares.}

## Post-generation customizations
{Empty at init. Filled in by charter-iterate as .ext files are modified.}

## Known limitations and future considerations
{Empty at init. Filled in as the project evolves.}
```

### `.charter/integrations.md`

```markdown
# Capability Integration Map

{For each integration point between capabilities:}

## {Capability A} → {Capability B}
**Trigger:** {what event causes the handoff}
**Payload:** {what data is passed}
**Why:** {the business reason this connection exists}
```

### `.charter/data-model.md`

```markdown
# Application Data Model

## Entities

### {EntityName}
**Owned by:** {capability that owns this entity}
**Purpose:** {what this entity represents in the domain}
**Key relationships:** {how it relates to other entities, in plain language}
```

### `.charter/decisions.md`

```markdown
# Architectural Decisions

This is an append-only log. Never edit existing entries.

---

## {Decision title} — {date}
**Decision:** {what was decided}
**Context:** {why this decision was needed}
**Rationale:** {why this option over alternatives}
**Alternatives considered:** {what else was considered and why rejected}
**Consequences:** {what this decision makes easier or harder going forward}
```

Seed this file with the significant decisions made during the interview — capability selection rationale, any non-default configurations that represent genuine choices.

## Generating the Initial Scaffold

After producing the charter files, generate the project scaffold using the framework reference data you fetched at the start of this session.

### Step 1 — Select and run the base stack scaffold

Using `stacks.md`, identify the best-fit base stack for this project's declared `stack` configuration. Check its `capabilities-covered` and `capabilities-partial` lists to understand what it provides out of the box.

Run the stack's scaffold command (e.g., `npx create-t3-app@latest`, `npx create-next-app@latest`, `django-admin startproject`, etc.) to generate the base project. Do not write the base project files from scratch — use the scaffold tool.

If no stack in `stacks.md` is a good fit, use the closest match and note the gap.

### Step 2 — Install capability libraries

For each selected capability, consult `libraries.md`. Match the capability and the project's stack family to find the recommended library. Install it. Do not generate library code from scratch when a library is available.

If a capability has no library entry (listed under "Capabilities without standard libraries"), it will be generated from spec in the next step.

### Step 3 — Generate capability code from specs

For each selected capability, use the capability spec you fetched (e.g., `capabilities/auth.md`) as the generation instructions. The spec tells you exactly what to generate, what the library covers, and what Charter needs to add on top.

Generate only what the spec says to generate. Do not add implementation beyond what the spec describes.

### Step 4 — Generate supporting files

1. Stub `.ext` files for each capability with typed interfaces
2. A `.env.example` containing all configuration values for all selected capabilities, grouped by capability, with comments
3. A `README.md` that points developers to the charter as the primary source of truth

## Closing the Init Session

End with a summary:

> "Your charter is ready. Here's what was created:
> {list of files}
>
> The project brief captures {N} capabilities: {list}.
> The key integration points are: {list}.
>
> To generate the initial implementation, run charter-generate.
> For all future development, charter-iterate will keep the charter
> in sync with your code automatically.
>
> The one thing to remember: the charter is the source of truth.
> If code and charter disagree, the charter is right until you
> explicitly decide otherwise."

## Quality Criteria

A good charter init session:
- Takes 10-15 minutes of conversation
- Asks fewer than 15 questions total
- Produces a project-brief.yaml that a developer would recognize as accurately describing their app
- Produces capability docs that would orient a new developer without them reading any code
- Captures the *why* behind decisions, not just the *what*
- Leaves implementation details to the LLM and configuration to the config files
