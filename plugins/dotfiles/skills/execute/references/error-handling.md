# Error Handling

Common error scenarios and recovery options.

## Build Failure

Verification command fails after build.

```
Phase 2 failed: middleware (build)

Verification: go test ./auth/...
Output:
  --- FAIL: TestValidateToken (0.00s)
      jwt_test.go:45: expected ErrExpired, got nil

Diagnosis:
  Token expiration check not being called...

Options:
1. Fix manually: /execute design-auth --resume
2. Debug interactively: /build design-auth/phase-2-middleware.md
```

## Review Failure (auto-fix exhausted)

Code-reviewer still finds issues after 3 fix attempts.

```
Phase 2 failed: middleware (auto-fix exhausted)

Attempts: 3/3

Attempt history:
  1. Fixed nil check → new issue (error handling)
  2. Fixed error handling → new issue (race condition)
  3. Fixed race condition → same issue persists

Remaining:
  - middleware.go:52 — race condition in token cache

Options:
1. Fix manually: /execute design-auth --resume
2. Debug: /build design-auth/phase-2-middleware.md
```

## Dependency Not Met

Phase cannot run because dependencies are incomplete.

```
Cannot execute phase-3-integration.md

Required dependencies not complete:
  - phase-2-api.md (status: failed)

Fix phase-2 first, then retry.
```

## Manifest Missing

Design directory has no manifest.yaml.

```
No manifest.yaml found in design-auth/

This design may predate manifest support.

Options:
1. Run phases manually: /build design-auth/phase-1-*.md
2. Create manifest.yaml (see design skill references)
```

## Resume After Failure

```bash
/execute design-auth --resume
```

This will:
1. Skip phases marked `complete`
2. Retry phases marked `failed`
3. Continue with remaining `pending` phases
