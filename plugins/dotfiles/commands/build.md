---
description: Execute a single phase from a design plan
skills: dotfiles:build
---

# Build Phase

Execute a single implementation phase in the main conversation.

**This skill runs directly — do NOT spawn subagents for the implementation work.**

For automated multi-phase execution with subagents, use `/execute` instead.

## Workflow

### 1. Load Context
1. Read `manifest.yaml` to find next pending phase (or use specified phase)
2. Parse phase frontmatter for dependencies and verification
3. Read `design.md` for architectural context
4. Read "Output for Next Phase" from completed dependencies

### 2. Execute Tasks

**MUST follow these principles:**

- **Read before writing** — ALWAYS read existing files before modifying
- **Follow plan precisely** — No extra features, refactoring, or "improvements"
- **Match codebase patterns** — Follow existing conventions
- **Use todo list** — Track individual task progress

**If blocked:**
- Check design.md for guidance
- Ask user before proceeding on ambiguity
- Do not assume on critical decisions

### 3. Verify

1. Run verification command from phase frontmatter
2. Compare output to expected result
3. On failure: diagnose, fix, retry (up to 3 attempts)
4. If still failing: report with diagnosis

### 4. Complete

1. Update phase frontmatter: `status: complete`, add `completed-at` timestamp
2. Update manifest.yaml status
3. Ensure "Output for Next Phase" section reflects what was built
4. Report completion summary

## What You MUST Do
- ✅ Read files before editing them
- ✅ Execute ALL tasks in the phase file
- ✅ Run the verification command
- ✅ Update status after completion
- ✅ Stay focused on the phase tasks only

## What You MUST NOT Do
- ❌ Skip reading files before editing
- ❌ Add features not in the phase file
- ❌ Refactor adjacent code
- ❌ Fix unrelated bugs you notice
- ❌ Mark complete before verification passes

## Completion Output Format

```
Phase N complete: <name>

Added:
- path/to/file.go — Description

Modified:
- path/to/existing.go — What changed

Verification: <command> PASS

Ready for: phase-N+1-<name>.md
```

Target: $ARGUMENTS
