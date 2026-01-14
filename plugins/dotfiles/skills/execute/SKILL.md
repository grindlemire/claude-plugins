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
- [Performance](#performance)
- [References](#references)

## Workflow

```
/execute design-auth
         │
         ▼
┌─────────────────────────┐
│  Validate               │  ← Check manifest + frontmatter
└───────────┬─────────────┘
            │ (abort if invalid)
┌───────────▼─────────────┐
│  Explore Agent          │  ← Gather codebase patterns (cached)
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

| Agent | Type | Model | Purpose |
|-------|------|-------|---------|
| Explore | `Explore` | haiku | Fast codebase reconnaissance before execution |
| Build | `general-purpose` | sonnet | Execute phase tasks |
| Review | `code-reviewer` | opus | Validate code quality and design adherence |
| Fix | `general-purpose` | haiku | Auto-fix specific review issues |

### Model Selection for Performance

- **Explore**: Use `haiku` — pattern matching and file discovery is fast and doesn't need deep reasoning
- **Build**: Use `sonnet` — implementing code requires understanding context and making decisions
- **Review**: Use `opus` — design adherence checks require deep reasoning to compare implementation against spec
- **Fix**: Use `haiku` for simple fixes (1-2 issues), `sonnet` for complex fixes (3+ issues or architectural)

## Execution Steps

### 1. Pre-execution: Validate

**This step is mandatory.** Before any work begins, validate the design structure:

1. **Manifest exists** — `manifest.yaml` must be present and valid YAML
2. **Phase frontmatter** — Every phase file must have required frontmatter:
   - `phase`, `name`, `design`, `depends-on`, `verification`, `status`
3. **Dependencies valid** — All `depends-on` references point to existing phase files
4. **Verification defined** — Each phase has `verification.command` and `verification.expected`

If validation fails, report specific errors and abort:
```
Validation failed for design-auth:

  ✗ phase-2-middleware.md: missing frontmatter field 'verification'
  ✗ phase-3-endpoints.md: depends-on 'phase-2-api.md' not found

Fix these issues before execution.
```

### 2. Pre-execution: Explore

Spawn `Explore` agent to gather codebase context:
- Existing patterns and conventions
- Integration points
- Testing patterns

This context is passed to all subsequent build agents.

**Caching**: When resuming (`--resume`), reuse cached Explore output if the codebase hasn't changed significantly (no new files in relevant directories).

### 3. Load Design

1. Read `manifest.yaml` for phase order and status
2. Read `design.md` for architectural context
3. Identify phases with `status: pending`

### 4. Per-Phase Execution

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

**Validate Output** → Before marking complete:
- Verify build output includes "Output for Next Phase" section
- If missing, prompt build agent to provide it
- This is required for subsequent phases to have context

**Update** → On success:
- Update manifest status to `complete`
- Update phase frontmatter with `completed-at` timestamp
- Store "Output for Next Phase" for next build agent
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
  Validate: ✓ manifest.yaml, 3 phases with frontmatter
  Explore:  Complete (cached)

Phase 1: core-types
  Build:   PASS (3 files)
  Review:  CONCERNS
  Fix (1/3): ✓ Fixed 2 issues
  Review:  PASS
  Output:  ✓ "Output for Next Phase" captured

Phase 2: middleware
  Build:   PASS (2 files)
  Review:  PASS
  Output:  ✓ "Output for Next Phase" captured

All phases complete.
```

## Performance

### Explore Caching

The Explore agent's output can be cached and reused:

```
design-<feature>/
├── .cache/
│   └── explore-context.md    # Cached Explore output
│   └── explore-hash.txt      # Hash of relevant directories
```

**When to reuse cache:**
- `--resume` flag is used
- No new files added to directories mentioned in design.md
- Cache is less than 1 hour old

**When to refresh:**
- First execution
- Significant time elapsed
- New files detected in relevant paths

### Fast Mode

Use `--skip-review` for trusted internal designs where speed matters:

```bash
/execute design-auth --skip-review
```

This skips the Review and Fix loop entirely, relying only on verification commands.

## References

For detailed information:

- **Subagent prompts**: See [references/subagent-prompts.md](references/subagent-prompts.md)
- **Auto-fix loop**: See [references/auto-fix.md](references/auto-fix.md)
- **Parallel execution**: See [references/parallel-execution.md](references/parallel-execution.md)
- **Error handling**: See [references/error-handling.md](references/error-handling.md)
