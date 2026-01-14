---
description: Create a technical design and phased implementation plan for a feature
skills: dotfiles:design
---

# Design Feature

Create a technical design and phased implementation plan.

## Non-Negotiable Requirements

These are MANDATORY. Failure to follow invalidates the output:

1. **Get design feedback BEFORE creating phases** — Present design.md to user and get explicit approval. Do NOT create phase files until user confirms.

2. **Phase files MUST have YAML frontmatter** — Every phase file requires: `phase`, `name`, `design`, `depends-on`, `verification`, `status`

3. **manifest.yaml is REQUIRED** — Every design must include a manifest tracking phase order, dependencies, and status.

## Workflow

### 1. Gather Context
- Parse spec for goals, constraints, scope
- Check CLAUDE.md, README.md for architecture
- Search for referenced files/types
- Identify integration points

### 2. Clarify Ambiguities
Ask before proceeding on unclear requirements. Do not assume on critical decisions.

### 3. Produce Design
Create `design-<feature>/design.md` with:
- Overview (one paragraph)
- Architecture (components, data flow)
- Interfaces (function signatures, types, API contracts)
- Trade-offs (options considered, rationale)

### 4. Get User Approval (CHECKPOINT)

**STOP HERE. Do NOT proceed without explicit approval.**

Present the design and ask:
> "Here is the proposed design. Please review and let me know if you'd like any changes before I create the implementation phases."

Wait for user confirmation before continuing.

### 5. Create Phases (only after approval)

Create phase files with frontmatter:
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

Create `manifest.yaml`:
```yaml
feature: <name>
created: <ISO timestamp>
design: design.md
phases:
  - file: phase-1-<name>.md
    status: pending
    depends-on: []
```

## Output Structure
```
design-<feature>/
├── manifest.yaml      # REQUIRED
├── design.md          # Architecture
├── phase-1-<name>.md  # With frontmatter
└── phase-2-<name>.md  # With frontmatter
```

Feature specification: $ARGUMENTS
