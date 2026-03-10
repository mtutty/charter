---
name: charter-iterate
description: Use this skill for all LLM-assisted development on an existing Charter project. Triggers include any coding task, feature request, bug fix, refactor, or change request made within a project that has a .charter/ directory. This skill reads the charter before acting, makes code changes, assesses whether those changes are semantically significant, and updates the charter documents automatically. The charter is always the primary source of truth.
---

You are operating within a Charter project. Before doing anything else, read the charter. It tells you what this application is, what it's made of, and why decisions were made. Code is downstream of the charter — not the other way around.

## Mandatory First Step

**Always read these files before making any change:**

1. `.charter/project-brief.yaml` — what the app is, what capabilities it has
2. `.charter/integrations.md` — how capabilities connect to each other
3. `.charter/capabilities/{relevant}.md` — the specific capability you're about to touch

If these files don't exist, stop and tell the developer this project hasn't been initialized with Charter. Offer to run charter-init.

If these files exist but appear inconsistent with the code you can see, flag the discrepancy before proceeding. Do not silently pick one over the other. Ask which is correct.

## Operating Modes

You operate in one of three modes depending on what's being requested. Identify the mode before acting.

### Mode 1: Standard Iteration

A bounded change to existing code — a bug fix, a UI update, adding a field, changing behavior within a capability. The capability structure itself isn't changing.

**Process:**
1. Read the charter (mandatory)
2. Make the change
3. Assess semantic significance (see below)
4. Update charter documents if significant
5. Report what changed and what was updated

### Mode 2: Capability Addition

A new capability is being added to an existing project. This is a meaningful structural change that requires more care.

**Process:**
1. Read the full charter (mandatory)
2. Shift into a brief conversational mode — ask the same focused questions charter-init would ask for this specific capability. Not a full interview, just the decisions that can't be defaulted.
3. Propose the capability configuration and confirm with the developer
4. Update `.charter/project-brief.yaml` with the new capability
5. Create `.charter/capabilities/{name}.md`
6. Update `.charter/integrations.md` if this capability connects to existing ones
7. Append to `.charter/decisions.md` explaining why this capability was added now
8. Generate the capability scaffold and implementation
9. Report the full set of changes

### Mode 3: Refactor or Restructure

A change that touches multiple capabilities or meaningfully reorganizes how the application works. Treat this with extra care.

**Process:**
1. Read the full charter (mandatory)
2. Before making any changes, describe what you're about to do and what charter documents will need updating. Get confirmation.
3. Make the changes
4. Update all affected charter documents
5. Do a full consistency pass (see below)
6. Report everything that changed

## Assessing Semantic Significance

After every change, assess whether it's semantically significant. State your assessment explicitly before deciding whether to update the charter.

**A change IS semantically significant if it:**
- Adds, removes, or reconfigures a capability
- Changes how two capabilities connect or hand off to each other
- Adds or removes an entity or meaningfully changes relationships between entities
- Changes a decision that was explicitly made during project init or recorded in decisions.md
- Adds behavior via a `.ext` file that represents a genuine product decision
- Changes the auth model, permission structure, billing behavior, or data ownership
- Introduces a new integration with an external service

**A change IS NOT semantically significant if it:**
- Fixes a bug without changing intended behavior
- Refactors implementation without changing what the code does
- Changes styling, copy, or presentation details
- Updates a configuration value (env var) without changing capability structure
- Adds a trivial utility function or helper
- Changes a UI label or error message

**When uncertain:** err toward updating. A slightly over-documented change is better than a missed semantic update. If you're genuinely unsure, say so and ask.

## Updating Charter Documents

When a change is semantically significant, update the relevant documents with precision. Update what changed. Don't rewrite what didn't.

### Updating `.charter/project-brief.yaml`
Update capability configuration blocks when capability behavior changes. Update `domain-context` only when your understanding of the domain itself has been corrected or expanded. Never rewrite the whole file — surgical updates only.

### Updating `.charter/capabilities/{name}.md`
Update the relevant section:
- **Configuration decisions** — when a decision changes or a new one is made
- **Integration points** — when connections to other capabilities change
- **Post-generation customizations** — when `.ext` files are modified with meaningful logic. Describe what the customization does and why, not how it's implemented.
- **Known limitations and future considerations** — when you encounter something the capability can't currently express, or when a developer mentions future plans

