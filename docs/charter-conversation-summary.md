# Charter Framework — Conversation Summary
*For continuing this work in separate contexts*

---

## What We Set Out to Do

Started with a question about what to call the new unit of reuse that LLM-assisted development makes possible. Not a component (too small), not a feature (too vague) — something that encompasses an entire functional flow: user registration + onboarding, subscription management + billing UX, a full Swagger API, an entire Express service with CRUD for a complex data model.

Landed on **capability** (colloquially: *cap*) as the right word. A capability is semantically complete, horizontally reusable across applications, carries intent not just code, and is LLM-addressable — you can describe one in natural language and an LLM can both recognize and instantiate it.

---

## The Central Insight

**The spec has regained primacy over the code.**

The Agile movement spent 20 years arguing that requirements documents were liabilities — slow, wrong, rotting. "Working software over comprehensive documentation." The entire culture moved away from specs.

LLM-assisted development inverts this completely. Code is now so cheap to generate that it's effectively ephemeral. What's durable, transferable, and valuable is the precise description of *what the thing is supposed to do and be*. The spec — the *charter* — is the artifact you maintain. Code is regenerated from it.

This works now for a reason that didn't exist before: the translation layer from spec to implementation is no longer lossy. LLMs are tireless, literal, and consistent. The spec doesn't just describe what you hope gets built — it's what *will* get built.

**The Agile folks were right that the old process was broken. They fixed the wrong variable.**

---

## The Framework Reframe

The original Lexicon concept was bottom-up: define a vocabulary for individual UI elements, field types, mutation steps. Teach the LLM to express small things precisely.

The reframe: that's the wrong level. Code-trained LLMs already know what a sortable table looks like, what user registration needs, how Stripe webhooks work. You don't need to teach them that.

What LLMs *can't* know:
- Which capabilities your app has
- How they connect to each other
- What's specific to your business (tier names, permission model, onboarding steps)
- Why decisions were made

The framework's job is to capture exactly those things — and only those things — leaving everything else to the LLM's training.

---

## The Framework: Charter

**Name:** Charter (framework name). The product it's part of is GForge — "Forge" was set aside to avoid collision.

**Core metaphor:** A charter defines what an organization is empowered to do. The project charter defines what an application is, what capabilities it has, and why decisions were made. Code is downstream of it.

### The Charter Document Set

Every Charter project contains `.charter/`:

```
.charter/
├── project-brief.yaml      # master capability config + domain context
├── capabilities/
│   └── {name}.md           # one file per capability, intent not implementation
├── integrations.md         # how capabilities connect and hand off
├── data-model.md           # semantic entity descriptions, not schema dumps
└── decisions.md            # append-only architectural decision log
```

**project-brief.yaml structure:**
```yaml
charter: 1.0
app:
  name: 
  description:              # 2-4 sentences, domain + users + value
  market:                   # consumer | smb | enterprise | internal | developer
  business-model:           # free | subscription | usage-based | enterprise | internal
  domain-context: >         # semantic context LLMs need — NOT a feature list
capabilities:
  {name}:
    {yaml config}
    context: >              # why configured this way for this specific app
```

**The `context:` field is the key innovation.** It's the prose alongside the structured YAML — enough semantic intent that the LLM generating the capability makes domain-specific choices rather than generic ones.

### Capability Blueprint Structure

Each capability has a layered YAML config with progressive complexity:

```yaml
capability: auth            # bare minimum — generates working defaults

providers:                  # layer 1: identity
  email-password: true
  google: false
  saml: false

session:                    # layer 2: session model
  mechanism: jwt
  duration: 30d
  sliding: true

security:                   # layer 3: security posture
  mfa:
    enforcement: optional
  password-policy: standard

account:                    # layer 4: account model
  unit: user                # user | org
  registration: open        # open | invite-only | closed

on:                         # layer 5: integration points
  registration-complete: onboarding
  login: null
```

**Design principle:** A bare `capability: auth` generates something working immediately. Complexity is opt-in. Capabilities are configurable post-generation without requiring exhaustive upfront specification.

### What the LLM Fills In (Without Being Asked)

Everything that's implementation rather than decision:
- Form fields, error states, validation logic
- JWT/cookie implementation details
- Password hashing, rate limiting, CSRF protection
- Email templates
- Route middleware, protected route wrappers

The spec captures decisions. The LLM supplies implementation. This is the clean division.

---

## The Two-Skill Architecture

Two skills, not one. Clean separation of concerns.

### `charter-init`

**When:** Starting a new project. Runs once (maybe twice for early pivots).

**Mode:** Conversational interview. High human interaction. Opinionated capability proposals.

**Process:**
1. App identity interview (what, who, business model, stage)
2. Capability proposal — LLM proposes, human confirms
3. Capability configuration — only genuine business decisions, 2-3 questions per cap max
4. Integration points — one question about non-obvious handoffs

**Output:** Complete `.charter/` document set + scaffold (directory structure, `.ext` stubs, `.env.example`, README). No implementation code — that's a separate step.

**Key principle:** Ask fewer than 15 questions total. Be opinionated. Don't present menus.

### `charter-iterate`

**When:** All subsequent development. Every coding session on an established project.

**Mode:** Action-oriented. Reads charter first, always. Updates charter as primary priority after semantically significant changes.

**Three operating modes:**
- **Standard iteration** — bounded change within existing capability
- **Capability addition** — new capability mid-project, triggers brief interview
- **Refactor/restructure** — multi-capability changes, requires explicit confirmation before acting

