---
name: prime-radiant
description: Personal knowledge vault — search, read, write, and organize notes with semantic search, wikilink graphs, and structured metadata.
user-invocable: true
metadata:
  {
    "openclaw": {
      "emoji": "🔮",
      "requires": { "bins": ["mcporter"], "env": ["PRIME_RADIANT_URL", "PRIME_RADIANT_TOKEN"] },
      "primaryEnv": "PRIME_RADIANT_TOKEN"
    }
  }
---

# Prime Radiant

You have access to the user's personal knowledge vault via MCP tools prefixed `prime-radiant.*`. Use it to persist and retrieve information across conversations.

The vault is a Cloudflare Worker backed by D1 (full-text search), R2 (file storage), and Vectorize (semantic embeddings). All notes are markdown with YAML frontmatter.

## Tools

| Tool | Purpose |
|------|---------|
| `prime-radiant.vault_search` | Search notes — `mode`: hybrid (default), keyword, semantic |
| `prime-radiant.vault_read` | Read full note by path |
| `prime-radiant.vault_write` | Create or update a note (full markdown + YAML frontmatter) |
| `prime-radiant.vault_append` | Append content to an existing note |
| `prime-radiant.vault_delete` | Delete a note |
| `prime-radiant.vault_list` | List notes — filter by `type`, `status`, `tag` |
| `prime-radiant.vault_graph` | Wikilink connection graph — local (with `path`) or full vault |
| `prime-radiant.vault_tags` | All tags with usage counts |
| `prime-radiant.vault_backlinks` | Notes that link to a given note |
| `prime-radiant.vault_stats` | Vault metrics (note count, tag count, link count, type distribution) |

## When to Use the Vault

**Search the vault** when the user:
- Asks about their preferences, history, or past decisions
- Mentions a person, project, or topic that may have notes
- Asks "what do I know about X?" or "remind me about Y"
- Needs context from prior conversations or experiences

**Write to the vault** when you learn:
- Personal preferences, opinions, or tastes
- Information about people in their life
- Project updates, decisions, milestones
- Concepts, ideas, or knowledge worth remembering
- Experiences, reflections, or insights
- Habits, routines, or recurring patterns

## Search Strategy

### 1. Start broad with hybrid search (default)

Combines keyword FTS5 + semantic vector search via Reciprocal Rank Fusion:
```
prime-radiant.vault_search query="morning routine habits"
```

### 2. Use semantic for conceptual queries

Best for "what do I know about X" — finds notes by meaning, not exact words:
```
prime-radiant.vault_search query="decision making under uncertainty" mode=semantic
```

### 3. Use keyword for exact matches

Best for names, specific phrases, property values:
```
prime-radiant.vault_search query="John Smith" mode=keyword
```

### 4. Follow the graph

After finding a relevant note, explore its connections:
```
prime-radiant.vault_graph path="02-notes/Spaced Repetition.md" depth=2
```

### 5. Check backlinks

Find what references a note — reveals context you might miss:
```
prime-radiant.vault_backlinks path="06-people/Sarah Chen.md"
```

### 6. Filter by metadata

List notes by type, status, or tag:
```
prime-radiant.vault_list type=project status=active
prime-radiant.vault_list tag=topic/health
```

## Writing Notes

### Always search before creating

Check if a relevant note already exists. If it does, **append** or **update** instead of creating a duplicate.

### Frontmatter schema

Every note requires YAML frontmatter:

```yaml
---
type: concept
title: "Descriptive Title"
created: YYYY-MM-DD
modified: YYYY-MM-DD
status: seed
tags:
  - topic/subtopic
aliases: []
related:
  - "[[Other Note]]"
source: ""
---
```

### Note types and folders

