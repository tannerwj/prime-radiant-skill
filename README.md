# prime-radiant-skill

Agent skill for [Prime Radiant](https://github.com/tannerwj/prime-radiant) — a personal knowledge vault with semantic search, wikilink graphs, and structured metadata.

Teaches any agent how to read, search, and write notes in your vault. Reads go through the Cloudflare Worker MCP (search, embeddings, graph). Writes go through the Obsidian CLI to keep the local vault as source of truth.

Works with any agent platform that supports [AgentSkills](https://agent-skills.md) (OpenClaw, Claude Code, etc.).

## Prerequisites

- [Obsidian](https://obsidian.md) with the Prime Radiant vault open (for CLI writes)
- A deployed [Prime Radiant worker](https://github.com/tannerwj/prime-radiant/tree/master/worker) (for MCP reads/search)
- [mcporter](https://mcporter.dev) installed (`npm install -g mcporter`)

## Setup

### 1. Set environment variables

```bash
export PRIME_RADIANT_URL="https://prime-radiant.<subdomain>.workers.dev"
export PRIME_RADIANT_TOKEN="<your-api-token>"
```

### 2. Register the MCP server

Import the included config:
```bash
mcporter config import ./mcporter.json
```

Or add manually:
```bash
mcporter config add prime-radiant \
  --url "$PRIME_RADIANT_URL/mcp" \
  --transport http \
  --header "Authorization: Bearer $PRIME_RADIANT_TOKEN"
```

### 3. Verify

```bash
mcporter list prime-radiant --schema
mcporter call prime-radiant.vault_stats
```

### 4. Install the skill

**OpenClaw:**
```bash
pnpm dlx add-skill https://github.com/tannerwj/prime-radiant-skill
```

**Claude Code:**
Copy or symlink `SKILL.md` into your project's skills directory.

**Other agents:**
Include the contents of `SKILL.md` in your agent's system prompt or skill configuration.

## What the agent gets

**7 read-only MCP tools** via mcporter (Cloudflare Worker):

| Tool | Purpose |
|------|---------|
| `vault_search` | Hybrid, keyword, or semantic search (D1 FTS5 + Vectorize embeddings) |
| `vault_read` | Read full note content (from R2) |
| `vault_list` | Filter by type, status, tag (from D1) |
| `vault_graph` | Wikilink traversal — local or full vault (from D1) |
| `vault_tags` | All tags with counts |
| `vault_backlinks` | Reverse link lookup |
| `vault_stats` | Vault metrics |

**Obsidian CLI** for all writes (create, append, move, delete, property management).

**Skill instructions** that teach the agent:
- Architecture: write locally via Obsidian CLI, read remotely via Worker MCP
- Search strategy ladder (hybrid → semantic → keyword → graph)
- Note classification (11 types with decision tree)
- Frontmatter schema and tagging conventions
- Wikilink patterns and output citation guidelines

## Invoke

```
/prime-radiant
```
