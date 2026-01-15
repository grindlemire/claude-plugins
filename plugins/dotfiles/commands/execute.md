---
description: Run all phases of a design plan via subagents
skills: dotfiles:execute
---

# Execute Design Plan via Subagents

**CRITICAL: You MUST use the Task tool to spawn subagents. NEVER implement code directly.**

You are the **orchestrator only** — spawn agents, monitor output, coordinate workflow.

## Workflow

### 1. Validate
Read `manifest.yaml` and all phase files. Abort if invalid.

### 2. Build Dependency Graph
Group phases into waves based on `depends-on`:

```
Wave 1: [phase-1, phase-2, phase-3, phase-4]  # No dependencies
Wave 2: [phase-5, phase-6]                     # Depend on wave 1
Wave 3: [phase-7]                              # Depends on phases 1, 3
```

### 3. Execute Waves (PARALLEL)

For each wave, spawn **all build agents in a single message** with multiple Task tool calls:

```
Task(subagent_type: "general-purpose", model: "<from frontmatter or sonnet>", ...)
Task(subagent_type: "general-purpose", model: "<from frontmatter or sonnet>", ...)
Task(subagent_type: "general-purpose", model: "<from frontmatter or sonnet>", ...)
```

**Build Agent Prompt:**
```
Execute phase N from the design plan.

Read: `<design-dir>/design.md`, `<design-dir>/phase-N-*.md`
If dependencies exist, read their `.cache/phase-*-output.md` files.

Execute tasks precisely. Run verification. Write output to `.cache/phase-N-output.md`.
```

**Review** — Check phase frontmatter:
- `review: skip` → Skip review, trust verification
- `review: verify-only` → Just confirm verification passed (use haiku)
- Default → Full review with `code-reviewer` agent (sonnet)

Spawn reviews in parallel after builds complete.

**Fix Loop** — If CONCERNS, spawn fix agent (haiku), re-review. Max 3 attempts.

### 4. Update Status
After wave completes, update `manifest.yaml` for all phases in wave.

## Phase Frontmatter Options

```yaml
model: haiku           # Use haiku for build (default: sonnet)
review: skip           # Skip review entirely
review: verify-only    # Just check verification passed
```

## Cache Structure

All agent outputs go to `<design-dir>/.cache/`:

| Agent | Writes |
|-------|--------|
| Build | `.cache/phase-N-output.md` |
| Review | `.cache/phase-N-review.md` |
| Fix | Appends to `.cache/phase-N-output.md` |

## Rules

**MUST:**
- ✅ Spawn parallel Task calls for independent phases (single message, multiple tools)
- ✅ Respect `model:` and `review:` frontmatter
- ✅ Write outputs to `.cache/`

**MUST NOT:**
- ❌ Write code directly — always spawn agents
- ❌ Run phases sequentially when they can be parallel
- ❌ Use opus for reviews (too expensive) — use sonnet

## Progress Output

```
Executing design-auth (8 phases, 4 waves)

Wave 1 (parallel): 1,2,3,4
  Phase 1: Build PASS, Review: skip  [haiku]
  Phase 2: Build PASS, Review: skip  [haiku]
  Phase 3: Build PASS, Review: skip  [haiku]
  Phase 4: Build PASS, Review: skip  [haiku]

Wave 2 (parallel): 5,6
  Phase 5: Build PASS, Review: PASS  [sonnet]
  Phase 6: Build PASS, Review: PASS  [sonnet]

All phases complete.
```

## Design directory target: $ARGUMENTS
