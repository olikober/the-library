---
name: seeds
description: Use when the user wants to set up, configure, or use seeds for git-native issue tracking for AI agent workflows. Triggers include requests to "use seeds", "set up seeds", "install seeds", "create seeds issue", "list seeds issues", or any seeds-related task.
---

# Seeds Skill

You are an expert in using Seeds - a git-native issue tracker for AI agent workflows. Seeds stores issues as JSONL files in `.seeds/`, making them fully diffable and mergeable via git.

## What is Seeds?

Seeds is a lightweight issue tracker designed for AI agent workflows:
- **Git-native** - JSONL storage, no binary DB files
- **Zero dependencies** - chalk + commander only
- **Multi-agent safe** - Advisory locks + atomic writes
- **Part of os-eco** - Works with mulch (expertise) and overstory

## Installation

### Production Installation
```bash
bun install -g @os-eco/seeds-cli
```

### Try Without Installing
```bash
npx @os-eco/seeds-cli --help
```

### Development Setup
```bash
git clone https://github.com/jayminwest/seeds
cd seeds
bun install
bun link              # Makes 'sd' available globally
```

## Quick Start

```bash
# Initialize in your project
sd init

# Create an issue
sd create --title "Add retry logic to mail client" --type task --priority 1

# List open issues
sd list

# Find work (open, unblocked)
sd ready

# Claim and complete
sd update seeds-a1b2 --status in_progress
sd close seeds-a1b2 --reason "Implemented with exponential backoff"

# Commit .seeds/ changes to git
sd sync
```

## Issue Commands

| Command | Description |
|---------|-------------|
| `sd init` | Initialize `.seeds/` in current directory |
| `sd create --title <text>` | Create a new issue |
| `sd show <id>` | Show issue details |
| `sd list` | List issues with filters |
| `sd ready` | Open issues with no unresolved blockers |
| `sd update <id>` | Update issue fields |
| `sd close <id> [<id2> ...]` | Close one or more issues |
| `sd dep add <issue> <depends-on>` | Add dependency |
| `sd dep remove <issue> <depends-on>` | Remove dependency |
| `sd dep list <issue>` | Show deps for an issue |
| `sd block <id> --by <blocker-id>` | Mark issue as blocked |
| `sd unblock <id> --from <blocker-id>` | Remove a blocker |
| `sd blocked` | Show all blocked issues |
| `sd label add <id> <label>` | Add a label |
| `sd label remove <id> <label>` | Remove a label |
| `sd label list <id>` | List labels on an issue |
| `sd label list-all` | List all labels across issues |
| `sd stats` | Project statistics |
| `sd sync` | Stage and commit `.seeds/` changes |

### Create Options
- `--title <text>` (required) - Issue title
- `--type <type>` - Issue type (task, bug, feature, doc)
- `--priority <0-4>` - Priority level (0=Critical, 1=High, 2=Medium, 3=Low, 4=Backlog)
- `--description <text>` - Issue description (alias: `--body`, `--desc`)
- `--assignee <text>` - Assigned person/agent

### Update Options
- `--status <status>` - open, in_progress, blocked, done, closed
- `--title <text>` - New title
- `--priority <0-4>` - New priority
- `--assignee <text>` - New assignee
- `--description <text>` - New description

### List Filters
- `--status <status>` - Filter by status
- `--type <type>` - Filter by type
- `--assignee <text>` - Filter by assignee
- `--label <label>` - Filter by label
- `--limit <num>` - Limit results
- `--all` - Show all issues (including closed)

## Template Commands

| Command | Description |
|---------|-------------|
| `sd tpl create --name <text>` | Create a template |
| `sd tpl step add <id> --title <text>` | Add step (supports `{prefix}` interpolation) |
| `sd tpl list` | List all templates |
| `sd tpl show <id>` | Show template with steps |
| `sd tpl pour <id> --prefix <text>` | Instantiate template into issues |
| `sd tpl status <id>` | Show convoys completion status |

## Health & Utility Commands

| Command | Description |
|---------|-------------|
| `sd doctor` | Check project health and data integrity (`--fix`) |
| `sd prime` | Output AI agent context (`--compact`) |
| `sd onboard` | Add seeds section to CLAUDE.md / AGENTS.md |
| `sd upgrade` | Upgrade seeds to latest version (`--check`) |
| `sd completions <shell>` | Output shell completion script |
| `sd migrate-from-beads` | Import `.beads/issues.jsonl` into `.seeds/` |

### Global Flags
- `-v`/`--version` - Show version
- `-q`/`--quiet` - Suppress output
- `--verbose` - Enable verbose logging
- `--timing` - Show timing information
- `--json` - Structured JSON output
- ANSI colors respect `NO_COLOR` environment variable

## Priority Scale

| Value | Label | Use |
|-------|-------|-----|
| 0 | Critical | System-breaking, drop everything |
| 1 | High | Core functionality |
| 2 | Medium | Default — important but not urgent |
| 3 | Low | Nice-to-have |
| 4 | Backlog | Future consideration |

## Issue Status

| Status | Description |
|--------|-------------|
| `open` | Newly created, not started |
| `in_progress` | Being worked on |
| `blocked` | Cannot proceed due to dependency |
| `done` | Completed (still visible in list) |
| `closed` | Closed (archived, hidden by default) |

## On-Disk Format

```
.seeds/
  config.yaml          # Project config: project name, version
  issues.jsonl         # All issues, one JSON object per line
  templates.jsonl      # Template definitions
  .gitignore           # Ignores *.lock files
```

### Git Configuration

Add to your `.gitattributes` (done automatically by `sd init`):

```
.seeds/issues.jsonl merge=union
.seeds/templates.jsonl merge=union
```

The `merge=union` strategy handles parallel agent branch merges. Seeds deduplicates by ID on read (last occurrence wins), so conflicts resolve automatically.

## Concurrency

Seeds is safe for concurrent multi-agent use:

- **Advisory file locks** - `O_CREAT | O_EXCL`, 30s stale threshold, 100ms retry with jitter, 30s timeout
- **Atomic writes** - temp file + rename under lock
- **Dedup on read** - last occurrence wins after `merge=union` git merges

## JSON Output

All commands support `--json` for structured output:

**Success:**
```json
{ "success": true, "command": "create", "id": "myproject-a1b2" }
```

**Error:**
```json
{ "success": false, "command": "create", "error": "Title is required" }
```

## Integration with Overstory

Overstory wraps `sd` via `Bun.spawn(["sd", ...])` with `--json` parsing:

| BeadsClient method | sd command |
|-------------------|------------|
| `ready()` | `sd ready --json` |
| `show(id)` | `sd show <id> --json` |
| `create(title, opts)` | `sd create --title "..." --json` |
| `claim(id)` | `sd update <id> --status=in_progress --json` |
| `close(id, reason)` | `sd close <id> --reason "..." --json` |

## Workflow Example

```bash
# Start of session - find work
sd ready

# Create new issue
sd create --title "Fix authentication bug" --type bug --priority 1 --description "Users cannot login with OAuth"

# Work on issue
sd update seeds-abc123 --status in_progress

# Add labels
sd label add seeds-abc123 bug
sd label add seeds-abc123 security

# Close issue
sd close seeds-abc123 --reason "Fixed OAuth token refresh logic"

# Sync changes
sd sync
```

## Important Notes

- Seeds uses **issue IDs** like `myproject-a1b2` - format is `{project}-{hash}`
- The `.seeds/` directory is tracked in git - all issue history is version controlled
- Use `sd list --all` to see closed issues
- Use `sd blocked` to find all issues that can't proceed
- Templates support `{prefix}` interpolation for bulk issue creation
