# Phase Template

Each phase is a self-contained markdown file. An AI agent should be able to execute a phase given only that file and the output from the previous phase.

```markdown
# Phase N: [Short Name]

## Prerequisites

- Phase N-1 complete
- [Any other prerequisites]

## Context from Previous Phase

[Copy the "Output for Next Phase" section from the previous phase here. For Phase 1, state the starting point.]

## Goal

[One sentence: what "working" looks like at the end of this phase]

## Tasks

### Task 1: [Short Description]

**Files:** `path/to/file.go`

**Changes:**
- Create `TypeName` struct
  - `FieldA string` — purpose
  - `FieldB int` — purpose
- Add `FunctionName(param Type) (Return, error)`
  - Handle case X by doing Y
  - Return error if Z

---

### Task 2: [Short Description]

**Files:** `path/to/file.go`, `path/to/other.go`

**Changes:**
- In `existing.go`, modify `ExistingFunc`:
  - Add call to `NewFunc` after [specific line/condition]
  - Handle new error case

---

[Continue for each task in this phase]

## Verification

[Concrete steps to prove this phase works. Must be runnable.]

**Run:**
```bash
go test ./auth/...
# or
curl http://localhost:8080/health
# or
./bin/myapp --version
```

**Expected output:**
```
PASS: TestValidateToken
PASS: TestExpiredToken
ok      myapp/auth    0.015s
```

**Manual check (if applicable):**
- [ ] [Specific observable behavior]

## Output for Next Phase

[Brief summary for the agent executing the next phase. Include:]

**What was added:**
- `auth/jwt.go` — JWT validation with `ValidateToken` and `ParseUnverified`
- `auth/jwt_test.go` — Unit tests for token validation

**Key decisions made:**
- Using RS256 for signatures (public key in env var `JWT_PUBLIC_KEY`)
- Expired tokens return `ErrExpired`, invalid signatures return `ErrInvalidSignature`

**Integration points for next phase:**
- Call `auth.ValidateToken()` from HTTP middleware
- `Claims` struct available for extracting user context
```

## Granularity Guidelines

**Good task granularity:**
```
- Create `auth/jwt.go`
  - Add `Claims` struct: UserID (string), Exp (time.Time), Roles ([]string)
  - Add `ValidateToken(token string, key *rsa.PublicKey) (Claims, error)`
    - Parse JWT, verify signature, check expiration
    - Return ErrExpired if token is expired
    - Return ErrInvalidSignature if verification fails
```

**Too coarse (agent must make design decisions):**
```
- Implement JWT authentication
```

**Too granular (wastes tokens, slows execution):**
```
- Add import "crypto/rsa" to auth/jwt.go
- Add import "time" to auth/jwt.go
```

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
