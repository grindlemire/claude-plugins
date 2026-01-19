---
name: execute
description: Run all phases of a design plan automatically. Use when asked to "execute a design", "run the phases", "build the design plan", or "implement the design". Spawns parallel subagents for speed.
allowed-tools: Task, Read, Write, Glob, Grep, Bash
---

# Execute

Orchestrate design plan execution using parallel subagents.

## Non-Negotiable Requirements

These requirements are **mandatory** and must never be violated:

1. **ALL implementation MUST be done by subagents** — The main execution context is for orchestration ONLY. You MUST spawn subagents (via the Task tool) for all code implementation, file modifications, and phase execution. The main context may only: read files, validate manifests, spawn agents, check results, and update status. **NEVER write code or edit files directly in the main execution context.**

2. **Never skip the subagent requirement** — Even for "simple" fixes or "quick" changes, you MUST spawn a subagent. There are no exceptions. If you find yourself tempted to make a direct edit, spawn a Fix agent instead.

Failure to follow these requirements invalidates the execution.

## Overview

`/execute <design-dir>` runs all pending phases from a design plan:

1. **Validates** manifest and phase files
2. **Groups** phases into waves by dependencies
3. **Spawns** build agents in parallel for each wave
4. **Reviews** code (optional, per-phase)
5. **Auto-fixes** issues up to 3 times

**Main context responsibilities (orchestration only):**
- Read manifest.yaml and phase files
- Compute dependency waves
- Spawn Build/Review/Fix agents via Task tool
- Wait for agent completion via TaskOutput
- Update manifest.yaml status fields
- Report progress to user

**Subagent responsibilities (all implementation):**
- Read design and phase files
- Write/edit code files
- Run verification commands
- Write output to `.cache/` directory

## Context Isolation

Each phase executes in a **clean context window** to prevent context bloat and ensure consistent behavior.

### Guarantees

1. **Subagents start fresh** — Each Task tool invocation creates an isolated subprocess with no conversation history from the main context or previous phases.

2. **Information flows only through `.cache/`** — Phases communicate ONLY via structured output files in `.cache/`. No context bleeds between phases.

3. **Main context stays lean** — The orchestrator MUST NOT accumulate phase outputs in its own context. After spawning an agent and confirming success, discard the detailed output. Only track: phase name, status (pass/fail), and file path to output.

### Main Context Anti-Patterns

**NEVER do this:**
```
# BAD: Accumulating outputs in main context
phase1_result = TaskOutput(phase1_id)
phase2_result = TaskOutput(phase2_id)
phase3_result = TaskOutput(phase3_id)
# Now main context has all three outputs bloating it
```

**DO this instead:**
```
# GOOD: Check status, discard details
phase1_result = TaskOutput(phase1_id)
phase1_passed = "COMPLETE" in phase1_result  # Extract only what you need
# Don't store phase1_result, let it go out of scope
```

### Output Size Limits

Phase outputs in `.cache/` MUST be concise:

| Section | Max Size |
|---------|----------|
| Status | 1 line |
| Files Changed | 10 lines max |
| Verification | 5 lines max |
| Exposed Interfaces | 15 lines max |
| Integration Points | 5 lines max |

**Total output: ~40 lines max per phase.** If you need more detail, reference file paths instead of inlining content.

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

context-files:      # Files to read before starting
  - pkg/auth/*.go
  - internal/models/user.go

verification:       # Multiple criteria (all must pass)
  criteria:
    - name: "Unit tests"
      command: "go test ./pkg/auth/..."
      expected: "PASS"
    - name: "Build"
      command: "go build ./..."
      expected: "exit 0"
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
