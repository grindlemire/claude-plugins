# Phase Template

Each phase is a self-contained markdown file. An AI agent should be able to execute a phase given only that file and the output from the previous phase.

**Philosophy:** Phases define WHAT and WHY, not HOW. Give the implementing agent constraints and goals, not line-by-line instructions. The agent is intelligent — trust it to make good implementation decisions within your constraints.

## Frontmatter

Every phase file must include YAML frontmatter for machine parsing:

```yaml
---
phase: N
name: short-name
design: ../design.md
depends-on: [phase-N-1-previous.md]  # Empty array for phase 1
verification:
  command: "go test ./path/..."
  expected: "PASS"
status: pending
---
```

Fields:
- `phase`: Phase number (1-indexed)
- `name`: Short identifier matching filename
- `design`: Relative path to design.md
- `depends-on`: Array of prerequisite phase files
- `verification.command`: Shell command to verify phase completion
- `verification.expected`: Expected output pattern (substring match)
- `status`: `pending` | `in-progress` | `complete` | `failed`

## Body

```markdown
# Phase N: [Short Name]

## Design Context

<!-- Key architectural decisions from design.md relevant to this phase -->
- **Approach**: [summary of chosen approach]
- **Key types**: [types this phase will use or create]
- **Integration point**: [where this connects to existing code]

## Context from Previous Phase

[Copy the "Output for Next Phase" section from the previous phase here. For Phase 1, describe the starting point.]

## Goal

[One sentence: what "working" looks like at the end of this phase]

## Tasks

### Task 1: [Short Description]

**Goal:** [What this task accomplishes]

**Constraints:**
- Must satisfy interface X from design.md
- Must handle error cases A, B, C
- Must be thread-safe / idempotent / etc.

**Suggested location:** `path/to/file.go` (or create new file)

**Notes:** [Any non-obvious requirements or gotchas]

---

### Task 2: [Short Description]

**Goal:** [What this task accomplishes]

**Constraints:**
- Integrate with existing `ExistingFunc` in `existing.go`
- Preserve backward compatibility with X

**Notes:** [Context the agent needs]

---

[Continue for each task in this phase]

## Verification

[Concrete steps to prove this phase works. Must be runnable.]

**Run:**
```bash
go test ./auth/...
```

**Expected output:**
```
ok      myapp/auth    0.015s
```

## Output for Next Phase

[Brief summary for the agent executing the next phase. Include:]

**What was added:**
- `auth/jwt.go` — JWT validation
- `auth/jwt_test.go` — Unit tests

**Key decisions made:**
- [Any implementation choices that affect future phases]

**Integration points for next phase:**
- [Functions/types available for the next phase to use]
```

## Writing Good Tasks

### Focus on Constraints, Not Implementation

**Good (constraint-based):**
```
### Task 1: JWT Token Validation

**Goal:** Validate JWT tokens and extract user claims.

**Constraints:**
- Must implement `TokenValidator` interface from design.md
- Must return distinct errors for: expired, invalid signature, malformed
- Must not panic on malformed input
- Must work with RS256 algorithm (keys from env var)

**Suggested location:** `auth/jwt.go`
```

**Bad (implementation-dictating):**
```
### Task 1: JWT Token Validation

- Create `auth/jwt.go`
- Add `Claims` struct with fields:
  - `UserID string`
  - `Exp time.Time`
  - `Roles []string`
- Add `ValidateToken(token string, key *rsa.PublicKey) (Claims, error)`
  - Call jwt.Parse() with the token
  - Check if claims.ExpiresAt < time.Now()
  - If expired, return ErrExpired
  - ...
```

### When to Be Specific

**Be specific about:**
- Public interfaces (from design.md) that must be satisfied
- Error types that other code depends on
- Integration points with existing code
- Performance or security constraints

**Let the agent decide:**
- Internal struct field names
- Helper function organization
- Implementation approach within constraints
- Import organization

## Phase Sizing Guidelines

**Right-sized phase:**
- 2-5 tasks
- Ends with a runnable verification
- Completable in one session

**Phase too large:**
- 10+ tasks
- Verification is vague ("it should work")
- Multiple independent features bundled

**Phase too small:**
- Single trivial task
- No meaningful verification possible
- Could be combined with adjacent phase
