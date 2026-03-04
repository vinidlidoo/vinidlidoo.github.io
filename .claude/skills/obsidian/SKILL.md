---
name: obsidian
description: Search, read, and edit Obsidian vault notes. Use when the user asks to interact with their vault, or when existing notes could provide useful context for the current task (e.g., research notes for a blog post).
---

# Obsidian Skill

Access the user's Obsidian vault via the `obsidian` CLI to search, read, create, and edit notes.

## When to Use

- When the user asks to search, read, create, or edit Obsidian notes
- When existing vault notes could provide context for the current task
- When asked to summarize what was learned for Obsidian (see global CLAUDE.md instructions)

## Vault Structure

- Default vault: user's primary vault (no `vault=` needed unless targeting a specific one)
- Study notes live in `Study/` (root directory)
- Notes use standard Obsidian markdown with YAML frontmatter and wikilinks

## Sandbox

The Obsidian CLI communicates with the running app via IPC (Unix socket). This **requires `dangerouslyDisableSandbox: true`** on all `Bash` calls — the sandbox blocks socket access. Also requires Obsidian to be running.

The user has `Bash(obsidian *)` in their allow list, so unsandboxed `obsidian` commands won't prompt for permission.

## Core Commands

All commands run via `Bash` tool with `dangerouslyDisableSandbox: true`. Quote values with spaces.

### Search

```bash
# Full-text search
obsidian search query="verkle tree"

# Search with context (shows matching lines)
obsidian search:context query="polynomial commitment" limit=5

# Search within a folder
obsidian search query="KZG" path="Study"
```

### Read

```bash
# Read by file name (like wikilinks, no path needed)
obsidian read file="Verkle Trees"

# Read by exact path
obsidian read path="Study/Verkle Trees.md"
```

### Browse

```bash
# List tags with counts
obsidian tags counts sort=count

# List files in a folder
obsidian files folder="Study"

# Get file info (dates, size, links)
obsidian file file="Verkle Trees"

# List backlinks to a note
obsidian backlinks file="KZG Commitments"
```

### Write

```bash
# Append to an existing note
obsidian append file="Verkle Trees" content="## New Section\nContent here"

# Prepend to a note
obsidian prepend file="Verkle Trees" content="Updated: 2026-03-03\n"

# Create a new note
obsidian create name="New Topic" path="Study/New Topic.md" content="---\ntags:\n  - Programming\n---\n\n# New Topic\n\nContent here"

# Set a frontmatter property
obsidian property:set file="New Topic" name="tags" value="Programming, DevOps" type=list
```

## Tips

- `file=` resolves by name (like wikilinks); `path=` is exact
- Use `\n` for newlines and `\t` for tabs in content values
- Search returns file paths; read individual files for full content
- The `search:context` variant is more useful than plain `search` since it shows matching lines
