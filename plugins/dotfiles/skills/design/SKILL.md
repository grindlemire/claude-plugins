---
name: design
description: Transforms feature specs into technical designs and phased implementation plans for AI agent execution. Use when asked to "design a feature," "plan implementation," or given a spec needing architecture, interfaces, and task breakdown.
---

# Design

Transform feature specifications into technical designs and phased implementation plans.

## Contents

- [Non-Negotiable Requirements](#non-negotiable-requirements)
- [Workflow](#workflow)
- [Output Structure](#output)
- [Execution](#execution)
- [References](#references)

## Non-Negotiable Requirements

These requirements are **mandatory** and must never be skipped:

1. **Get design feedback before phases** — Present the design to the user and receive explicit approval before creating any phase files or manifest. Do NOT proceed without user confirmation.

2. **Phase files MUST have YAML frontmatter** — Every phase file must include the frontmatter block with `phase`, `name`, `design`, `depends-on`, `verification`, and `status` fields. Phase files without frontmatter are invalid.

3. **Manifest file is REQUIRED** — Every design must include a `manifest.yaml` file. The manifest tracks phase order, dependencies, and status. A design directory without a manifest is incomplete.

Failure to follow these requirements invalidates the design output.

## Workflow

1. **Gather context** — Read spec, find codebase context
2. **Clarify ambiguities** — Ask before proceeding on unclear requirements
3. **Produce design** — Architecture, interfaces, trade-offs
4. **Get design feedback** — Present design to user, incorporate feedback
5. **Produce phases** — File/function-level tasks with verification

### Step 1: Gather Context

Parse the spec for:
- Goal and success criteria
- Constraints (performance, compatibility)
- Scope boundaries

Find codebase context:
1. Check `CLAUDE.md` or `AGENTS.md` at repo root
2. Check `README.md` for architecture
3. Search for referenced files, functions, types
4. Identify integration points and existing patterns
5. Check if `./designs` directory exists (determines output location)

### Step 2: Clarify Ambiguities

Before designing, resolve:
- Unclear requirements
- Missing constraints
- Multiple valid approaches

Ask focused questions. Do not assume on critical decisions.

### Step 3: Produce Design

See [references/design-template.md](references/design-template.md).

Covers:
- **Overview** — One-paragraph summary
- **Architecture** — Components, data flow, modules
- **Interfaces** — Function signatures, types, API contracts
- **Trade-offs** — Options considered, rationale
- **Open questions** — Deferred items

Match depth to complexity.

### Step 4: Get Design Feedback

**This step is non-negotiable.** Before creating any phase files or the manifest:

1. **Present the design** — Share the complete design.md with the user
2. **Request explicit feedback** — Ask if they approve the design or want changes
3. **Iterate until approved** — Make requested changes and re-present
4. **Only proceed after approval** — Do NOT create phases until user confirms

Example prompt:
> "Here is the proposed design. Please review and let me know if you'd like any changes before I create the implementation phases."

This checkpoint prevents wasted effort building phases for a design the user doesn't want.

### Step 5: Produce Phases

See [references/plan-template.md](references/plan-template.md).

Each phase:
- Ends with runnable verification
- Has YAML frontmatter for machine parsing
- Specifies output for next phase

**Phase principles:**
- Each phase must "work" — concrete verification, not "trust me"
- Phases build incrementally — N+1 assumes N complete
- Right-size phases — completable in one session

**Task granularity** — Target file and function level:

```
- Create `auth/jwt.go`
  - Add `Claims` struct: UserID, Exp, Roles
  - Add `ValidateToken(token, key) (Claims, error)`
```

Not too coarse (`Implement JWT auth`) or too granular (`Add import`).

## Output

### Output Location

**Before creating any files, determine where to put the design:**

1. Check if `./designs` directory exists
2. If yes: create design at `./designs/design-<feature>/`
3. If no: create design at `./design-<feature>/` (current directory)

### Directory Structure

```
[designs/]design-<feature-name>/
├── manifest.yaml          # REQUIRED: Phase order, dependencies, status
├── design.md              # Architecture, interfaces, trade-offs
├── phase-1-<name>.md      # First phase tasks + verification (MUST have frontmatter)
├── phase-2-<name>.md      # Second phase (MUST have frontmatter)
└── ...
```

### manifest.yaml (REQUIRED)

**Every design MUST have a manifest.yaml file.** A design without a manifest is incomplete and invalid.

See [references/manifest-template.yaml](references/manifest-template.yaml).

```yaml
feature: <name>
created: <ISO timestamp>
design: design.md
phases:
  - file: phase-1-<name>.md
    status: pending
    depends-on: []
  - file: phase-2-<name>.md
    status: pending
    depends-on: [phase-1-<name>.md]
```

Status: `pending` | `in-progress` | `complete` | `failed`

### Phase frontmatter (REQUIRED)

**Every phase file MUST include YAML frontmatter.** Phase files without frontmatter are invalid and will cause execution failures.

```yaml
---
phase: 1
name: short-name
design: ../design.md
depends-on: []
verification:
  command: "go test ./path/..."
  expected: "PASS"
status: pending
---
```

## Execution

**Automated** (recommended):
```bash
/execute design-<feature>          # Run all phases via subagents
/execute design-<feature> --resume # Resume after fix
```

**Manual**:
```bash
/build design-<feature>            # Run next pending phase
/build design-<feature>/phase-1-*  # Run specific phase
```

## References

- [references/design-template.md](references/design-template.md) — Design document structure
- [references/plan-template.md](references/plan-template.md) — Phase file structure
- [references/manifest-template.yaml](references/manifest-template.yaml) — Manifest format
