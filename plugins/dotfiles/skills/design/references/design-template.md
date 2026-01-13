# Design Template

Use this structure for the design section. Adjust depth based on feature complexity.

```markdown
## Design

### Overview

[One paragraph: what this feature does, why it exists, key approach]

### Architecture

[Describe components and how they interact. Include:]

- Module/package structure
- Data flow between components
- External dependencies
- Key abstractions

[For complex features, include a diagram in mermaid or ASCII]

### Interface Definitions

[Define the public contracts. For each significant type/function:]

#### `TypeName`

```go
type TypeName struct {
    FieldA string    // purpose
    FieldB int       // purpose
}
```

#### `FunctionName`

```go
// FunctionName does X.
// Returns Y on success, error if Z.
func FunctionName(param1 Type1, param2 Type2) (ReturnType, error)
```

[Include expected behavior, edge cases, error conditions]

### Trade-off Analysis

[For each significant decision:]

**Decision: [what was decided]**

Options considered:
1. **Option A** — [brief description]
   - Pros: [list]
   - Cons: [list]
2. **Option B** — [brief description]
   - Pros: [list]
   - Cons: [list]

Chosen: **Option [X]** because [rationale].

### Open Questions

[List anything unresolved or deferred:]

- [ ] Question 1 — context/impact
- [ ] Question 2 — context/impact
```