**Semantic significance criteria:**

A change IS significant if it:
- Adds/removes/reconfigures a capability
- Changes how capabilities connect
- Adds/removes entities or relationships
- Adds `.ext` behavior representing a genuine product decision
- Changes auth model, permissions, billing, or data ownership

A change IS NOT significant if it:
- Fixes a bug without changing intended behavior
- Refactors without changing what code does
- Changes styling, copy, or presentation
- Updates a config value without changing capability structure

**When uncertain: err toward updating.**

**The append-only decisions.md rule:** Never edit existing entries. New decisions always append. Six months from now someone needs to understand not just what was decided but why.

**Charter drift handling:** If documents and code disagree, documents are right until a human explicitly says otherwise. Flag discrepancies, don't silently resolve them.

---

## Key Design Principles

**1. Capabilities are configurable, not interrogated**
Don't require 20 questions upfront. Generate working defaults. Let configuration happen post-generation. The spec captures decisions that matter; everything else can be changed later.

**2. Documents explain intent, not implementation**
BAD: "Auth uses jsonwebtoken 9.0 with HS256 algorithm"
GOOD: "Auth uses stateless JWT sessions — tokens kept out of JavaScript scope intentionally given the enterprise security requirements"

**3. The charter is always the primary source of truth**
Code is downstream of the charter. If they disagree, the charter is right. This is not a documentation convention — it's the load-bearing principle of the whole system.

**4. Automated maintenance, not human discipline**
The reason documentation always rots: updating it is a separate step with no immediate consequence for skipping. Charter-iterate makes document updates an atomic part of making code changes. The developer doesn't maintain the docs. The skill does.

**5. Prompt-first before agent workflows**
Resist the urge to build explicit agent graphs with defined edges and nodes until a well-crafted prompt proves insufficient. The iterate skill is a prompt, not a workflow engine. Test it on real projects for two weeks before adding structural complexity.

---

## The Capability Landscape (Brainstormed)

**Core (every serious app):**
Auth, Onboarding, Billing, User Management, Notifications, Settings

**Second tier:**
Permissions, Audit Log, Search, File/Media Management, Reporting/Analytics, API Access, Multi-tenancy, Impersonation

**Collaboration:**
Comments/Annotations, Presence, Activity Feed, Sharing

**Data lifecycle:**
Soft Delete + Trash, Versioning/History, Import/Export, Archiving

**Communication:**
Transactional Email, In-app Messaging, Announcements

**Growth/Engagement:**
Referral, Feature Flags, Feedback Collection, Waitlist

**Structural observations:**
- Natural dependency graph exists (Permissions depends on User Management, Audit Log depends on Auth)
- Some caps are universal singletons (one Auth per app), others are applied per resource (Versioning, Comments can attach to any entity)
- The framework needs to capture both patterns

---

## Auth Capability — Worked Example

The most detailed capability spec developed in this conversation. Key points:

- Bare `capability: auth` generates working email/password auth with sensible defaults
- Layered configuration: providers → session → security → account → integration points
- Config surface (`.env`) is significantly larger than spec surface — credentials, durations, limits. Config is for values that change per environment. Spec is for decisions that have architectural significance.
- Several config values are runtime overrides of spec decisions (`AUTH_REGISTRATION_MODE`, `AUTH_MFA_ENFORCEMENT`) — lets you change behavior without regeneration

Full spec and config surface were developed in the conversation and should be treated as a reference implementation for how other capability specs should be structured.

---

## Open Threads (Good Starting Points for New Contexts)

**1. Capability spec development**
Auth is the most complete. Needs the same treatment for: Onboarding, Billing, User Management, Permissions, Notifications, Multi-tenancy. Each should have the same layered YAML config + generated `.env` surface.

**2. Integration point design**
The `on:` section of a capability spec is where capabilities wire together. The `integrations.md` document captures these seams. How exactly does one capability hand off to another? What's the payload? How does the generated code implement a trigger?

**3. The capability registry**
The "publishable artifact" question — what does a shareable capability blueprint look like? How does versioning work? What's the registry model? This is the npm analog for capabilities.

**4. charter-generate**
A third tool implied by charter-init's closing summary but not yet designed. Takes the project brief and generates actual implementation code (not just scaffold). How does it work? Does it generate capability by capability? In what order?

**5. The prompt architecture**
How much context does charter-iterate actually need in its context window to operate reliably? What's the minimum viable charter read at the start of each session? Does it read everything or just the relevant capability?

**6. Testing the skills**
Both skills need to be run against real projects to discover where they break down. The semantic significance criteria in charter-iterate are the most judgment-dependent and will need tuning from real usage.

**7. Stack decisions**
The framework implies a tech stack but it hasn't been explicitly locked. React, Express, PostgreSQL was the assumption from the earlier Lexicon work. Does Charter inherit those decisions or is it stack-agnostic?

---

## Files Produced in This Conversation

- `charter-init/SKILL.md` — the charter-init skill prompt
- `charter-iterate/SKILL.md` — the charter-iterate skill prompt
- This summary document

---

## Context for New Sessions

When continuing this work in a new context, provide:
1. This summary document
2. The relevant skill file(s) if working on skill refinement
3. The statement: *"We're building Charter, an LLM-native application framework where capabilities are the unit of composition and the project charter is the primary source of truth. Code is downstream of the charter, not the other way around."*

That's enough to re-establish the conceptual foundation without replaying the full conversation.
