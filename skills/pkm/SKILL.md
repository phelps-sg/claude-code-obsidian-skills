---
name: pkm
model-invocable: true
description: Read and write notes in the shared Obsidian knowledge base
allowed-tools: Read, Write, Glob, Grep, Edit
---

# Personal Knowledge Management (Obsidian)

Manage a shared Obsidian vault at `~/Documents/obsidian-vault/`.

This vault serves as persistent memory across Claude Code sessions and projects. Both the agent and the user can read and write to it. Notes are structured, tagged, and interlinked following the conventions below.

**The codebase is always the ultimate source of truth.** Notes capture understanding, context, and reasoning at a point in time, but code evolves. If a note is significantly older than recent commits on the files it describes, treat it as potentially stale — verify against the code before relying on architectural details, and update the note if it has drifted.

## Vault Structure

**Flat — no folders.** All notes live at the vault root. Organisation is via tags and wiki-links, not folder hierarchy. A note can belong to multiple dimensions simultaneously (e.g. architecture + decision + project), which folders can't express.

```
obsidian-vault/
  api-gateway-design.md          #architecture #project/my-app
  decision-auth-strategy.md      #decision #project/my-app
  domain-order-lifecycle.md      #domain/ecommerce
  retry-strategy-methodology.md  #methodology
  ...
```

## Conventions

### Tags (Primary Organisation)

Tags provide multi-dimensional classification. Every note must have at least one tag.

- `#project/<name>` — link notes to projects (e.g. `#project/my-app`, `#project/shared-libs`)
- `#service/<name>` — which service within a monorepo (e.g. `#service/api`, `#service/worker`, `#service/auth`)
- `#lib/<name>` — shared library (e.g. `#lib/http-client`, `#lib/db-utils`)
- `#architecture` — system design, data flows, contracts
- `#domain/<area>` — domain knowledge (e.g. `#domain/ecommerce`, `#domain/payments`)
- `#decision` — architectural decisions, open questions
- `#pr` — pull request notes tracking WIP, review feedback, and status
- `#sketch` — speculative designs and exploratory thinking not yet reflected in merged code
- `#methodology` — patterns, processes, lessons learned
- `#infra` — infrastructure, deployment, cloud resources
- `#reference` — external papers, API docs, conventions
- `#backlog` — actionable items to be done later (see Backlog section below)

### Wiki Links (Knowledge Graph)

Use `[[note-name]]` to link between notes. This builds the knowledge graph in Obsidian's graph view. Prefer links over duplication — if a concept is explained in another note, link to it.

### File Naming

- Use kebab-case: `api-gateway-design.md`
- Prefix PRs: `pr-NNNNN-short-description.md`
- Prefix decisions: `decision-short-description.md`
- Prefix libraries: `lib-<name>.md`
- Prefix infrastructure: `infra-<topic>.md`

### Note Structure

Every note should have:
1. H1 title
2. Tags on the line immediately after the title
3. **Repo links** — for project-specific notes, include a `**Repo**:` line with the wiki-link to the project note and the local checkout path (e.g. `**Repo**: [[my-app]] (\`~/code/my-app\`)`).
4. Brief overview paragraph
5. Structured content (tables, code blocks, links to other notes)
6. Source references where applicable (file paths, URLs)

### Searching by Tag

Since tags appear as plain text in markdown, use Grep to search for them:

```
# Find all notes tagged with a specific service
Grep: pattern="#service/api" path="~/Documents/obsidian-vault"

# Find all library notes
Grep: pattern="#lib/" path="~/Documents/obsidian-vault"
```

For a **quick taxonomy overview** — every note has its tags on line 2, so a single grep for tag-heavy lines gives you the full relationship map across the vault:

```
Grep: pattern="^#" path="~/Documents/obsidian-vault" output_mode="content"
```

This returns one line per note showing all its tags, giving a cheap holistic view of how projects, services, and libraries interrelate without reading any note bodies.

## Backlog

Use the daily note to capture actionable items that should be done later but not right now. Backlog items are markdown checkboxes tagged with `#backlog`:

```markdown
## Backlog
- [ ] (2026-02-18) Refactor the order validator to use the new schema #backlog #project/my-app
- [ ] (2026-02-17) Write up the retry strategy as a methodology note #backlog
- [ ] (2026-02-15) Investigate flaky test in worker service #backlog #service/worker
```

### Backlog conventions

- Add backlog items to the **current daily note** under a `## Backlog` heading.
- Each item is a markdown checkbox (`- [ ]`) on a single line, prefixed with the date it was added: `- [ ] (YYYY-MM-DD) Description #backlog`.
- Always include the `#backlog` tag. Add other tags (project, service, etc.) for discoverability.
- When an item is completed, tick the checkbox (`- [x]`) and optionally note the date or link to the relevant note/PR.
- When starting a new daily note, carry forward any unchecked `#backlog` items from the previous day.

## When to Use

### Proactively write notes when:
- Tracing a complex data flow through the codebase
- Discovering how a system works (architecture, contracts, data models). When documenting data models, take a **model-centric approach** — document ORM model classes, their relationships, and source file paths rather than raw table names.
- Making or discussing architectural decisions
- Completing a PR or significant piece of work
- Learning domain-specific concepts
- Identifying reusable patterns or methodology insights

### Proactively read notes when:
- **Starting any new session** — do three things to orient:
    1. Read the most recent daily note (`YYYY-MM-DD.md`, sorted descending). If it is brief, read the previous daily note too. Go back up to 3 daily notes to find enough session context.
    2. Run a quick tag-line grep (`^#` across the vault) to get a holistic picture of how projects, services, and libraries interrelate.
    3. Check for open backlog items.
- Working on something that might have been explored before
- Needing context about architecture or domain concepts

## Usage

```
/pkm                    # List all notes in the vault
/pkm read <topic>       # Search for and read notes about a topic
/pkm write <topic>      # Create or update a note
/pkm backlog <item>     # Add a backlog item to today's daily note
```

When invoked without arguments, list the vault contents to orient.
When invoked with `read`, search for relevant notes using Glob and Grep.
When invoked with `write`, create or update a note following the conventions above.
When invoked with `backlog <item>`, append the item as a checkbox to today's daily note under the `## Backlog` heading (creating the heading if it doesn't exist).
