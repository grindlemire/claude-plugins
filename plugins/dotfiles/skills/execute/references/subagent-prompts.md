# Subagent Prompts

Concise prompts for execution agents. All agents write to `<design-dir>/.cache/`.

## Build Agent

```
Task(
  subagent_type: "general-purpose",
  model: "<frontmatter model: or sonnet>",
  description: "Build phase N: <name>"
)
```

**Prompt:**
```
Build phase N of design plan.

Read: `<design-dir>/design.md`, `<design-dir>/phase-N-*.md`
Dependencies: `<design-dir>/.cache/phase-*-output.md` (if any)

Execute tasks. Run verification. Write to `.cache/phase-N-output.md`:

# Phase N Output: <name>
## Files Changed
<list>
## Verification
Command: <cmd>, Result: PASS/FAIL
## Output for Next Phase
<summary>
```

## Review Agent

```
Task(
  subagent_type: "code-reviewer",
  model: "sonnet",
  description: "Review phase N: <name>"
)
```

**Prompt:**
```
Review phase N build output.

Read: `.cache/phase-N-output.md`, `design.md`, `phase-N-*.md`
Then read the changed files.

Check: Design adherence, correctness, code quality.
Write to `.cache/phase-N-review.md`:

# Phase N Review: <name>
## Status: PASS | CONCERNS
## Issues
<file:line references or "None">
```

## Verify-Only Review

For phases with `review: verify-only`:

```
Task(
  subagent_type: "general-purpose",
  model: "haiku",
  description: "Verify phase N: <name>"
)
```

**Prompt:**
```
Check phase N verification passed.
Read `.cache/phase-N-output.md`. Confirm verification shows PASS.
Write `.cache/phase-N-review.md` with Status: PASS or CONCERNS.
```

## Fix Agent

```
Task(
  subagent_type: "general-purpose",
  model: "haiku",
  description: "Fix phase N (attempt M/3)"
)
```

**Prompt:**
```
Fix issues from review.

Read: `.cache/phase-N-review.md`
Fix ONLY listed issues. Re-run verification.
Append to `.cache/phase-N-output.md`:

## Fix Attempt M
### Issues Fixed
<list>
### Verification
Result: PASS/FAIL
```
