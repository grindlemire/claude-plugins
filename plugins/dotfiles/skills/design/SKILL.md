---
name: design
description: Refine high-level feature specs or issues into detailed technical designs and implementation plans suitable for AI agent execution. Use when asked to "plan out how to implement," "design a feature," "create an implementation plan," or when given a feature spec/issue that needs architectural decisions, interface definitions, trade-off analysis, and a concrete task breakdown.
---

# Design

Transform feature specifications into technical designs and implementation plans.

Plans are designed to be executed by the `build` skill, which processes phases sequentially while maintaining context from the overall design and previous phases.

## Workflow

1. **Gather context** — Read the feature spec, find relevant codebase context
2. **Clarify ambiguities** — Ask questions before proceeding if requirements are unclear
3. **Produce design** — Architecture, interfaces, trade-offs
4. **Produce implementation plan** — File and function-level tasks for an AI agent

## Step 1: Gather Context

### Read the feature spec

Parse the provided spec/issue for:
- Goal and success criteria
- Constraints (performance, compatibility, dependencies)
- Scope boundaries (what's explicitly in/out)

### Find codebase context

Check for context files in this order:
1. `CLAUDE.md` or `AGENTS.md` at repo root
2. Any `README.md` that describes architecture

If the spec references existing code, locate it:
- Search for mentioned files, functions, types
- Identify integration points
- Note existing patterns the new code should follow

If ambiguity remains after searching, ask the user before proceeding.

## Step 2: Clarify Ambiguities

Before designing, resolve:
- Unclear requirements ("should this handle X case?")
- Missing constraints ("what's the expected scale?")
- Multiple valid approaches ("option A vs B—preference?")

Ask focused questions. Do not proceed with assumptions on critical decisions.

## Step 3: Produce Design

See [references/design-template.md](references/design-template.md) for the full template.

The design section covers:
- **Overview** — One-paragraph summary
- **Architecture** — Components, data flow, module structure
- **Interface definitions** — Key function signatures, types, API contracts
- **Trade-off analysis** — Options considered, rationale for choices
- **Open questions** — Anything deferred or needing future input

Match depth to complexity. A small feature needs less than a system redesign.

## Step 4: Produce Implementation Plan

Break implementation into **phases**. Each phase:
- Ends with something runnable that proves it works
- Has its own markdown file with YAML frontmatter
- Specifies verification steps and output summary for the next phase

See:
- [references/plan-template.md](references/plan-template.md) for the phase template
- [references/manifest-template.yaml](references/manifest-template.yaml) for the manifest format

### Phase design principles

**Each phase must "work"** — After completing a phase, there must be a concrete way to verify success: a test passes, a command produces output, an endpoint returns data, a UI renders. No phase should end in a "trust me, it's wired up" state.

**Phases build incrementally** — Phase 2 assumes Phase 1 is complete and verified. The phase output summary provides context for the next phase.

**Right-size the phases** — A phase should be completable in one focused session. Too large and verification becomes unclear; too small and overhead dominates.

### Task granularity within phases

Target file and function level:

```
- Create `auth/jwt.go`
  - Add `Claims` struct with fields: UserID (string), Exp (time.Time), Roles ([]string)
  - Add `ValidateToken(token string, key *rsa.PublicKey) (Claims, error)`
  - Add `ParseUnverified(token string) (Claims, error)` for inspection without validation
```

Not too coarse:
```
- Implement JWT authentication  # too vague
```

Not too granular:
```
- Add import "crypto/rsa"  # unnecessary detail
```

## Output

Produce multiple files:

```
design-<feature-name>/
├── manifest.yaml              # Phase order, dependencies, status tracking
├── design.md                  # Architecture, interfaces, trade-offs
├── phase-1-<short-name>.md    # First phase tasks + verification
├── phase-2-<short-name>.md    # Second phase tasks + verification
└── ...
```

### manifest.yaml

The manifest tracks phase order, dependencies, and execution status:

```yaml
feature: <feature-name>
created: <ISO timestamp>
design: design.md

phases:
  - file: phase-1-<short-name>.md
    status: pending
    depends-on: []
    started-at: null
    completed-at: null

  - file: phase-2-<short-name>.md
    status: pending
    depends-on: [phase-1-<short-name>.md]
    started-at: null
    completed-at: null
```

Status values: `pending`, `in-progress`, `complete`, `failed`

See [references/manifest-template.yaml](references/manifest-template.yaml) for the full template.

### Phase files

Each phase file includes YAML frontmatter for machine parsing:

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

Each phase file is self-contained and can be handed to the `build` skill independently (given prior phases are complete).

### Execution

After creating a design, run phases with the build skill:

```
/build design-<feature-name>              # Run next pending phase
/build design-<feature-name>/phase-1-*.md # Run specific phase
```
