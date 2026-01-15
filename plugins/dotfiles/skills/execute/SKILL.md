---
name: execute
description: Run all phases of a design plan automatically. Use when asked to "execute a design", "run the phases", "build the design plan", or "implement the design". Spawns parallel subagents for speed.
allowed-tools: Task, Read, Write, Glob, Grep, Bash
---

# Execute

Orchestrate design plan execution using parallel subagents.

## Overview

`/execute <design-dir>` runs all pending phases from a design plan:

1. **Validates** manifest and phase files
2. **Groups** phases into waves by dependencies
3. **Spawns** build agents in parallel for each wave
4. **Reviews** code (optional, per-phase)
5. **Auto-fixes** issues up to 3 times

## Quick Reference

| Agent | Model | Purpose |
|-------|-------|---------|
| Build | sonnet (or `model: haiku`) | Execute phase tasks |
| Review | sonnet | Validate code quality |
| Fix | haiku | Auto-fix review issues |

### Phase Frontmatter Options

```yaml
model: haiku        # Cheaper build agent
review: skip        # Skip review entirely
review: verify-only # Just check verification passed
```

### Flags

```bash
/execute design-auth              # Parallel execution
/execute design-auth --skip-review # Skip all reviews
/execute design-auth --sequential  # No parallelization
/execute design-auth --dry-run     # Show plan only
```

## References

- [Subagent prompts](references/subagent-prompts.md) — Prompt templates for each agent
- [Auto-fix loop](references/auto-fix.md) — How fix retries work
- [Parallel execution](references/parallel-execution.md) — Wave-based parallelization
- [Error handling](references/error-handling.md) — Failure recovery
