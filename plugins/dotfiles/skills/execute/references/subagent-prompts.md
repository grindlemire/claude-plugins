# Subagent Prompts

Concise prompts for execution agents. All agents write to `<design-dir>/.cache/`.

## Context Isolation

Each subagent runs in a **clean context window**:
- No conversation history from main context
- No memory of previous phases
- Information flows ONLY through `.cache/` files

**Output size limits** (to keep downstream context lean):
| Section | Max Lines |
|---------|-----------|
| Status | 1 |
| Files Changed | 10 |
| Verification | 5 |
| Exposed Interfaces | 15 |
| Integration Points | 5 |

**Total: ~40 lines max.** Reference file paths instead of inlining large content.

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
If phase has `context-files` in frontmatter, read those files first.
Dependencies: `<design-dir>/.cache/phase-*-output.md` (if any)

Execute tasks. Run ALL verification criteria (may be array).

Write to `.cache/phase-N-output.md` (MUST be ≤40 lines total):

# Phase N Output: <name>
## Status
COMPLETE | BLOCKED: <reason>
## Files Changed (max 10 lines)
- `path/to/file.go` — what was added/changed
## Verification (max 5 lines)
| Criteria | Command | Result |
|----------|---------|--------|
| <name>   | <cmd>   | PASS/FAIL |
## Exposed Interfaces (max 15 lines)
- `FuncName(args) ReturnType` — brief description
- `GET /api/endpoint` — what it returns
## Integration Points (max 5 lines)
How downstream phases should use what was built.

IMPORTANT: Keep output concise. Reference file paths instead of inlining code.
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
