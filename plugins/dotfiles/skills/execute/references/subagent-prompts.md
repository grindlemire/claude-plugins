# Subagent Prompts

Prompt templates for each subagent type spawned during execution.

## Explore Agent

```
Task(
  subagent_type: "Explore",
  description: "Gather codebase context for <feature>"
)
```

**Prompt:**

```
Explore the codebase to gather context for implementing: <feature from design.md>

Find and summarize:

1. **Existing patterns** — How similar features are implemented, code organization, naming conventions

2. **Integration points** — Files that need modification, existing types/functions to extend

3. **Testing patterns** — Test structure, utilities available, mocking patterns

4. **Configuration** — How config is loaded, environment variables, feature flags

Provide a concise summary (under 500 words) for build agents to follow existing conventions.
```

## Build Agent

```
Task(
  subagent_type: "general-purpose",
  description: "Build phase N: <phase-name>"
)
```

**Prompt:**

```
Execute a build phase from a design plan. Follow instructions precisely.

## Codebase Context
<output from Explore agent>

## Design Context
<key decisions from design.md relevant to this phase>

## Previous Phase Output
<"Output for Next Phase" from dependency, or "Starting point" for phase 1>

## Phase Frontmatter
<YAML frontmatter from phase file — contains verification command>

## Phase to Execute
<full contents of phase-N-*.md>

## Instructions
1. Execute each task in the Tasks section
2. Follow the plan precisely — no extra features or refactoring
3. Match existing codebase patterns from Codebase Context
4. Run the verification command from the Phase Frontmatter (verification.command)
5. Compare output against verification.expected from frontmatter

## Output
1. Files added/changed (with paths)
2. Verification result: Run `<verification.command>`, compare to `<verification.expected>`
3. Deviations from plan (if any)
4. "Output for Next Phase" section — REQUIRED, summarize what was built for next phase
```

## Review Agent

```
Task(
  subagent_type: "code-reviewer",
  description: "Review phase N: <phase-name>"
)
```

**Prompt:**

```
Review code changes from this build phase.

## Files Changed
<list from build agent output>

## Design Requirements
<relevant section from design.md — interface definitions, types, function signatures>

## Phase Tasks
<tasks from the phase file that were supposed to be implemented>

## Review Criteria

1. **Design Adherence** (highest priority)
   - Do implemented types match design.md interface definitions exactly?
   - Do function signatures match what design.md specified?
   - Are all specified behaviors implemented?
   - Were any unplanned features added? (flag as CONCERN)

2. **Correctness** — Edge cases handled? Error handling appropriate?

3. **Code Quality** — Follows codebase patterns? Readable? Obvious bugs?

4. **Security** — Injection vulnerabilities? Input validation? Secrets handled?

5. **Testing** — Tests adequate? Edge cases covered?

6. **Output Validation** — Does the build output include "Output for Next Phase"?

## Output
1. **Status**: PASS or CONCERNS
2. **Design Adherence**: Matches / Deviates (list specific deviations)
3. **Issues** (if any): Specific problems with file:line references
4. **Suggestions** (optional): Non-blocking improvements
```

## Fix Agent

```
Task(
  subagent_type: "general-purpose",
  description: "Fix phase N issues (attempt M/3)"
)
```

**Prompt:**

```
Fix the issues identified by the code reviewer.

## Issues to Fix
<list from code-reviewer with file:line references>

## Files to Modify
<files that need changes>

## Codebase Context
<patterns to follow from Explore agent>

## Instructions
1. Read each file needing modification
2. Fix ONLY the specific issues listed — no other changes
3. Follow existing codebase patterns
4. Re-run verification command after fixing

## Output
1. **Fixes applied**: Each issue and how it was fixed
2. **Verification result**: PASS/FAIL
3. **Files modified**: List with paths
```
