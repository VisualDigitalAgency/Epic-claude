# epic-claude

> CLI tool for generating and managing software architecture using AI

[![CI](https://github.com/VisualDigitalAgency/Epic-claude/actions/workflows/blank.yml/badge.svg?branch=main)](https://github.com/VisualDigitalAgency/Epic-claude/actions/workflows/blank.yml)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.1.0-green.svg)](https://github.com/VisualDigitalAgency/Epic-claude/releases/tag/v1.1.0)
[![PyPI](https://img.shields.io/badge/PyPI-coming%20soon-lightgrey.svg)](#install)

Describe what you want to build. `epic` asks the right questions, proposes a tech stack, generates a complete architecture, and syncs it into your project so Claude Code always has full context.

---

## What it does

```
epic plan "SaaS invoicing app"
  │
  ├── Asks clarifying questions (scale, database, auth, deployment)
  ├── Proposes a tech stack — you confirm or adjust
  ├── Validates a 6-point pre-generation checklist
  ├── Generates 5 architecture artifacts (PRD, API spec, schema, stack, boundaries)
  ├── Validates every artifact before writing to disk
  └── Syncs architecture context into CLAUDE.md
```

After that, Claude Code opens your project and builds with full architectural awareness — no context loss, no guessing, no drift.

---

## Generated artifacts

Every artifact is validated before it touches disk. All fail together or succeed together — no partial writes.

| File | Format | What it contains |
|---|---|---|
| `prd.md` | Markdown | Feature-by-feature PRD with F-IDs, acceptance criteria, API endpoints, DB tables |
| `api.yaml` | OpenAPI 3.1 | Full API spec validated by `openapi-spec-validator` |
| `schema.sql` | SQL | Complete schema with FK integrity, validated by `sqlglot` |
| `techstack.md` | Markdown | Confirmed stack with rationale and alternatives considered |
| `boundaries.md` | Mermaid | Service boundary map with ownership rules |

---

## Install

> **Note:** epic-claude is not yet on PyPI. Install directly from the GitHub release below.
> PyPI publishing is planned — see [Coming soon](#coming-soon).

### Current method — install from GitHub release

**Step 1** — Download the wheel from the [Releases page](https://github.com/VisualDigitalAgency/Epic-claude/releases/tag/v1.1.0):

```
epic_claude-1.1.0-py3-none-any.whl
```

**Step 2** — Install with pipx (recommended):

```bash
# Install pipx if you don't have it
pip install --user pipx && pipx ensurepath

# Install the wheel
pipx install epic_claude-1.1.0-py3-none-any.whl

# Verify
epic version
```

Or with pip:

```bash
pip install epic_claude-1.1.0-py3-none-any.whl
```

Or directly from the GitHub repo:

```bash
pipx install git+https://github.com/VisualDigitalAgency/Epic-claude.git
```

**Why pipx?** Global CLI tools should be isolated. pipx gives `epic` its own environment — no conflicts with your project venv, ever.

### PATH not found after install?

```bash
pipx ensurepath
source ~/.bashrc    # or ~/.zshrc, or restart terminal
```

### Coming soon — PyPI install *(not yet available)*

Once published to PyPI, installation will be:

```bash
# pipx (recommended)
pipx install epic-claude

# uv
uv tool install epic-claude

# pip
pip install epic-claude
```

---

## First-time setup

```bash
# 1. Initialise the global registry (~/.epic/)
#    This creates the global state directory — run once per machine
epic init

# 2. Set your Anthropic API key — never stored in config files
export ANTHROPIC_API_KEY=sk-ant-...
# Add to ~/.bashrc or ~/.zshrc to persist

# 3. Health check — confirms everything is wired
epic doctor
```

> **Note:** `epic init` sets up the global registry (`~/.epic/`). It does not create a
> project. To start a project, `cd` into your project directory and run `epic plan`.

`epic doctor` output when everything is ready:

```
  ✓ OK   Python version:    3.10.x (or higher)
  ✓ OK   Home directory:    ~/.epic
  ✓ OK   config.json
  ✓ Set  ANTHROPIC_API_KEY: sk-...xxxx
  ✓ OK   epic in PATH:      /home/user/.local/bin/epic
  ✓ OK   Schema versions:   v1
```

---

## Connect to Claude Code

Add to `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "epic": {
      "command": "epic",
      "args": ["system", "serve"]
    }
  }
}
```

Restart Claude Code. `epic` appears in the MCP tools panel. Claude Code can now call all 18 architecture tools directly mid-conversation.

---

## Quickstart

### Start a new project

```bash
cd ~/projects/my-app

epic plan "multi-tenant SaaS invoicing app in Python"
```

`epic` will:

1. Ask targeted questions about scale, database, auth, and deployment
2. Infer what it can from your description — never re-asks what you already said
3. Propose a full tech stack — you confirm or adjust individual decisions
4. Generate all 5 architecture artifacts
5. Write context into `CLAUDE.md` — Claude Code reads this automatically

The whole flow takes 2–3 minutes for a typical project.

### Non-interactive (CI/CD)

```bash
epic plan --config requirements.yaml --yes
```

`requirements.yaml`:

```yaml
description: "multi-tenant SaaS invoicing app in Python"
answers:
  scale: "1k-10k users"
  database: "PostgreSQL"
  deployment: "Railway"
  auth: "JWT"
```

---

## Supported project types

| Type | Description |
|---|---|
| `backend_saas` | Multi-tenant API, subscriptions, billing, auth |
| `rest_api` | Resource-oriented API, CRUD, webhooks, versioning |
| `fullstack_web` | Frontend + backend + database + auth |

Mobile, infra, data pipelines, and AI/ML projects are planned for v2.

---

## Command reference

### Core workflow

```bash
epic plan "<description>"          # Full flow: clarify → stack → generate → sync
epic plan --config file.yaml       # Non-interactive (CI)
epic plan --yes                    # Skip confirmations
epic plan --no-sync                # Stop before CLAUDE.md sync

epic generate                      # Re-run architect agent (all 5 artifacts)
epic generate --scope prd          # Regenerate one artifact
epic generate --scope api_spec     # Valid: prd, api_spec, db_schema, tech_stack, boundaries
epic generate --yes                # Skip assumption prompts (CI)

epic sync                          # Sync architecture to CLAUDE.md
epic sync --task "build payments"  # Task-scoped context
epic sync --dry-run                # Preview without writing
epic sync --force                  # Overwrite manual edits
epic sync --rollback               # Restore from last backup

epic status                        # Project health dashboard
```

### Tech stack

```bash
epic tech show                     # Show current confirmed stack
epic tech confirm                  # Lock the proposed stack
epic tech adjust                   # Change one category interactively
epic tech adjust --category database --name MySQL --rationale "client preference"
```

### Projects

```bash
epic project list                  # List all registered projects
epic project switch                # Interactive project switcher
epic project switch <id-prefix>    # Switch by ID prefix or path
```

### System

```bash
epic system serve                  # Start MCP server (stdio)
epic system migrate                # Run DB schema migrations
epic system migrate --check        # Check for pending migrations
epic system doctor                 # Health check
```

### Debug

```bash
epic debug memory list             # List memory entries
epic debug memory list --type decision
epic debug memory search "JWT"     # Full-text search
epic debug memory show <id>        # Full entry detail
epic debug artifacts               # List generated artifacts
```

### Global options

All commands accept:

```bash
--json          # Raw JSON output (scripting-friendly)
--quiet         # Suppress decorative output
--project-id    # Override auto-detection
```

---

## How it works

### Project detection

`epic` looks for `.epic/project.json` in the current directory and parent directories. If found — you're in a project. If not found, commands that require a project will tell you exactly what to do:

```
No epic project found in this directory.

Start here:
  epic plan "describe your project"

Or switch to an existing project:
  epic project list
  epic project switch
```

No silent fallbacks. No wrong-project accidents.

### Execution model

`epic` is **per-command, per-invocation** — no background daemon, no ports, no zombie processes.

```
Claude Code  ──JSON-RPC 2.0──▶  epic system serve  ──▶  exits on disconnect
CLI user     ──────────────▶  epic <command>  ──▶  runs → writes → exits
```

Transport is stdin/stdout. Every write is atomic (`tmpfile → rename`). Safe to kill at any time.

### Validation gates

Three gates run before anything reaches disk:

| Gate | When | What it checks |
|---|---|---|
| Gate 1 | Before generation | 6-point checklist: project type, deployment, auth, scale, multi-tenancy, data ownership |
| Gate 2 | After generation, before write | Per-artifact validation (OpenAPI, SQL syntax, F-ID uniqueness, Mermaid syntax, stack completeness, SHA256) |
| Gate 3 | After write | Re-reads each file, verifies checksum matches — rolls back on mismatch |

All 5 artifacts pass Gate 2 or none are written.

### CLAUDE.md sync

`epic` writes only to a clearly delimited managed section inside `CLAUDE.md`:

```
<!-- EPIC-CLAUDE-MANAGED-START | version:4 | synced:2026-03-23T... | hash:a3f9... -->
## Architecture Context (do not edit manually)
...
<!-- EPIC-CLAUDE-MANAGED-END -->
```

Rules:
- Content outside the markers is **never read, never modified, never deleted**
- Every sync creates a timestamped backup first (30-day retention)
- If you edit inside the markers manually, `epic sync` detects it and prompts:

```
⚠  Manual edit detected inside managed section.
   epic sync --force     → overwrite with current architecture
   epic sync --rollback  → restore from backup
```

### State layout

```
~/.epic/                            ← Global registry — created by: epic init
├── registry.db                     ← Project index
├── config.json                     ← Settings (no credentials ever stored here)
├── server.log                      ← MCP server log (auto-rotated)
└── projects/
    └── <project-uuid>/
        ├── memory.db               ← Source of truth: all decisions, artifacts
        ├── project.json            ← Project metadata + schema versions
        ├── architecture/           ← Generated artifacts (derived from memory.db)
        │   ├── prd.md
        │   ├── api.yaml
        │   ├── schema.sql
        │   ├── techstack.md
        │   └── boundaries.md
        └── claude_md_backups/      ← Pre-sync backups (30-day retention)

your-project/                       ← Your code repository
└── .epic/                          ← Created by: epic plan
    └── project.json                ← Local marker (used for project detection)
```

State authority (highest → lowest): `memory.db → architecture/ → CLAUDE.md → registry.db`

`CLAUDE.md` is always derived. Delete it — `epic sync` rebuilds it in seconds.

---

## MCP tools

When connected via Claude Code, all 18 tools are available:

| Tool | What it does |
|---|---|
| `project_start` | Begin a project — detection → clarification |
| `clarify_answer` | Submit answers (including follow-ups for unknowns) |
| `clarify_status` | Check clarification completeness |
| `clarify_revise` | Revise an answer before proceeding |
| `stack_decide` | Generate proposed tech stack from clarification |
| `stack_confirm` | Lock the tech stack — required before generation |
| `stack_adjust` | Change one category without re-clarifying |
| `stack_get` | Retrieve current confirmed stack |
| `architect_generate` | Run checklist → generate → validate → persist |
| `architect_regenerate` | Regenerate all or one artifact type |
| `artifact_get` | Get a specific artifact by type |
| `artifact_list` | List artifacts with validation status |
| `memory_write` | Store an architectural decision with confidence level |
| `memory_recall` | Query memory (full-text + tag + type + confidence) |
| `memory_list` | List entries with filters |
| `memory_supersede` | Mark a decision outdated (never deletes — full audit trail) |
| `context_sync` | Run full sync state machine → write CLAUDE.md |
| `context_recall` | Get task-relevant context without writing |

---

## Upgrading

### Current method — reinstall from new release

Download the latest wheel from the [Releases page](https://github.com/VisualDigitalAgency/Epic-claude/releases/tag/v1.1.0), then:

```bash
# Reinstall with pipx
pipx install --force epic_claude-<new-version>-py3-none-any.whl

# Always run after upgrade — applies any schema changes
epic system migrate

# Confirm everything still works
epic doctor
```

### Coming soon — PyPI upgrade *(not yet available)*

Once on PyPI, upgrading will be:

```bash
pipx upgrade epic-claude
epic system migrate
epic doctor
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `epic: command not found` | `pipx ensurepath` then restart terminal |
| `ANTHROPIC_API_KEY is not set` | `export ANTHROPIC_API_KEY=sk-ant-...` |
| `No epic project found` | `cd` to your project root, then run `epic plan "describe your app"` |
| Stack not confirmed error | `epic tech show` then `epic tech confirm` |
| Manual edit detected | `epic sync --force` to overwrite, or move edits outside the managed section markers |
| Schema mismatch after upgrade | `epic system migrate` |
| CLAUDE.md corrupted | `epic sync --rollback` |
| Wrong project detected | `epic --project-id <id> <command>` |

---

## Design principles

**Clarify first.** Never assume. Ask exactly one follow-up for unknown answers. Document everything it can't know.

**Validate before write.** No artifact reaches disk without passing all three gates. If any gate fails, nothing is written. Existing files are preserved.

**State has one owner.** `memory.db` is the source of truth. Every other file is derived from it and can be reconstructed.

**Sync is safe by design.** Always backup before write. Only touch the managed section. Verify checksum after write. Roll back on mismatch.

**Fail loudly, recover cleanly.** No silent failures. Every error is logged, structured, and actionable. Every failure has a recovery path.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

Issues and PRs welcome at [github.com/VisualDigitalAgency/Epic-claude](https://github.com/VisualDigitalAgency/Epic-claude/issues).

When referencing features in issues or PRs, use the `EC-FXXX` prefix for `epic` platform features, and `FXXX` for features in a user's generated project. These namespaces must never be mixed.

---

## License

MIT — see [LICENSE](LICENSE).
