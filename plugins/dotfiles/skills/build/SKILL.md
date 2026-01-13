---
name: build
description: Executes a single phase from a design plan in the main conversation. Reads phase file and design context, executes tasks, runs verification, updates status. Use "/build <design-dir>" for next pending phase or "/build <phase-file>" for a specific phase.
---

# Build

Execute a single implementation phase in the main conversation.

For automated multi-phase execution, use `/execute` instead.

## Usage

```bash
/build design-auth              # Run next pending phase
/build design-auth/phase-2-*.md # Run specific phase
```

## Workflow

### 1. Load Context

1. Read `manifest.yaml` to find next pending phase (or use specified phase)
2. Parse phase frontmatter for dependencies and verification
3. Read `design.md` for architectural context
4. Read "Output for Next Phase" from dependency phases

### 2. Execute Tasks

For each task:
1. **Read before writing** — Understand existing files before modifying
2. **Follow plan precisely** — No extra features, refactoring, or "improvements"
3. **Match codebase patterns** — Follow existing conventions
4. **Track progress** — Use todo list for individual tasks

If blocked:
- Check design.md for guidance
- Ask user before proceeding on ambiguity
- Do not assume on critical decisions

### 3. Verify

1. Run verification command from phase frontmatter
2. Compare output to expected result
3. On failure: diagnose, fix, retry (up to 3 attempts)
4. If still failing: report with diagnosis and suggested fixes

### 4. Complete

1. Update phase frontmatter: `status: complete`
2. Update manifest.yaml with timestamp
3. Verify "Output for Next Phase" reflects what was built
4. Report completion summary

## Principles

**Follow the plan** — Design decisions are made. Execute, don't redesign.

**Verify before done** — Phase complete only when verification passes.

**Preserve context** — Update "Output for Next Phase" if implementation deviated.

**Stay focused** — Don't fix unrelated bugs, refactor adjacent code, or add features.

## Completion Output

```
Phase 2 complete: middleware

Added:
- auth/middleware.go — JWT validation
- auth/middleware_test.go — Tests

Verification: go test ./auth/... PASS

Ready for: phase-3-endpoints.md
```

## Error Handling

**Prerequisite incomplete:**
```
Error: Phase 2 requires phase-1-core-types.md to be complete.
Run: /build design-auth/phase-1-core-types.md
```

**Verification failure:**
```
Verification failed (attempt 2/3):
Command: go test ./auth/...
Output: FAIL TestValidateToken

Diagnosing...
```

**Ambiguous task:**
```
Task 3 ambiguous: "Add error handling for edge cases"

Design.md specifies: ErrExpired, ErrInvalidSignature

Should I also handle malformed tokens?
[Asks user]
```
