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

### 2. Spawn Explore Agent (REQUIRED)
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

Provide a concise summary (under 500 words) for build agents to follow existing conventions.
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

## Codebase Context
<output from Explore agent>

## Design Context
<key decisions from design.md relevant to this phase>

## Previous Phase Output
<"Output for Next Phase" from dependency, or "Starting point" for phase 1>

## Phase to Execute
<full contents of phase-N-*.md including frontmatter>

## Instructions
1. Execute each task in the Tasks section
2. Follow the plan precisely — no extra features or refactoring
3. Match existing codebase patterns from Codebase Context
4. Run the verification command from the Phase Frontmatter
5. Compare output against verification.expected

## Output (REQUIRED)
1. Files added/changed (with paths)
2. Verification result: PASS/FAIL with output
3. Deviations from plan (if any)
4. "Output for Next Phase" section — summarize what was built
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

## Files Changed
<list from build agent output>

## Design Requirements
<relevant section from design.md — interface definitions, types>

## Phase Tasks
<tasks from the phase file that were supposed to be implemented>

## Review Criteria
1. **Design Adherence** — Do types/signatures match design.md exactly?
2. **Correctness** — Edge cases handled? Error handling appropriate?
3. **Code Quality** — Follows codebase patterns? Readable?
4. **Security** — Injection vulnerabilities? Input validation?
5. **Testing** — Tests adequate?

## Output
1. **Status**: PASS or CONCERNS
2. **Design Adherence**: Matches / Deviates (list specific deviations)
3. **Issues** (if any): Specific problems with file:line references
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

## Issues to Fix
<list from code-reviewer with file:line references>

## Instructions
1. Read each file needing modification
2. Fix ONLY the specific issues listed — no other changes
3. Re-run verification command after fixing

## Output
1. Fixes applied (each issue and how fixed)
2. Verification result: PASS/FAIL
"""
)
```
Loop: Fix → Re-review (max 3 attempts).

### 4. Update Status
After each phase passes review, update `manifest.yaml` status to `complete`.

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
