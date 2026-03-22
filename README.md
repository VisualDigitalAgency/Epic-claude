Here's the complete install and usage guide for EPIC-Claude.

---

## Install

### Option 1 — pipx (recommended, isolated)

```bash
# Install pipx if you don't have it
pip install --user pipx && pipx ensurepath

# Install from the wheel file
pipx install epic_claude-1.0.0-py3-none-any.whl

# Verify
epic-claude version
```

### Option 2 — pip (if you prefer your current env)

```bash
pip install epic_claude-1.0.0-py3-none-any.whl
```

---

## First-time Setup

```bash
# 1. Initialise the global registry (~/.epic-claude/)
epic-claude init

# 2. Set your Anthropic API key (required for LLM calls)
export ANTHROPIC_API_KEY=your-key-here

# Add to ~/.bashrc or ~/.zshrc so it persists:
echo 'export ANTHROPIC_API_KEY=your-key-here' >> ~/.bashrc

# 3. Health check — confirms everything is wired
epic-claude doctor
```

**Expected `doctor` output:**
```
  ✓ OK  Python version: 3.11.x
  ✓ OK  Home directory: ~/.epic-claude
  ✓ OK  config.json
  ✓ Set  ANTHROPIC_API_KEY: sk-...xxxx
  ✓ Up to date  Schema versions: registry v1
```

---

## Add to Claude Code

This is what makes EPIC-Claude work as an MCP server — Claude Code can call all 18 tools directly.

```bash
# Edit Claude Code's settings file
nano ~/.claude/settings.json
```

Add:

```json
{
  "mcpServers": {
    "epic-claude": {
      "command": "epic-claude",
      "args": ["serve"]
    }
  }
}
```

Restart Claude Code. You'll see EPIC-Claude appear in the MCP tools panel.

---

## Start Your First Project

### From Claude Code (recommended)

Once the MCP server is connected, just describe your project in chat:

```
I want to build a multi-tenant SaaS invoicing app in Python with FastAPI
```

Claude Code will automatically call:
1. `project_start` → detects/creates project, begins clarification
2. `clarify_answer` → you answer questions about scale, auth, DB, deployment
3. `stack_decide` / `stack_confirm` → review and lock the tech stack
4. `architect_generate` → generates all 5 architecture artifacts
5. `context_sync` → writes architecture context to your `CLAUDE.md`

### From the CLI

```bash
# Navigate to your project folder first
cd ~/projects/my-saas-app

# Start a new project (interactive flow)
epic-claude plan "multi-tenant SaaS invoicing app in Python with FastAPI"

# Check what was generated
epic-claude status

# Sync architecture context to CLAUDE.md
epic-claude sync

# See the generated artifacts
ls .epic-claude/    # project.json lives here
cat CLAUDE.md       # architecture context synced here
```

---

## Day-to-Day Usage

```bash
# See all projects
epic-claude projects list

# Check active project status
epic-claude status

# View the confirmed tech stack
epic-claude stack show

# Change one stack decision (no re-clarification needed)
epic-claude stack adjust --category database --name MySQL --rationale "Client prefers MySQL"

# Regenerate all architecture artifacts
epic-claude generate

# Regenerate just one artifact
epic-claude generate --scope prd

# Sync updated architecture to CLAUDE.md
epic-claude sync

# Sync with task context (focuses the managed section)
epic-claude sync --task "implement the payments feature"

# Preview what sync would write without writing
epic-claude sync --dry-run

# Roll back CLAUDE.md to previous version
epic-claude sync --rollback

# Search your architecture decisions
epic-claude memory search "JWT auth"

# List all decisions
epic-claude memory list --type decision

# Show full entry
epic-claude memory show <id>
```

---

## Project Structure After First Run

```
~/projects/my-saas-app/
├── CLAUDE.md                    ← synced architecture context (managed section)
├── .epic-claude/
│   └── project.json             ← project identity + schema version

~/.epic-claude/                  ← global state (all projects)
├── registry.db                  ← project index
├── config.json                  ← settings
├── server.log                   ← MCP server log
└── projects/
    └── <project-id>/
        ├── memory.db            ← source of truth (decisions, artifacts)
        ├── architecture/
        │   ├── prd.md           ← feature-by-feature PRD
        │   ├── api.yaml         ← OpenAPI 3.1 spec
        │   ├── schema.sql       ← database schema
        │   ├── techstack.md     ← confirmed stack
        │   └── boundaries.md   ← service boundary map + Mermaid diagram
        └── claude_md_backups/   ← timestamped CLAUDE.md backups
```

---

## CI/CD Usage

All interactive prompts can be suppressed for automation:

```bash
# Non-interactive: skip assumption confirmation
epic-claude generate --yes

# Force sync without manual-edit prompt
epic-claude sync --force

# All output as JSON (for scripting)
epic-claude status --json
epic-claude stack show --json

# Run schema migrations after upgrade
epic-claude migrate
```

---

## Upgrading

```bash
# Install new version
pipx install --force epic_claude-1.1.0-py3-none-any.whl

# Always run migrate after upgrade to apply any schema changes
epic-claude migrate

# Check everything still works
epic-claude doctor
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `epic-claude: command not found` | Run `pipx ensurepath` then restart terminal |
| `ANTHROPIC_API_KEY is not set` | `export ANTHROPIC_API_KEY=your-key` |
| `No project found in this directory` | `cd` to your project root, or run `epic-claude init` then `epic-claude plan` |
| `Stack not confirmed` | Run `epic-claude stack show` then `epic-claude stack confirm` |
| `Manual edit detected` | Either `epic-claude sync --force` to overwrite, or move your edits outside the managed section markers |
| Schema mismatch after upgrade | Run `epic-claude migrate` |
| Corrupted CLAUDE.md | Run `epic-claude sync --rollback` |
