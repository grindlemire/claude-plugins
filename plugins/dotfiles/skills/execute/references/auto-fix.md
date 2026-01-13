# Auto-Fix Loop

When the code-reviewer finds issues, the execute skill automatically attempts to fix them.

## Flow

```
Review: CONCERNS
    │
    ▼
Fix Agent (attempt 1/3)
    │
    ▼
Re-run Verification
    │
    ▼
Re-run Review ───► PASS? → Continue
    │
    └──► CONCERNS? → Loop (max 3)
              │
              └──► Exhausted? → Manual intervention
```

## Loop Logic

```python
max_attempts = 3
attempt = 0

while attempt < max_attempts:
    review_result = run_code_reviewer()

    if review_result.status == "PASS":
        break

    attempt += 1
    if attempt >= max_attempts:
        report_failure(review_result.issues)
        return

    fix_result = run_fix_agent(review_result.issues)

    if fix_result.verification == "FAIL":
        report_failure(fix_result)
        return

    # Loop back to review
```

## Progress Output

### Successful Auto-Fix

```
Phase 2: middleware
  Build:  PASS (2 files)
  Review: CONCERNS
    - middleware.go:45 — missing nil check
    - middleware.go:67 — error swallowed
  Fix (1/3): Fixing 2 issues...
    ✓ Added nil check at middleware.go:45
    ✓ Added error propagation at middleware.go:67
    Verification: PASS
  Review: PASS
  Status: complete
```

### Auto-Fix Exhausted

```
Phase 2: middleware
  Build:  PASS (2 files)
  Review: CONCERNS (2 issues)
  Fix (1/3): Applied fixes, verification PASS
  Review: CONCERNS (1 remaining)
  Fix (2/3): Applied fix, verification PASS
  Review: CONCERNS (1 different issue)
  Fix (3/3): Applied fix, verification PASS
  Review: CONCERNS (1 persists)
  Status: FAILED (auto-fix exhausted)

Remaining issues:
  - middleware.go:52 — race condition in token cache

Manual intervention required:
  /build design-auth/phase-2-middleware.md
```

## Disabling Auto-Fix

Use `--no-auto-fix` to pause immediately on review issues:

```bash
/execute design-auth --no-auto-fix
```

This outputs:

```
Phase 2 paused: middleware (review concerns)

Issues found:
  - middleware.go:45 — missing nil check

Options:
1. Fix manually, then: /execute design-auth --resume
2. Enable auto-fix: /execute design-auth --resume
3. Debug interactively: /build design-auth/phase-2-middleware.md
```
