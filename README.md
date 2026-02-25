# Claude Code + Obsidian Skills

A set of [Claude Code](https://claude.ai/claude-code) skills that turn an Obsidian vault into a shared knowledge base for human and AI collaboration on software projects.

These skills are companion material for the blog post [A Shared Memory for Claude Code: Using Obsidian to Organise Human and AI Knowledge](https://sphelps.substack.com/).

## What's included

| Skill | Invocation | Purpose |
|-------|-----------|---------|
| **Vault Manager** | `/pkm` | Core read/write skill. Defines vault conventions and instructs the agent to write notes when discovering architecture, and to orient at the start of each session. |
| **Vault Auditor** | `/vault-insights` | Periodic audit that checks the vault for contradictions, broken links, and latent cross-cutting insights implied by the graph structure. |
| **External Sync** | `/external-sync` | Bridges an external knowledge source (e.g. Notion, Confluence) into the vault, with deduplication and human confirmation. |
| **WIP Dashboard** | `/wip` | Generates an Obsidian Kanban board from the vault, GitHub, and external project tools. |

## Installation

1. Copy the skill directories into your Claude Code project's `.claude/skills/` directory (or your global `~/.claude/skills/`).
2. Edit the vault path in each `SKILL.md` to point to your Obsidian vault.
3. Adapt the tag taxonomy to your domain (the examples use `#service/`, `#project/`, `#domain/` — replace with your own).
4. For the External Sync skill, replace the Notion MCP tool references with whatever API your external source uses.
5. For the WIP Dashboard skill, replace the GitHub repo references with your own.

## Adapting to your stack

These skills were extracted from a real working setup and anonymised. You will need to customise:

- **Vault path**: currently set to `~/Documents/obsidian-vault/`
- **Tag taxonomy**: the `#service/`, `#lib/`, `#project/` hierarchy reflects a monorepo with multiple services — adapt to your repo structure
- **External tools**: the sync and WIP skills reference Notion and GitHub — swap for your equivalents
- **File naming prefixes**: `pr-NNNNN-`, `decision-`, `lib-` etc. — adjust to match your workflow

The vault manager skill is the foundation. Start there, get the conventions working, then add the others incrementally.

## License

MIT
