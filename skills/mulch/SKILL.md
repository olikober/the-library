---
name: mulch
description: Use when the user wants to set up, configure, or use mulch for structured expertise management for AI agent workflows. Triggers include requests to "use mulch", "set up mulch", "install mulch", "configure mulch expertise", "record expertise with mulch", "query mulch", or any mulch-related task.
---

# Mulch Skill

You are an expert in using Mulch - a structured expertise management tool for AI agent workflows. Mulch helps agents accumulate knowledge across sessions, domains, and teammates.

## What is Mulch?

Mulch is a passive CLI tool for managing structured expertise files that agents record and query. It has:
- **Zero LLM dependency** - Mulch makes no LLM calls
- **Provider-agnostic** - Any agent with bash access can use it
- **Git-native** - Everything lives in `.mulch/`, tracked in version control
- **Append-only JSONL** - Zero merge conflicts, trivial schema validation

## Installation

### Production Installation
```bash
bun install -g @os-eco/mulch-cli
```

### Try Without Installing
```bash
npx @os-eco/mulch-cli --help
```

### Development Setup
```bash
git clone https://github.com/jayminwest/mulch
cd mulch
bun install
bun link              # Makes 'ml' available globally
```

## Quick Start

```bash
ml init                                            # Create .mulch/ in your project
ml add database                                    # Add a domain
ml record database --type convention "Use WAL mode for SQLite"
ml record database --type failure \
  --description "VACUUM inside a transaction causes silent corruption" \
  --resolution "Always run VACUUM outside transaction boundaries"
ml query database                                  # See accumulated expertise
ml prime                                           # Get full context for agent injection
ml prime database                                  # Get context for one domain only
```

## Core Commands

| Command | Description |
|---------|-------------|
| `ml init` | Initialize `.mulch/` in the current project |
| `ml add <domain>` | Add a new expertise domain |
| `ml record <domain> --type <type>` | Record an expertise record |
| `ml edit <domain> <id>` | Edit an existing record by ID or 1-based index |
| `ml delete <domain> [id]` | Delete records by ID, `--records <ids>`, or `--all-except <ids>` |
| `ml query [domain]` | Query expertise |
| `ml prime [domains...]` | Output AI-optimized expertise context |
| `ml search [query]` | Search records across domains with BM25 ranking |
| `ml compact [domain]` | Analyze or apply compaction |
| `ml diff [ref]` | Show expertise changes between git refs |
| `ml status` | Show expertise freshness and counts |
| `ml validate` | Schema validation across all files |
| `ml doctor` | Run health checks on expertise records |
| `ml setup [provider]` | Install provider-specific hooks |
| `ml onboard` | Generate AGENTS.md/CLAUDE.md snippet |
| `ml prune` | Remove stale tactical/observational entries |
| `ml ready` | Show recently added or updated records |
| `ml sync` | Validate, stage, and commit `.mulch/` changes |
| `ml outcome <domain> <id>` | Append an outcome to a record |
| `ml upgrade` | Upgrade mulch to the latest version |
| `ml learn` | Show changed files and suggest domains for recording |
| `ml completions <shell>` | Output shell completion script |

### Global Flags
- `-v`/`--version` - Show version
- `-q`/`--quiet` - Suppress output
- `--verbose` - Enable verbose logging
- `--timing` - Show timing information
- `--json` - Structured JSON output
- ANSI colors respect `NO_COLOR` environment variable

## Record Types

All record types support optional:
- `--classification` (foundational / tactical / observational)
- `--evidence-commit`, `--evidence-issue`, `--evidence-file`
- `--tags`
- `--relates-to` (cross-domain references using `domain:mx-hash` format)
- `--supersedes`
- `--outcome-status` (success/failure)

### Type: convention
**Required Fields:** `content`
**Use Case:** "Use WAL mode for SQLite connections"
```bash
ml record database --type convention "Use WAL mode for SQLite"
```

### Type: pattern
**Required Fields:** `name`, `description`
**Use Case:** Named patterns with optional file references
```bash
ml record api --type pattern --name "Error Handling Pattern" --description "Use result types for error handling"
```

### Type: failure
**Required Fields:** `description`, `resolution`
**Use Case:** What went wrong and how to avoid it
```bash
ml record database --type failure --description "VACUUM inside a transaction causes silent corruption" --resolution "Always run VACUUM outside transaction boundaries"
```

### Type: decision
**Required Fields:** `title`, `rationale`
**Use Case:** Architectural decisions and their reasoning
```bash
ml record architecture --type decision --title "SQLite over PostgreSQL" --rationale "Local-only product, no network dependency acceptable"
```

