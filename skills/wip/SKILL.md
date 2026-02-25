---
name: wip
model-invocable: true
description: Kanban board of active work — backlog items, own PRs, review requests, related PRs, assigned issues, and sketches
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Work-in-Progress Dashboard

Produce a Kanban board of all active work: backlog items, own PRs, review requests, related PRs by others, assigned issues, and active sketches. Output is an Obsidian Kanban plugin board file for interactive drag-to-reorder viewing.

Sources: Obsidian vault at `~/Documents/obsidian-vault/` and GitHub.

**Note:** This skill template uses a single GitHub repo. Adapt the `--repo` flags to your own repositories. If you use external project management tools (Notion, Jira, Linear, etc.), add additional steps to query them.

## Procedure

### 0. Check previous board for completed items

Before gathering data, check whether a previous board exists for today:

```
Read: ~/Documents/obsidian-vault/wip-YYYY-MM-DD.md
```

If it exists, scan the entire board for checked checkboxes (`- [x]`) that have a `#backlog` tag. For each such card:
1. Extract the item's description text and source note `[[link]]`
2. Read the source note and find the matching `- [ ] ... #backlog` line
3. Change `- [ ]` to `- [x]` in the source note using Edit

Report what was marked done. If no checked backlog items are found or the file doesn't exist, skip silently.

### 1. Open backlog items

Search for unchecked `#backlog` checkboxes across the vault:

```
Grep: pattern="- \[ \].*#backlog" path="~/Documents/obsidian-vault" output_mode="content"
```

**Deduplicate** results by normalising each item: strip the date prefix and any wiki-links, then compare the remaining description text. If the same item appears in multiple notes, keep only one card with the earliest date and the most recent source note.

Group by source note. Show the item text and date.

While gathering backlog items, extract all `#service/*` and `#lib/*` tags — these define the user's **active focus areas** used to filter later sections.

### 2. My PRs

Two sources — the vault and GitHub. Run these in parallel:

**a) Vault PR notes** (`#pr` tag or `pr-` filename prefix):

```
Grep: pattern="#pr\b" path="~/Documents/obsidian-vault" output_mode="files_with_matches"
Glob: pattern="pr-*.md" path="~/Documents/obsidian-vault"
```

For each PR note, extract the PR number from the filename (`pr-NNNNN-*`) and check its status:

```bash
gh pr view <number> --repo <owner>/<repo> --json state,title,mergedAt,closedAt,updatedAt
```

**b) Open PRs on GitHub** (catches PRs with no vault note):

```bash
gh pr list --repo <owner>/<repo> --author @me --state open --json number,title,updatedAt,url
```

Merge both sources. For each PR, report:
- PR number, title, current GitHub state (open/merged/closed)
- Vault note link if one exists, or "(no vault note)" if it only came from GitHub
- Flag merged/closed PRs that still appear in vault as stale

Also fetch changed files for each own open PR to augment focus areas:

```bash
gh pr diff <number> --repo <owner>/<repo> --name-only
```

### 3. Active sketches

Find all notes tagged `#sketch`:

```
Grep: pattern="#sketch" path="~/Documents/obsidian-vault" output_mode="files_with_matches"
```

List each with its title. Extract any `#service/*` or `#lib/*` tags and add to active focus areas.

### 4. Build active focus areas

At this point you have a set of active focus areas derived from:
- `#service/*` and `#lib/*` tags on `#backlog` items
- `#service/*` and `#lib/*` tags on `#sketch` notes
- File paths from own open PRs

This set is used to filter and tier sections 5–6.

### 5. Review Requests

Fetch PRs requesting review from the user:

```bash
gh pr list --repo <owner>/<repo> --search "review-requested:@me" --state open --json number,title,author,updatedAt
```

For each candidate, check which files it touches:

```bash
gh pr diff <number> --repo <owner>/<repo> --name-only
```

Tier the results:
- **Likely relevant** — PR touches paths that overlap active focus areas
- **Other** — blanket CODEOWNERS requests in unrelated areas. Show as a count.

