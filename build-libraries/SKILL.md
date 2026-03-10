---
name: build-libraries
description: >
  Maintenance tool for the Charter framework. Creates or updates libraries.md —
  the registry mapping Charter capabilities to recommended libraries per stack family.
  Run this when the library landscape shifts significantly for a capability, when
  adding a new stack family, or periodically (semi-annually is reasonable) to verify
  recommendations are still current. Run build-stacks first if stacks.md is also
  being updated.
---

You are maintaining the Charter framework's library registry. Charter is an LLM-native
application framework where capabilities are the unit of composition. The library
registry tells charter-init which library to install and configure for each capability
on each stack — replacing "generate from scratch" with "configure what already exists."

The library registry is organized by capability and points at capabilities without
modifying the capability files themselves. Capability files remain isolated; this
file is the bridge between capability semantics and concrete library choices.

## Step 1 — Read current project state

Read the following files before doing any research:

1. List all files in `capabilities/` and read each one. You need to understand:
   - What each capability does (to evaluate library fitness)
   - What config options it exposes (to write accurate `when:` conditions)
   - What infrastructure dependencies it declares

2. Read `stacks.md` if it exists. The stack names and runtime identifiers used in
   stacks.md should be used consistently in libraries.md.

3. If `libraries.md` exists at the project root, read it. You are updating an
   existing registry. Note which capabilities already have entries and which
   recommendations may need verification. If `libraries.md` does not exist,
   you are creating it from scratch.

## Step 2 — Determine scope

**Stack families to cover.** Libraries.md is organized by capability, with library
recommendations per stack family. Cover these stack families:

- `node-typescript` — Node.js with TypeScript (Express, Fastify, or framework-agnostic)
- `nextjs` — Next.js specifically (App Router unless otherwise noted)
- `remix` — Remix specifically
- `python-django` — Python with Django and Django REST Framework
- `python-fastapi` — Python with FastAPI

Do not try to cover every possible stack. These five cover the likely Charter user base.
If stacks.md lists a stack with a distinct runtime not covered above, add a
corresponding stack family entry.

**Capabilities that warrant library entries.** Not every capability has a meaningful
library ecosystem. Focus research on capabilities where a library genuinely replaces
from-scratch generation. As a guide:

**High-value library coverage** (these have well-established libraries — prioritize):
auth, billing, payment, mailer, background-jobs, caching, search, file-management,
feature-flags, permissions, api-access, webhooks, observability, migrations

**Moderate library coverage** (libraries exist but are less definitive — include if clear):
multi-tenancy, versioning, import-export, reporting, real-time-transport

**Primarily generated from spec** (no standard library — skip or note as "generate from spec"):
comments, presence, activity-feed, inbox, announcements, onboarding, feedback,
waitlist, archiving, soft-delete, sharing, impersonation, referral, messaging

For the "generate from spec" capabilities, add a one-line entry noting there is no
standard library and the capability is generated from the Charter spec. This is
informative — its absence would be confusing.

## Step 3 — Research library recommendations

For each capability in the high-value and moderate categories, research the library
landscape for each stack family. For each candidate library, evaluate:

**Inclusion criteria — a library belongs in the registry if:**
- Actively maintained (commit activity in the last 6 months, or clearly in maintenance mode with known reasons)
- Production-proven (meaningful real-world usage, not just a personal project)
- Covers the Charter capability's core requirements without requiring massive custom code around it
- Fits Charter's architectural model (the capability spec's generation instructions
  should be reducible to "configure this library" rather than "reimplement around this library")

**Exclusion criteria — do not include:**
- Libraries that are abandoned or have known critical security issues
- Libraries that require so much configuration they offer little advantage over generating from scratch
- Hosted SaaS services where the primary value is the service, not the library
  (exception: include as `hosted-alternative` with a note when the hosted option
  is genuinely the common choice — e.g., Clerk for auth, Stripe for payments)

**For each recommended library, capture:**
- Current version / release date (confirms it's active)
- What the library covers vs. what Charter still needs to generate on top of it
- Any config options from the capability spec that affect which library to choose
  (this becomes the `when:` field)

## Step 4 — Write libraries.md

Produce `libraries.md` at the project root in the following format.

```markdown
# Charter Libraries Registry

Maps Charter capabilities to recommended libraries per stack family. Used by
charter-init to select what to install rather than generate from scratch, and
to inform generation instructions with concrete library APIs.

**How charter-init uses this file:**
1. For each selected capability, look up this file by capability name
2. Match the stack family to the project's declared stack
3. Install `recommended` (or `hosted-alternative` if appropriate)
4. Generation instructions configure the library — they do not reimplement it

**How to read `when:` fields:**
`when:` conditions reference config keys from the capability's project-brief.yaml
configuration. When a condition matches the project's config, use that library.

*Last updated: {date}*

---

## {capability-name}

> {one sentence: what this capability does and why library coverage matters here}

### node-typescript

| Role | Library | Notes |
|---|---|---|
| recommended | `{library}` | {what it covers} |
| alternative | `{library}` | when: {condition from capability config} |
| hosted-alternative | `{service}` | when: {condition — e.g., team prefers managed auth} |

**What the library covers:** {list the capability requirements the library fulfills}
**What Charter generates on top:** {list what still needs to be generated — UI, wiring, domain logic}

### python-django

| Role | Library | Notes |
|---|---|---|
| recommended | `{library}` | {what it covers} |

**What the library covers:** ...
**What Charter generates on top:** ...

### python-fastapi

(repeat pattern)

### nextjs

(repeat pattern — may defer to node-typescript if no Next.js-specific option)

### remix

(repeat pattern — may defer to node-typescript if no Remix-specific option)

---

## {next capability}

...

---

## Capabilities without standard libraries

The following capabilities are generated from their Charter specs. No standard
library covers their requirements — the generation instructions produce the
implementation directly.

| Capability | Reason |
|---|---|
| comments | No standard library; threading, reactions, moderation are app-specific |
| presence | Thin layer over real-time-transport; not enough logic to warrant a library |
| ... | ... |

```

## Step 5 — Quality checks before writing

- Every capability name used as a section header matches a filename in `capabilities/`
- Every `when:` condition references a real config key from that capability's spec
- The "What Charter generates on top" section is honest — if a library covers 90%
  and Charter generates 10%, say so; if it's 50/50, say so
- No library is listed that fails the inclusion criteria
- Stack family names are consistent with those used in `stacks.md`

If updating an existing file: preserve existing entries that you did not re-research,
but note their original date is unchanged. Add a comment if a previously recommended
library shows signs of reduced maintenance activity — don't silently remove it, flag it.

After writing the file, summarize: capabilities added, recommendations changed,
libraries flagged for reduced maintenance, and any capabilities moved between
"library-covered" and "generate from spec" categories.