| Type | Folder | When to use |
|------|--------|-------------|
| `person` | `06-people/` | About a specific person |
| `project` | `03-projects/` | Project updates, tasks, goals |
| `daily` | `01-daily/` | Journal entries (append to `YYYY-MM-DD.md`, don't create new) |
| `concept` | `02-notes/` | Ideas, knowledge, explanations |
| `experience` | `02-notes/` | Past events, memories, lessons |
| `preference` | `02-notes/` | Likes, dislikes, opinions |
| `habit` | `02-notes/` | Routines, behaviors, frequencies |
| `reflection` | `02-notes/` | Introspection, insights |
| `resource` | `05-resources/` | Books, articles, courses |
| `area` | `04-areas/` | Ongoing life areas (health, career, finance) |
| `capture` | `00-inbox/` | Can't classify yet — promote later |

### Classification decision tree

```
About a specific person?         → person     → 06-people/
A project update or task?        → project    → 03-projects/
A daily log or journal entry?    → daily      → append to 01-daily/YYYY-MM-DD.md
A preference or opinion?         → preference → 02-notes/
About a habit or routine?        → habit      → 02-notes/
A past event or memory?          → experience → 02-notes/
A thought or introspection?      → reflection → 02-notes/
Reference material?              → resource   → 05-resources/
A concept, idea, or knowledge?   → concept    → 02-notes/
Too vague to classify?           → capture    → 00-inbox/
```

### Status progression

- **seed** — raw capture, minimal content
- **sprout** — partially developed
- **evergreen** — mature, vetted, reliable
- **archived** — no longer active

### Tagging rules

Three approved namespaces, max 2 levels deep:
```
topic/   — subject domains (topic/psychology, topic/finance, topic/cooking)
area/    — life areas (area/health, area/career, area/relationships)
source/  — provenance (source/book, source/article, source/conversation)
```

Max 2-4 tags per note. Check existing tags before creating new ones:
```
prime-radiant.vault_tags
```

Don't duplicate frontmatter fields as tags (no `#person` tag when `type: person` exists).

### Linking rules

- Link liberally in note body using `[[wikilinks]]`
- Every note should link to at least one other note
- Use `[[Note Title|display text]]` when the full title is awkward in prose
- Backlinks are automatic — only create forward links

### Metadata rules

- **Dates** — always `YYYY-MM-DD`, never relative ("yesterday")
- **Tags** — YAML list syntax, never inline
- **`modified`** — update whenever you meaningfully edit a note
- **`related`** — use `[[wikilink]]` format for curated connections

## Appending to Notes

For daily logs, interaction records, or evolving notes:
```
prime-radiant.vault_append path="01-daily/2026-03-14.md" content="
- 3pm: met with Sarah about project timeline
- decided to push launch to April"
```

For person notes, append to the relevant section:
```
prime-radiant.vault_append path="06-people/Sarah Chen.md" content="
## Interactions
- 2026-03-14: discussed project timeline, she prefers async standups"
```

## Output Guidelines

When returning vault information to the user:

1. **Cite sources** — reference note titles: `(from [[Morning Routine]])`
2. **Distinguish facts from inference** — "Your notes say X" vs "Based on your notes, it seems like Y"
3. **Respect maturity** — `seed` notes are rough captures, `evergreen` notes are vetted
4. **Surface connections** — mention relationships the user might not see
5. **Flag staleness** — if `modified` date is old, note that content may be outdated

## Example Workflows

### User says: "I just had coffee with Jake, he's switching to a new startup"

1. Search: `vault_search query="Jake"` — check for existing person note
2. If found: `vault_append` to Jake's person note with the update
3. If not: `vault_write` a new person note at `06-people/Jake.md`
4. Append to today's daily note: interaction log entry

### User asks: "What do I know about productivity systems?"

1. Search: `vault_search query="productivity systems" mode=semantic`
2. Read top results: `vault_read` the most relevant 2-3 notes
3. Follow graph: `vault_graph` on the most connected result
4. Synthesize: combine information across notes, cite sources

### User shares: "I've been thinking about switching from coffee to matcha"

1. Search: `vault_search query="coffee matcha beverage preferences"`
2. If preference note exists: update it
3. If not: create `02-notes/Beverage Preferences.md` with `type: preference`
4. Link to any existing habit notes about morning routine