### 6. Related PRs

Fetch recent PRs by others:

```bash
gh pr list --repo <owner>/<repo> --state merged --json number,title,author,mergedAt --limit 20
gh pr list --repo <owner>/<repo> --state open --json number,title,author,updatedAt --limit 20
```

Filter out the user's own PRs. For each remaining candidate, check file overlap with active focus areas. Only include PRs whose changed files overlap.

Add a **one-line editorial summary** at the top of this section describing what work is happening in areas that overlap with yours.

To keep this fast: if the focus areas set is empty, skip this section entirely.

### 7. Build Kanban board

Generate a Kanban board file in Obsidian Kanban plugin format. The file will be saved as `wip-YYYY-MM-DD.md`.

The board has H2 columns with `- [ ]` checkbox cards. Use `--` as field separator within cards. Wiki-links `[[note]]` for navigation, tags for searchability. Empty columns get a single `- [ ] None` card.

**Template:**

```markdown
---
kanban-plugin: basic
---

## Heads Up

## Backlog
- [ ] Description [[source-note]] @{YYYY-MM-DD} #backlog

## My PRs
- [ ] [#NNNNN](https://github.com/<owner>/<repo>/pull/NNNNN) Title -- state [[pr-nnnnn-description]]

## Review Requests
- [ ] [#NNNNN](https://github.com/<owner>/<repo>/pull/NNNNN) Title -- @author -- touches focus/area #review
- [ ] N other blanket CODEOWNERS requests #review/other

## Related PRs
- [ ] Context: Active work near your focus -- one-line editorial summary
- [ ] [#NNNNN](https://github.com/<owner>/<repo>/pull/NNNNN) Title -- @author -- merged YYYY-MM-DD

## Sketches
- [ ] [[note-name]] Title #sketch

## Active


%% kanban:settings
{"kanban-plugin":"basic"}
%%
```

**Card format rules:**
- All cards are `- [ ]` single-line checkboxes (mandatory for Kanban plugin)
- Use `--` as field separator (not `—`)
- Wiki-links `[[note]]` for navigation, tags for searchability
- `@{YYYY-MM-DD}` dates only on Backlog cards (date the item was added)
- Empty columns get a single `- [ ] None` card
- **Heads Up** is a drag-to-prioritise column — the user moves cards here from other columns in Obsidian. The skill leaves it empty unless there are notable items (imminent deadlines, etc.).
- **Active column** is always the last column and starts empty. The user drags items they're currently working on here.

### 8. Editorial observations (terminal only)

After the board markdown, print a short "Notes" section in the terminal (not on the board) with observations. Possible triggers:
- Merge conflicts / drift risk — related PRs that just merged into areas you have open PRs in
- Review requests blocking others — especially those waiting several days
- Stale own PRs — open with no update in 7+ days
- Relevant work by others that changes assumptions

If nothing is notable, skip the Notes section.

### 9. Save Kanban board

After presenting the board in the terminal, ask the user whether they'd like to save it using `AskUserQuestion`. If the user declines, stop.

If the user accepts:

**a) Write the Kanban file.** Write `~/Documents/obsidian-vault/wip-YYYY-MM-DD.md` with the full board content.

**b) Link from the daily note.** If today's daily note doesn't contain `[[wip-YYYY-MM-DD]]`, append a `## WIP` section with the link.

## Constraints

- **Read-only during gathering.** Do not edit any vault notes while gathering data (steps 1–7). The only exception is step 0, which marks completed backlog items. Only write to the vault in step 9, and only if the user opts in.
- **Fast.** Use parallel tool calls where possible. Limit diff checks to the 10 most recent PRs if there are many candidates.
- **No verbosity.** One line per item. The user wants an at-a-glance dashboard, not prose.
- **Kanban format compliance.** Generated file must have YAML frontmatter with `kanban-plugin: basic`, H2 headings for columns, `- [ ]` checkboxes for cards, and the settings block at the end.

## Usage

```
/wip           # Generate Kanban board of active work
```