### Updating `.charter/integrations.md`
Update when any integration point changes — a new trigger, a changed payload, a removed connection. Be specific about what changed and why.

### Updating `.charter/data-model.md`
Update when entities are added, removed, or when relationships between them change. Keep descriptions semantic — what the entity *means*, not what columns it has.

### Appending to `.charter/decisions.md`
Append a new entry when:
- A significant architectural decision is made during this iteration
- A capability is added or substantially reconfigured
- An alternative approach was considered and rejected
- A constraint was discovered that will affect future development

**Decision entry format:**
```markdown
## {Short descriptive title} — {date}
**Decision:** {what was decided, one or two sentences}
**Context:** {why this decision was needed — what problem or situation prompted it}
**Rationale:** {why this choice over alternatives}
**Alternatives considered:** {what else was considered, briefly}
**Consequences:** {what this makes easier or harder going forward}
```

Write decision entries as a future developer would want to read them — six months from now, someone will read this and need to understand not just what was decided but why, so they don't inadvertently reverse a decision whose reasoning has been forgotten.

**Never edit existing entries in decisions.md. Append only.**

## Writing Good Charter Prose

The charter documents should explain intent, not describe implementation.

**BAD** (describes implementation):
> Auth uses jsonwebtoken 9.0 with HS256 algorithm. Tokens are stored in httpOnly cookies with the secure flag set to true in production.

**GOOD** (explains intent):
> Auth uses stateless JWT sessions. Tokens are kept out of JavaScript scope intentionally — a security decision appropriate for this app's enterprise market where XSS risk is taken seriously.

The implementation detail belongs in the code. The semantic intent and the reasoning belong in the charter. An LLM reading the charter six months from now should understand *what* and *why*, not *how*.

## The Consistency Pass

After any Mode 2 or Mode 3 change, and periodically during Mode 1 work, do a consistency check:

- Do the charter documents accurately describe what the code actually does?
- Are all integration points in integrations.md reflected in the code?
- Are there `.ext` file customizations that aren't documented in the relevant capability doc?
- Are there entities in the code that aren't in the data model?
- Are there capabilities in the code that aren't in the project brief?

Flag any inconsistencies explicitly. Don't silently fix them — the developer needs to know the charter has drifted and confirm the correct state before you update.

## Handling Charter Drift

If you find that the charter and the code have diverged meaningfully — more than a small missed update — treat this as a significant event:

1. List every discrepancy you can find
2. For each discrepancy, state which you believe is correct (code or charter) and why
3. Ask the developer to confirm before making any updates
4. Once confirmed, update the charter to reflect the confirmed state
5. Append an entry to decisions.md noting that drift was found and reconciled

Charter drift is a signal that charter-iterate hasn't been used consistently. Note this without judgment and correct it.

## Adding a New Capability Mid-Project

When a developer asks to add a capability that isn't in the current charter:

First, ask the focused configuration questions for that capability — the same questions charter-init would ask. Don't assume defaults are correct for an established app with existing context.

Then confirm the integration points explicitly:
> "Before I add this capability, I want to confirm how it connects to what you already have. It looks like {capability} should trigger {new capability} when {event}. Is that right, and are there other connections I should know about?"

Then proceed with Mode 2 process above.

## Reporting

After every interaction, provide a brief report:

**For non-significant changes:**
> Made the requested change. No charter updates needed — this was an implementation detail within the existing {capability} capability.

**For significant changes:**
> Made the requested change. Updated the charter:
> - `capabilities/auth.md` — added post-generation customization note for the new token expiry logic
> - `decisions.md` — recorded the decision to use sliding sessions for this user segment
>
> The change affects the auth → onboarding integration point, which is now reflected in integrations.md.

**For discrepancies found:**
> Before making this change, I noticed the charter and code disagree on {topic}. The charter says {X}, but the code does {Y}. Which is correct?

## Quality Criteria

A good charter-iterate session:
- Always reads the charter before acting
- Never makes a semantically significant change without updating the charter
- Writes charter updates that explain intent, not implementation
- Flags discrepancies rather than silently resolving them
- Keeps decision entries honest about tradeoffs — not just what was decided but what was given up
- Leaves the charter more accurate and useful than it found it
