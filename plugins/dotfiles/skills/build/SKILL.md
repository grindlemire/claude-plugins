---
name: build
description: Execute a single phase from a design plan created by the design skill. Reads the phase file, understands the overall design context, executes tasks, runs verification, and updates status. Use "/build <phase-file>" for a specific phase or "/build <design-dir>" to run the next pending phase.
---

# Build

Execute implementation phases created by the design skill.

## Inputs

Accepts either:
- A specific phase file: `/build design-auth/phase-2-middleware.md`
- A design directory: `/build design-auth` (runs next pending phase)

## Workflow

### Step 1: Load Context

1. **Locate files**
   - If given a directory, read `manifest.yaml` to find the next pending phase
   - If given a phase file, use it directly

2. **Parse phase frontmatter**
   ```yaml
   ---
   phase: 2
   name: middleware
   design: ../design.md
   depends-on: [phase-1-core-types.md]
   verification:
     command: "go test ./auth/..."
     expected: "PASS"
   status: pending
   ---
   ```

3. **Read design.md** — Load the overall architecture, interfaces, and trade-off decisions

4. **Read previous phase output** — If `depends-on` is specified, read the "Output for Next Phase" section from each dependency

5. **Check prerequisites** — Verify all dependent phases have `status: complete` in manifest

### Step 2: Execute Tasks

For each task in the phase:

1. **Read before writing** — If modifying existing files, read them first to understand current state

2. **Follow the plan precisely**
   - Implement exactly what the task specifies
   - Do not add features, refactor surrounding code, or "improve" beyond scope
   - Match existing patterns in the codebase

3. **Track progress** — Use the todo list to track individual tasks within the phase

4. **Handle blockers**
   - If a task is unclear, check the design.md for guidance
   - If still blocked, ask the user before proceeding
   - Do not make assumptions on ambiguous requirements

### Step 3: Verify

1. **Run verification command** from the phase frontmatter or Verification section:
   ```bash
   # From frontmatter
   verification:
     command: "go test ./auth/..."
   ```

2. **Check expected output** — Compare against the expected result in the phase file

3. **Handle failures**
   - Diagnose the root cause
   - Fix the issue
   - Retry verification (up to 3 attempts)
   - If still failing after 3 attempts, report failure with:
     - What was attempted
     - Error output
     - Suspected cause
     - Suggested next steps

### Step 4: Complete Phase

1. **Update phase file** — Change frontmatter status:
   ```yaml
   status: complete
   completed-at: 2025-01-13T10:30:00Z
   ```

2. **Update manifest.yaml** — Mark phase complete with timestamp

3. **Verify "Output for Next Phase"** — Ensure this section accurately reflects what was built. Update if implementation deviated from plan.

4. **Report completion**
   ```
   Phase 2 complete: middleware

   Added:
   - auth/middleware.go — JWT validation middleware
   - auth/middleware_test.go — Integration tests

   Verification: go test ./auth/... PASS

   Ready for: phase-3-endpoints.md
   ```

## Execution Principles

### Follow the plan
The design skill already made architectural decisions. The build skill's job is execution, not design. If the plan says "use Redis," use Redis—don't switch to memcached because it seems simpler.

### Verify before declaring done
A phase is complete only when:
- All tasks are implemented
- Verification command passes
- Output matches expected results

Never mark complete if tests are failing or verification is skipped.

### Preserve context for next phase
The "Output for Next Phase" section is the interface between phases. If you made implementation decisions (e.g., named a function differently than planned), update this section so the next phase has accurate information.

### Stay focused
- Don't fix unrelated bugs you notice
- Don't refactor adjacent code
- Don't add error handling for impossible cases
- Don't add features "while you're in there"

Complete the phase. Nothing more.

## Error Handling

### Prerequisite phase incomplete
```
Error: Phase 2 requires phase-1-core-types.md to be complete.
Current status: pending

Run: /build design-auth/phase-1-core-types.md
```

### Verification failure
```
Verification failed (attempt 2/3):

Command: go test ./auth/...
Output:
  --- FAIL: TestValidateToken (0.00s)
      jwt_test.go:45: expected ErrExpired, got ErrInvalidSignature

Analyzing failure...
[diagnosis and fix attempt]
```

### Ambiguous task
```
Task 3 is ambiguous: "Add error handling for edge cases"

The design.md specifies these error cases:
- Expired tokens → ErrExpired
- Invalid signatures → ErrInvalidSignature

Should I also handle:
- Malformed tokens (not valid JWT structure)?
- Missing tokens (empty string)?

[Asks user for clarification]
```

## Manifest Integration

When a manifest.yaml exists, the build skill:

1. **Reads phase order** from manifest instead of inferring from filenames
2. **Checks dependencies** before starting a phase
3. **Updates status** after completion
4. **Enables `/build <dir>`** to auto-select next pending phase

Example manifest.yaml:
```yaml
feature: user-authentication
created: 2025-01-13T09:00:00Z
design: design.md

phases:
  - file: phase-1-core-types.md
    status: complete
    completed-at: 2025-01-13T09:30:00Z

  - file: phase-2-middleware.md
    status: in-progress
    started-at: 2025-01-13T10:00:00Z
    depends-on: [phase-1-core-types.md]

  - file: phase-3-endpoints.md
    status: pending
    depends-on: [phase-2-middleware.md]
```

## Usage Examples

### Run specific phase
```
/build design-auth/phase-2-middleware.md
```

### Run next pending phase
```
/build design-auth
```

### Check status without running
```
Read the manifest.yaml to see current progress:
cat design-auth/manifest.yaml
```
