# Parallel Execution

Phases with satisfied dependencies can run in parallel.

## Dependency Analysis

```yaml
# Example manifest
phases:
  - file: phase-1-core.md      # no deps     → Run first
    depends-on: []
  - file: phase-2-api.md       # deps: 1     → After 1
    depends-on: [phase-1-core.md]
  - file: phase-3-ui.md        # deps: 1     → After 1 (parallel with 2!)
    depends-on: [phase-1-core.md]
  - file: phase-4-integration  # deps: 2, 3  → After both
    depends-on: [phase-2-api.md, phase-3-ui.md]
```

Execution order:
1. Phase 1 (sequential)
2. Phase 2 + Phase 3 (parallel)
3. Phase 4 (sequential, after both 2 and 3)

## Parallel Spawn

```python
# Find phases where all dependencies are complete
ready_phases = [p for p in phases if all_deps_complete(p)]

# Spawn all in parallel
for phase in ready_phases:
    Task(
        subagent_type="general-purpose",
        description=f"Build phase: {phase.name}",
        prompt=build_prompt(phase),
        run_in_background=True
    )

# Wait for all to complete
for phase in ready_phases:
    TaskOutput(task_id=phase.task_id, block=True)

# Check all verification results
if all_passed(ready_phases):
    continue_to_next_batch()
else:
    report_failures()
```

## Progress Output

```
Executing design-auth (4 phases)

Phase 1: core
  Build:  PASS
  Review: PASS

Phase 2: api  ┐
  Build:  PASS │ parallel
  Review: PASS │
              │
Phase 3: ui   ┘
  Build:  PASS
  Review: PASS

Phase 4: integration
  Build:  PASS
  Review: PASS

All phases complete.
```

## Failure Handling

If any parallel phase fails, all others in that batch are allowed to complete, then execution pauses:

```
Phase 2: api   → PASS
Phase 3: ui    → FAILED (review concerns)

Phase 2 complete, Phase 3 failed.
Fix Phase 3 before continuing to Phase 4.
```
