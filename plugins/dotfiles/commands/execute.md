---
description: Run all phases of a design plan via subagents
skills: dotfiles:execute
---

# Execute Design Plan via Subagents

**CRITICAL CONSTRAINT: You MUST use the Task tool to spawn subagents. You are FORBIDDEN from implementing code directly in this conversation.**

This skill orchestrates design plan execution by spawning specialized agents. You are the **orchestrator only** — your job is to spawn agents, monitor their output, and coordinate the workflow.

## Mandatory Workflow

### 1. Validate Design
Read and validate `manifest.yaml` and all phase files before proceeding.

### 2. Gather Codebase Context

**Check for cached context first:**
1. Look for `<design-dir>/.cache/explore-context.md`
2. If it exists and is recent (< 1 hour old), skip to Step 3
3. If missing or stale, spawn Explore agent

**Spawn Explore Agent (if no cache):**
```
Task(
  subagent_type: "Explore",
  model: "haiku",
  description: "Gather codebase context for <feature>",
  prompt: """
Explore the codebase to gather context for implementing: <feature from design.md>

Find and summarize:
1. **Existing patterns** — How similar features are implemented, naming conventions
2. **Integration points** — Files that need modification, existing types to extend
3. **Testing patterns** — Test structure, utilities available, mocking patterns
4. **Configuration** — How config is loaded, environment variables

## REQUIRED OUTPUT

You MUST write your findings to a cache file. Use the Write tool to create:

**File:** `<design-dir>/.cache/explore-context.md`

**Format:**
```markdown
# Codebase Context for <feature>

Generated: <ISO timestamp>

## Existing Patterns
<findings>

## Integration Points
<findings>

## Testing Patterns
<findings>

## Configuration
<findings>
```

This file will be read by build agents. Do NOT skip writing this file.
"""
)
```

### 3. For EACH Pending Phase

**Step A: Spawn Build Agent (REQUIRED — DO NOT implement inline)**
```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  description: "Build phase N: <phase-name>",
  prompt: """
Execute a build phase from a design plan. Follow instructions precisely.

## Context Files to Read

Before starting, read these files:
1. `<design-dir>/.cache/explore-context.md` — Codebase patterns and conventions
2. `<design-dir>/design.md` — Architecture and interface definitions
3. `<design-dir>/phase-N-*.md` — The phase to execute

If previous phases exist, also read their "Output for Next Phase" sections from completed phase files.

## Instructions
1. Read all context files listed above
2. Execute each task in the phase file
3. Follow constraints precisely — no extra features or refactoring
4. Match existing codebase patterns from explore-context.md
5. Run the verification command from the phase frontmatter

## Output (REQUIRED)

Write your output to: `<design-dir>/.cache/phase-N-output.md`

**Format:**
```markdown
# Phase N Output: <name>

## Files Changed
- path/to/file.go — description

## Verification
Command: <verification.command>
Result: PASS/FAIL
Output: <actual output>

## Output for Next Phase
<summary of what was built, key decisions, integration points>
```

This file will be read by the review agent and subsequent phases.
"""
)
```

**Step B: Spawn Review Agent (REQUIRED)**
```
Task(
  subagent_type: "code-reviewer",
  model: "opus",
  description: "Review phase N: <phase-name>",
  prompt: """
Review code changes from this build phase.

## Context Files to Read

Before reviewing, read these files:
1. `<design-dir>/.cache/phase-N-output.md` — What the build agent changed
2. `<design-dir>/design.md` — Interface definitions and requirements
3. `<design-dir>/phase-N-*.md` — Tasks that were supposed to be implemented
4. `<design-dir>/.cache/explore-context.md` — Codebase patterns to follow

Then read the actual changed files listed in phase-N-output.md.

## Review Criteria
1. **Design Adherence** — Do types/signatures match design.md exactly?
2. **Correctness** — Edge cases handled? Error handling appropriate?
3. **Code Quality** — Follows codebase patterns from explore-context.md?
4. **Security** — Injection vulnerabilities? Input validation?
5. **Testing** — Tests adequate?

## Output (REQUIRED)

Write your review to: `<design-dir>/.cache/phase-N-review.md`

**Format:**
```markdown
# Phase N Review: <name>

## Status: PASS | CONCERNS

## Design Adherence
Matches | Deviates
<list specific deviations if any>

## Issues
<specific problems with file:line references, or "None">

## Suggestions
<non-blocking improvements, or "None">
```
"""
)
```

**Step C: If Review returns CONCERNS, spawn Fix Agent**
```
Task(
  subagent_type: "general-purpose",
  model: "haiku",
  description: "Fix phase N issues (attempt M/3)",
  prompt: """
Fix the issues identified by the code reviewer.

## Context Files to Read

1. `<design-dir>/.cache/phase-N-review.md` — Issues to fix
2. `<design-dir>/.cache/explore-context.md` — Codebase patterns to follow

## Instructions
1. Read the review file to understand what needs fixing
2. Read each file needing modification
3. Fix ONLY the specific issues listed — no other changes
4. Re-run verification command after fixing

## Output (REQUIRED)

Update: `<design-dir>/.cache/phase-N-output.md`

Append a section:
```markdown
## Fix Attempt M

### Issues Fixed
- <issue>: <how fixed>

### Verification
Command: <command>
Result: PASS/FAIL
```
"""
)
```
Loop: Fix → Re-review (max 3 attempts).

### 4. Update Status
After each phase passes review, update `manifest.yaml` status to `complete`.

## Cache Directory Structure

All agent outputs are persisted in `<design-dir>/.cache/`:

```
design-<feature>/
├── .cache/
│   ├── explore-context.md    # Codebase patterns (from Explore agent)
│   ├── phase-1-output.md     # Build output + fix attempts
│   ├── phase-1-review.md     # Review results
│   ├── phase-2-output.md
│   ├── phase-2-review.md
│   └── ...
├── manifest.yaml
├── design.md
└── phase-*.md
```

**Benefits:**
- Agents read context from files, not inline prompts (more reliable)
- Resume capability — can restart from any point
- Audit trail — see what each agent did
- Subsequent phases can read previous phase outputs

## What You MUST Do
- ✅ Use Task tool for ALL implementation work
- ✅ Pass full context to each subagent (design.md, phase file, previous output)
- ✅ Run Review agent after EVERY Build
- ✅ Run the auto-fix loop when Review returns CONCERNS

## What You MUST NOT Do
- ❌ Write or edit code files directly
- ❌ Skip the Review step
- ❌ Implement "for efficiency" without spawning agents
- ❌ Decide that subagents are unnecessary

**If you find yourself about to write code, STOP. Spawn a Task agent instead.**

## Progress Output

Report progress to the user after each phase:

```
Executing <design-name> (N phases)

Pre-execution:
  Validate: ✓ manifest.yaml, N phases with frontmatter
  Explore:  ✓ Codebase context gathered

Phase 1: <name>
  Build:   PASS (N files)
  Review:  PASS
  Status:  complete

Phase 2: <name>
  Build:   PASS (N files)
  Review:  CONCERNS
  Fix (1/3): ✓ Fixed N issues
  Review:  PASS
  Status:  complete

All phases complete.
```

## Design directory target: $ARGUMENTS
