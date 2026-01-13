---
name: execute
description: Orchestrates full execution of a design plan using specialized subagents. Spawns Explore agent for codebase context, general-purpose agents for building phases, and code-reviewer agents for validation. Includes auto-fix loop for review issues. Use "/execute <design-dir>" to run all pending phases.
---

# Execute

Orchestrate design plan execution using specialized subagents.

## Contents

- [Workflow](#workflow)
- [Subagents](#subagents)
- [Usage](#usage)
- [References](#references)

## Workflow

```
/execute design-auth
         │
         ▼
┌─────────────────────────┐
│  Explore Agent          │  ← Gather codebase patterns
└───────────┬─────────────┘
            │
┌───────────▼─────────────┐
│  For each phase:        │
│                         │
│  Build ──► Review ──┐   │
│              │      │   │
│          CONCERNS?  │   │
│              │      │   │
│              ▼      │   │
│           Fix ◄─────┘   │  ← Auto-fix loop (max 3)
│              │          │
│         Re-Review       │
│              │          │
│           PASS ─► Next  │
└─────────────────────────┘
```

## Subagents

| Agent | Type | Purpose |
|-------|------|---------|
| Explore | `Explore` | Fast codebase reconnaissance before execution |
| Build | `general-purpose` | Execute phase tasks |
| Review | `code-reviewer` | Validate code quality |
| Fix | `general-purpose` | Auto-fix review issues |

## Execution Steps

### 1. Pre-execution: Explore

Spawn `Explore` agent to gather codebase context:
- Existing patterns and conventions
- Integration points
- Testing patterns

This context is passed to all subsequent build agents.

### 2. Load Design

1. Read `manifest.yaml` for phase order and status
2. Read `design.md` for architectural context
3. Identify phases with `status: pending`

### 3. Per-Phase Execution

For each pending phase (respecting `depends-on`):

**Build** → Spawn `general-purpose` agent with:
- Codebase context (from Explore)
- Design context (from design.md)
- Previous phase output
- Phase file content

**Review** → Spawn `code-reviewer` agent to check:
- Correctness against design
- Code quality and patterns
- Security issues
- Test coverage

**Auto-Fix** → If review finds CONCERNS:
- Spawn `general-purpose` agent to fix specific issues
- Re-run verification
- Re-run review
- Repeat up to 3 times
- If still failing, pause for manual intervention

**Update** → On success:
- Update manifest status to `complete`
- Update phase frontmatter
- Continue to next phase

## Usage

```bash
/execute design-auth              # Run all phases with auto-fix
/execute design-auth --no-auto-fix # Pause on review issues
/execute design-auth --skip-review # Skip review (fastest)
/execute design-auth --resume      # Continue after manual fix
/execute design-auth --dry-run     # Show plan without executing
```

## Progress Output

```
Executing design-auth (3 phases)

Pre-execution:
  Explore: Complete

Phase 1: core-types
  Build:  PASS (3 files)
  Review: CONCERNS
  Fix (1/3): ✓ Fixed 2 issues
  Review: PASS

Phase 2: middleware
  Build:  PASS (2 files)
  Review: PASS

All phases complete.
```

## References

For detailed information:

- **Subagent prompts**: See [references/subagent-prompts.md](references/subagent-prompts.md)
- **Auto-fix loop**: See [references/auto-fix.md](references/auto-fix.md)
- **Parallel execution**: See [references/parallel-execution.md](references/parallel-execution.md)
- **Error handling**: See [references/error-handling.md](references/error-handling.md)