### Type: reference
**Required Fields:** `name`, `description`
**Use Case:** Key files, endpoints, or resources worth remembering
```bash
ml record api --type reference --name "API Documentation" --description "https://api.example.com/docs"
```

### Type: guide
**Required Fields:** `name`, `description`
**Use Case:** Step-by-step procedures for recurring tasks
```bash
ml record devops --type guide --name "Deploy to Production" --description "1. Run tests 2. Build Docker 3. Push to registry 4. Deploy to k8s"
```

## Classification Tiers

Records are classified into three tiers that govern shelf life and pruning:

- **foundational** - Core architectural decisions, essential conventions (long-lived)
- **tactical** - Patterns and practices for specific scenarios (medium-lived)
- **temporary** - Observations and temporary workarounds (short-lived, pruned)

## Multi-Agent Safety

Mulch is designed for multi-agent workflows with concurrent access.

### Safety Levels

| Safety Level | Commands | Notes |
|--------------|----------|-------|
| **Fully safe** (read-only) | `prime`, `query`, `search`, `status`, `validate`, `learn`, `ready` | No file writes. Any number of agents, any time. |
| **Safe** (locked writes) | `record`, `edit`, `delete`, `compact`, `prune`, `doctor` | Acquire per-file lock before writing. |
| **Serialize** (setup ops) | `init`, `add`, `onboard`, `setup` | Modify config or external files. Run once during project setup. |

### How Locking Works
- **Advisory file locking** - Write commands acquire a `.lock` file before modifying JSONL
- **Retry logic** - Retries every 50ms for up to 5 seconds
- **Stale lock cleanup** - Locks older than 30 seconds are auto-removed
- **Atomic writes** - Write to temp file first, then rename

### Batch Recording
For recording multiple records atomically:
```bash
# From a JSON file
ml record api --batch records.json

# From stdin
echo '[{"type":"convention","content":"Use UTC timestamps"}]' | ml record api --stdin

# Preview first (dry-run)
ml record api --batch records.json --dry-run
```

## Programmatic API

Mulch exports both low-level utilities and a high-level programmatic API:

```typescript
// High-level API — recommended for most use cases
import {
  recordExpertise,   // Record a new expertise entry (with dedup and locking)
  searchExpertise,   // Search records across domains
  queryDomain,       // Query all records for a domain
  editRecord,        // Edit an existing record by ID
  appendOutcome,     // Append an outcome to a record (with locking)
} from "@os-eco/mulch-cli";

// Scoring utilities
import {
  computeConfirmationScore,
  sortByConfirmationScore,
  getSuccessRate,
} from "@os-eco/mulch-cli";

// Low-level utilities
import {
  readConfig,
  getExpertisePath,
  readExpertiseFile,
  searchRecords,
  appendRecord,
  writeExpertiseFile,
  findDuplicate,
  generateRecordId,
  recordSchema,
} from "@os-eco/mulch-cli";
```

## What's in `.mulch/`

```
.mulch/
├── expertise/
│   ├── database.jsonl        # All database knowledge
│   ├── api.jsonl             # One JSONL file per domain
│   └── testing.jsonl         # Each line is a typed, structured record
└── mulch.config.yaml         # Config: domains, governance settings
```

## Workflow Integration

The critical workflow pattern:

1. **ml init** → Creates .mulch/ with domain JSONL files
2. **Agent reads expertise** → Grounded in everything the project has learned
3. **Agent does work** → Normal task execution
4. **Agent records insights** → Before finishing, writes learnings back to .mulch/
5. **git push** → Teammates' agents get smarter too

Step 4 is **agent-driven**. Before completing a task, the agent reviews its work for insights worth preserving and calls `ml record`.

## Provider Setup

Install provider-specific hooks for agent integration:
```bash
ml setup claude      # Claude Code hooks
ml setup cursor      # Cursor hooks
ml setup codex       # Codex hooks
ml setup gemini      # Gemini hooks
ml setup windsurf    # Windsurf hooks
ml setup aider       # Aider hooks
```

## Example Output

```bash
$ ml query database

## database (6 entries, updated 2h ago)

### Conventions
- Use WAL mode for all SQLite connections
- Migrations are sequential, never concurrent

### Known Failures
- VACUUM inside a transaction causes silent corruption
  → Always run VACUUM outside transaction boundaries

### Decisions
- **SQLite over PostgreSQL**: Local-only product, no network dependency acceptable
```

## Important Notes

- Mulch is a **passive layer** - it does not contain an LLM. Agents use Mulch, Mulch does not use agents.
- All expertise is git-tracked - clone a repo and agents immediately have the project's accumulated expertise.
- JSONL storage means zero merge conflicts between branches.
- Use `ml prime` to generate AI-optimized context for agent injection.
