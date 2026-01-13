---
name: design
description: Transforms feature specs into technical designs and phased implementation plans for AI agent execution. Use when asked to "design a feature," "plan implementation," or given a spec needing architecture, interfaces, and task breakdown.
---

# Design

Transform feature specifications into technical designs and phased implementation plans.

## Contents

- [Workflow](#workflow)
- [Output Structure](#output)
- [Execution](#execution)
- [References](#references)

## Workflow

1. **Gather context** — Read spec, find codebase context
2. **Clarify ambiguities** — Ask before proceeding on unclear requirements
3. **Produce design** — Architecture, interfaces, trade-offs
4. **Produce phases** — File/function-level tasks with verification

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

### Step 4: Produce Phases

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

```
design-<feature-name>/
├── manifest.yaml          # Phase order, dependencies, status
├── design.md              # Architecture, interfaces, trade-offs
├── phase-1-<name>.md      # First phase tasks + verification
├── phase-2-<name>.md      # Second phase
└── ...
```

### manifest.yaml

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

### Phase frontmatter

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
