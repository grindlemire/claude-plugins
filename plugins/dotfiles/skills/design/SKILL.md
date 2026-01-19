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

1. **Ask clarifying questions BEFORE designing** — After gathering context, you MUST ask the user a series of clarifying questions before producing any design. These questions should cover feature scope, ambiguities, edge cases, and proposed architecture. Do NOT start writing the design until questions are answered.

2. **Get design feedback before phases** — Present the design to the user and receive explicit approval before creating any phase files or manifest. Do NOT proceed without user confirmation.

3. **Phase files MUST have YAML frontmatter** — Every phase file must include the frontmatter block with `phase`, `name`, `design`, `depends-on`, `verification`, and `status` fields. Phase files without frontmatter are invalid.

4. **Manifest file is REQUIRED** — Every design must include a `manifest.yaml` file. The manifest tracks phase order, dependencies, and status. A design directory without a manifest is incomplete.

Failure to follow these requirements invalidates the design output.

## Workflow

1. **Gather context** — Read spec, find codebase context
2. **Ask clarifying questions** — REQUIRED: Ask user about scope, ambiguities, and architecture before designing
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

### Step 2: Ask Clarifying Questions

**This step is non-negotiable.** Before producing any design, you MUST ask the user clarifying questions. Present all questions at once in a structured format.

**Questions must cover:**

1. **Feature scope & boundaries**
   - What exactly should this feature do?
   - What should it explicitly NOT do?
   - What are the success criteria?

2. **Ambiguity resolution**
   - Any unclear requirements from the spec
   - Edge cases that need decisions
   - Error handling expectations

3. **Architecture & approach**
   - Propose 2-3 architectural approaches with trade-offs
   - Ask which approach the user prefers
   - Identify integration points with existing code

4. **Constraints & preferences**
   - Performance requirements
   - Compatibility constraints
   - Technology/library preferences

**Example question format:**
```
Before I create the design, I have some questions:

**Scope:**
1. Should X also handle Y, or is that out of scope?
2. What should happen when Z occurs?

**Architecture:**
I see two main approaches:
- **Option A:** [description] — Pros: faster. Cons: more complex.
- **Option B:** [description] — Pros: simpler. Cons: slower.
Which approach do you prefer?

**Constraints:**
3. Are there performance requirements I should know about?
4. Should this integrate with [existing system] or be standalone?
```

**Do NOT proceed to Step 3 until questions are answered.**

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

**Task philosophy** — Define constraints, not implementation:

Tasks should specify WHAT and WHY, not HOW. Give the implementing agent goals and constraints, trust it to make good implementation decisions.

```
### Task: JWT Token Validation

**Goal:** Validate JWT tokens and extract user claims.

**Constraints:**
- Must implement `TokenValidator` interface from design.md
- Must return distinct errors for: expired, invalid signature, malformed
- Must work with RS256 algorithm

**Suggested location:** `auth/jwt.go`
```

**Be specific about:** Interfaces to satisfy, error types, integration points, security constraints.

**Let the agent decide:** Internal struct fields, helper organization, implementation details.

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
context-files:              # Optional: files agent should read first
  - pkg/auth/*.go
verification:
  command: "go test ./path/..."
  expected: "PASS"
status: pending
---
```

For phases requiring multiple verification checks:

```yaml
verification:
  criteria:
    - name: "Unit tests"
      command: "go test ./pkg/auth/..."
      expected: "PASS"
    - name: "Build"
      command: "go build ./..."
      expected: "exit 0"
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
