---
name: build-stacks
description: >
  Maintenance tool for the Charter framework. Creates or updates stacks.md —
  the registry of known base stacks that charter-init uses to scaffold new projects.
  Run this when adding a new stack, when a stack's capabilities change significantly,
  or on a periodic basis (quarterly is reasonable) to verify coverage lists are current.
---

You are maintaining the Charter framework's stacks registry. Charter is an LLM-native
application framework where capabilities are the unit of composition and the project
charter is the primary source of truth. The stacks registry tells charter-init what
pre-built base stacks exist, what Charter capabilities each covers out of the box,
and where to fetch current information about each stack at project init time.

## Step 1 — Read current project state

Read the following files before doing any research:

1. List all files in `capabilities/` — this gives you the full set of Charter capabilities
   that stacks can be evaluated against. The capability names (filenames without .md)
   are the canonical identifiers used in the registry.

2. If `stacks.md` exists at the project root, read it. You are updating an existing
   registry. Note which stacks are already listed and which `fetch-for-current-state`
   URLs are defined — you will re-fetch those to verify coverage is still accurate.
   If `stacks.md` does not exist, you are creating it from scratch.

## Step 2 — Identify stacks to evaluate

Research and evaluate the following stacks. These are the primary base stacks
Charter should know about. For each one, fetch its current documentation to
determine what it includes today — do not rely on prior knowledge, which may be stale.

**Node.js / TypeScript:**
- Epic Stack — https://github.com/epicweb-dev/epic-stack
- create-t3-app — https://create.t3.gg / https://github.com/t3-oss/create-t3-app
- create-next-app (bare Next.js) — https://nextjs.org/docs/getting-started/installation

**Python:**
- Django + DRF starter — evaluate the conventional Django + djangorestframework setup
  (no single canonical repo — describe the conventional stack)
- FastAPI — https://fastapi.tiangolo.com (evaluate the conventional FastAPI + SQLAlchemy/SQLModel setup)

**PHP:**
- Laravel Breeze or Jetstream — https://laravel.com/docs/starter-kits

**Search for new notable stacks:** Run a web search for "full stack starter kit [current year]"
and "opinionated web framework starter [current year]". If any new stack has significant
community adoption (>2k GitHub stars, active maintenance, covers multiple Charter
capabilities), add it to the registry. Use your judgment — Charter's registry should
list stacks that a developer would seriously consider, not every starter template
that exists.

## Step 3 — Evaluate each stack

For each stack, determine:

**Capabilities covered** — which Charter capabilities does this stack provide working
implementations of, out of the box, without additional library installation?

Use this evaluation standard:
- **Covered**: the stack includes a working implementation that a developer would
  actually use in production. Not just a stub or an empty directory.
- **Partial**: the stack includes infrastructure (connection setup, library installed)
  but not a complete implementation (e.g., email transport configured but no
  template system, auth routes exist but social login not wired).
- Not listed: the stack does not address this capability at all.

Evaluate against every capability in the `capabilities/` directory.

**Stack fitness dimensions** (record these as notes):
- What type of application is this stack best suited for?
- What are its meaningful limitations?
- What does it opinionatedly exclude that Charter would need to add?
- Is it actively maintained? (Check last commit date, open issues, release cadence)

## Step 4 — Write stacks.md

Produce `stacks.md` at the project root in the following format. Write it as a
markdown document with a YAML block per stack. Preserve any existing stacks from
the current file even if you did not re-research them — mark them with a
`last-verified` date if you did verify them this run.

```markdown
# Charter Stacks Registry

Reference document for charter-init. Lists known base stacks, what Charter capabilities
each covers, and where to fetch current state at project init time.

**How charter-init uses this file:**
1. Propose an appropriate base stack based on the app description
2. Fetch the URL in `fetch-for-current-state` to verify current coverage
3. Cross-reference selected capabilities against `capabilities-covered` and
   `capabilities-partial` to identify what Charter needs to add
4. For gaps, consult `libraries.md` for recommended library per capability

*Last updated: {date}*

---

## {Stack Name}

\`\`\`yaml
name: {Stack Name}
url: {primary URL}
last-verified: {YYYY-MM-DD}
fetch-for-current-state: {URL of README, features doc, or changelog to fetch at init time}

runtime: {node | python | php | go | java}
framework: {primary framework}
language: {typescript | javascript | python | php | go}
database: {default database}

capabilities-covered:
  - {capability-name}        # brief note on what's included
  - {capability-name}

capabilities-partial:
  - capability: {capability-name}
    what-is-included: {what the stack provides}
    what-is-missing: {what Charter still needs to generate}

notes: >
  {2-4 sentences. What this stack is best for, what it opinionatedly excludes,
  any meaningful caveats for Charter users.}
\`\`\`

---
```

Repeat the block for each stack. Order: Node.js/TypeScript stacks first (most common
Charter target), then Python, then others.

## Step 5 — Verify and finalize

Before writing the file, check:
- Every capability-name in `capabilities-covered` and `capabilities-partial` matches
  an actual filename in `capabilities/` (without the .md extension)
- Every `fetch-for-current-state` URL resolves (you fetched it in Step 2)
- No stack is listed as covering a capability it only partially covers

If updating an existing file: stacks that were previously listed but not re-researched
this run should be preserved with their existing data, but their `last-verified` date
should remain unchanged so it's clear they weren't verified in this run.

After writing the file, summarize what changed: stacks added, stacks updated, stacks
removed (if any), and any notable coverage changes you found.
