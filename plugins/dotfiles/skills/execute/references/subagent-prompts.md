# Subagent Prompts

Prompt templates for each subagent type spawned during execution.

**Key principle:** All agents read from and write to `<design-dir>/.cache/` files. This ensures reliable context passing and enables resume capability.

## Explore Agent

```
Task(
  subagent_type: "Explore",
  model: "haiku",
  description: "Gather codebase context for <feature>"
)
```

**Prompt:**

```
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
```

## Build Agent

```
Task(
  subagent_type: "general-purpose",
  model: "sonnet",
  description: "Build phase N: <phase-name>"
)
```

**Prompt:**

```
Execute a build phase from a design plan. Follow instructions precisely.

## Context Files to Read

Before starting, read these files:
1. `<design-dir>/.cache/explore-context.md` — Codebase patterns and conventions
2. `<design-dir>/design.md` — Architecture and interface definitions
3. `<design-dir>/phase-N-*.md` — The phase to execute

If previous phases exist, also read their output files: `<design-dir>/.cache/phase-*-output.md`

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
```

## Review Agent

```
Task(
  subagent_type: "code-reviewer",
  model: "opus",
  description: "Review phase N: <phase-name>"
)
```

**Prompt:**

```
Review code changes from this build phase.

## Context Files to Read

Before reviewing, read these files:
1. `<design-dir>/.cache/phase-N-output.md` — What the build agent changed
2. `<design-dir>/design.md` — Interface definitions and requirements
3. `<design-dir>/phase-N-*.md` — Tasks that were supposed to be implemented
4. `<design-dir>/.cache/explore-context.md` — Codebase patterns to follow

Then read the actual changed files listed in phase-N-output.md.

## Review Criteria

1. **Design Adherence** (highest priority)
   - Do implemented types match design.md interface definitions?
   - Are all specified behaviors implemented?
   - Were any unplanned features added? (flag as CONCERN)

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
```

## Fix Agent

```
Task(
  subagent_type: "general-purpose",
  model: "haiku",
  description: "Fix phase N issues (attempt M/3)"
)
```

**Prompt:**

```
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
```
