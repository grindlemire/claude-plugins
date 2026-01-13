# Claude Plugins

A collection of Claude Code plugins for AI-assisted software development. Transform feature specifications into structured designs, execute phased implementations, and automate code review workflows.

## Installation

```bash
# Step 1: Add the marketplace
/plugin marketplace add grindlemire/claude-plugins

# Step 2: Install the dotfiles plugin
/plugin install dotfiles@grindlemire-plugins
```

After installation, restart Claude Code to load the plugin.

## Quick Start

```bash
# Design a new feature
/design Add user authentication with JWT tokens

# Execute the design
/execute design-user-auth
```

## What This Does

This plugin turns vague feature requests into structured, reviewable implementations:

1. **Design** - Transform a feature spec into technical architecture and phased tasks
2. **Build** - Execute individual phases with verification steps
3. **Execute** - Orchestrate full implementations using specialized AI subagents
4. **Review** - Automated code review with auto-fix capabilities

## Commands

| Command | Description |
|---------|-------------|
| `/design <spec>` | Create a technical design and phased implementation plan |
| `/build <phase>` | Execute a single phase from a design plan |
| `/execute <design-dir>` | Run all phases with subagent orchestration |
| `/commit` | Stage, commit, and push changes |
| `/pr` | Create a pull request with structured description |
| `/pull` | Pull latest changes with interactive conflict resolution |

## Design Workflow

When you run `/design`, the plugin:

1. **Gathers context** from your codebase (CLAUDE.md, README, relevant files)
2. **Asks clarifying questions** about ambiguous requirements
3. **Produces a design document** with architecture, interfaces, and trade-offs
4. **Breaks work into phases** with specific tasks and verification steps

### Output Structure

```
design-<feature-name>/
├── manifest.yaml       # Phase tracking and status
├── design.md           # Architecture and interfaces
├── phase-1-setup.md    # First implementation phase
├── phase-2-core.md     # Second phase
└── phase-N-final.md    # Final phase
```

## Execute Workflow

The `/execute` command orchestrates multiple AI subagents:

```
┌─────────────────────┐
│   Explore Agent     │  Gather codebase patterns
└──────────┬──────────┘
           ↓
    For each phase:
           ↓
┌─────────────────────┐
│    Build Agent      │  Execute phase tasks
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│   Review Agent      │  Validate code quality
└──────────┬──────────┘
           ↓
      Has issues?
       ↙     ↘
     Yes      No
      ↓        ↓
┌──────────┐   ↓
│Fix Agent │   ↓
└────┬─────┘   ↓
     ↓         ↓
  Re-review    ↓
     ↓         ↓
     └────→ Next Phase
```

- **Explore Agent** - Fast codebase reconnaissance
- **Build Agent** - Execute phase tasks
- **Review Agent** - Code quality validation
- **Fix Agent** - Auto-fix issues (max 3 attempts)

## Additional Skills

### Git Worktree Management

Work on multiple branches simultaneously:

```bash
wt list              # List all worktrees
wt create feature-x  # Create and switch to worktree
wt delete feature-x  # Delete worktree (with safety checks)
```

### Mage Build Commands

Run Go project build targets:

```bash
mage build   # Build the project
mage test    # Run tests
mage -l      # List available targets
```

## Phase File Format

Each phase includes YAML frontmatter:

```yaml
---
phase: 1
name: setup-database
status: pending
dependencies: []
verification: "go test ./internal/db/..."
---

# Phase 1: Setup Database

## Tasks
1. Create database schema in `internal/db/schema.sql`
2. Add migration runner in `internal/db/migrate.go`

## Verification
Run `go test ./internal/db/...` - expect all tests passing

## Output for Next Phase
- Database connection pool available via `db.Pool()`
- Migration system ready for phase 2 schema additions
```

## Configuration

The plugin requires these permissions (configured automatically):

```json
{
  "permissions": {
    "allow": [
      "WebFetch(domain:raw.githubusercontent.com)",
      "Bash(find:*, cat:*, tree:*, mkdir:*, wc:*)"
    ]
  }
}
```

## Project Structure

```
claude-plugins/
├── plugins/
│   └── dotfiles/
│       ├── commands/       # User-facing commands
│       │   ├── design.md
│       │   ├── build.md
│       │   ├── execute.md
│       │   ├── commit.md
│       │   ├── pr.md
│       │   └── pull.md
│       └── skills/         # Core implementations
│           ├── design/
│           ├── build/
│           ├── execute/
│           ├── worktree/
│           └── mage/
└── .claude-plugin/
    └── marketplace.json    # Plugin registry
```
