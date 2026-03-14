# prime-radiant-skill

Agent skill for [Prime Radiant](https://github.com/tannerwj/prime-radiant) — a personal knowledge vault with semantic search, wikilink graphs, and structured metadata.

Teaches any agent how to search, read, write, and organize notes in your vault. Works with any agent platform that supports [AgentSkills](https://agent-skills.md) (OpenClaw, Claude Code, etc.).

## Prerequisites

- A deployed [Prime Radiant worker](https://github.com/tannerwj/prime-radiant/tree/master/worker)
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

**10 vault tools** via mcporter MCP:

| Tool | Purpose |
|------|---------|
| `vault_search` | Hybrid, keyword, or semantic search |
| `vault_read` | Read full note content |
| `vault_write` | Create or update note (markdown + frontmatter) |
| `vault_append` | Append to existing note |
| `vault_delete` | Delete note |
| `vault_list` | Filter by type, status, tag |
| `vault_graph` | Wikilink traversal (local or full) |
| `vault_tags` | All tags with counts |
| `vault_backlinks` | Reverse link lookup |
| `vault_stats` | Vault metrics |

**Skill instructions** that teach the agent:
- When to check the vault vs. write to it
- Search strategy ladder (semantic → keyword → hybrid → graph)
- Note classification (11 types with decision tree)
- Frontmatter schema and tagging conventions
- Wikilink patterns and output citation guidelines

## Invoke

```
/prime-radiant
```
