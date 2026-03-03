# Auto-Todo Decision Rules Reference

## 1. FR-to-Task Granularity Classifier (FR-002)

### Decision Tree

```
For each FR in the parsed requirement document:

Step 1: Count ACs
  - No ACs → infer from FR description length, tag [NO AC - INFERRED]
  - Has ACs → use actual count

Step 2: Check for depth artifacts
  - Decision trees, state machines, computation specs, impact matrices
  - Yes = has depth artifacts

Step 3: Apply classifier
```

| AC Count | Depth Artifacts? | Same-CD Small FRs? | Decision | Action |
|----------|-----------------|---------------------|----------|--------|
| ≤2 | No | Yes | **MERGE** | Combine with other ≤2 AC FRs in same CD that share priority |
| ≤2 | No | No | **PASS-THROUGH** | 1 FR → 1 Task |
| 3-5 | No | — | **PASS-THROUGH** | 1 FR → 1 Task |
| 3-5 | Yes | — | **SPLIT** | Split by artifact boundary (each artifact gets its own task) |
| >5 | No | — | **SPLIT** | Split by AC groups (cluster related ACs into 2-3 tasks) |
| >5 | Yes | — | **SPLIT** | Split by concern (separate data/logic/UI/integration) |

### Merge Rules

When merging FRs into a single task:
- Only merge FRs within the **same CD** (Capability Domain)
- Only merge FRs with the **same MoSCoW priority**
- Merged task title: descriptive name covering all source FRs
- Merged task `traces_to`: list all source FR IDs
- Merged task ACs: union of all source FRs' ACs
- Max 4 FRs per merge (beyond this, keep separate)

### Split Rules

When splitting an FR into multiple tasks:
- Each subtask must have **clear boundaries** (what's in vs. out)
- Each subtask must be **independently testable**
- Each subtask gets its own AC subset
- Subtask IDs: append suffix (e.g., FR-005 → T-05a, T-05b)
- All subtasks share `traces_to: FR-005`
- **Max 1 additional split round** — if a subtask still seems too large, leave it and tag `[LARGE - consider manual split]`

### Granularity Presets

User can select:
- **Fine**: Bias toward splitting. Tasks target 2-4 hours. MERGE threshold: ≤1 AC.
- **Standard** (default): Balanced. Tasks target 2-8 hours. As per table above.
- **Coarse**: Bias toward merging. Tasks target 4-12 hours. MERGE threshold: ≤4 AC.

---

## 2. Dependency Rewrite Rules (FR-004)

When FRs have `depends_on` relationships, these must be translated to task-level dependencies after merge/split operations.

### Impact Matrix

| FR Relationship | Task Mapping | Dependency Result |
|-----------------|-------------|-------------------|
| A depends_on B | Both pass-through | Task-A depends_on Task-B |
| A depends_on B | A split → T-A1, T-A2 | T-A1 (earliest subtask) inherits the dependency on Task-B |
| A depends_on B | B split → T-B1, T-B2 | Task-A depends on T-B2 (latest subtask — all must complete) |
| A depends_on B | Both merged into T-Z | Dependency eliminated (internal to merged task) |
| A depends_on B | A merged with C into T-AC | T-AC inherits A's dependency on Task-B |
| A depends_on B | B merged with D into T-BD | Task-A depends on T-BD |

### Cycle Detection

After all dependency rewrites:
1. Build directed graph: nodes = tasks, edges = depends_on
2. Run DFS-based cycle detection
3. If cycle found → **abort** with error message:
   ```
   ❌ Circular dependency detected: T-03 → T-07 → T-12 → T-03
   Please review the requirement dependencies or modify the task breakdown.
   ```

### Implicit Dependencies

Beyond explicit `depends_on`, detect implicit dependencies:
- Task modifies a file that another task reads → ordering dependency
- Task creates an API that another task consumes → ordering dependency
- Tag implicit dependencies as `[IMPLICIT]` for user awareness

---

## 3. Task Complexity Scoring (FR-009)

### Formula

```
score = AC_count × 1.0
      + depth_artifact_count × 3.0
      + dependency_fan_out × 0.5
```

Where:
- `AC_count`: number of acceptance criteria in the task
- `depth_artifact_count`: number of decision trees, state machines, computation specs, impact matrices referenced
- `dependency_fan_out`: number of tasks that depend on THIS task (downstream consumers)

### Classification

| Score Range | Size | Label |
|-------------|------|-------|
| < 3 | **S** (Simple) | Straightforward implementation |
| 3 — 7 | **M** (Medium) | Moderate complexity |
| > 7 | **L** (Large) | Significant complexity, may need experienced developer |

### Edge Cases

- **No ACs available**: infer from FR description length. <100 words → S, 100-300 words → M, >300 words → L
- **Score exactly at boundary** (3.0 or 7.0): round UP (conservative estimate)
- **Merged task**: sum ACs from all source FRs
- **Split task**: divide original AC count proportionally

All scores tagged `[AI estimate]` in output.

---

## 4. Confidence Signal Rules (FR-016)

| Source | Format Quality | Confidence |
|--------|---------------|------------|
| Tier 1 (auto-requirement) + Must FR + complete ACs | High quality | **High** |
| Tier 1 + Should/Could FR | Good quality | **High** |
| Tier 2 (structured markdown) + inferred IDs | Moderate quality | **Medium** |
| Tier 3 (free-form) or `[INFERRED]` ACs | Low quality | **Low** |
| Any FR with `[NO AC - INFERRED]` | Missing data | **Low** |

Low confidence tasks are highlighted in the review summary (Phase 6) for focused user attention.

---

## 5. Phase Grouping Strategies (FR-006)

### Strategy Selection

1. **By CD domain** (preferred when CDs are well-defined): tasks from same CD go to same phase
2. **By architecture layer** (when cross-cutting): Foundation → Core Logic → Integration → Validation
3. **By dependency cluster** (fallback): group tasks with dense internal dependencies

### Constraints

- 3-7 tasks per phase (hard limits)
- <3 tasks → merge with closest related phase
- \>7 tasks → split by sub-concern
- No task may depend on a task in a LATER phase
- Phase names must be descriptive (not "Phase 1" or "Miscellaneous")

### Standard Phase Name Templates

If CD-derived names aren't suitable, use:
- **Foundation**: setup, configuration, schema, infrastructure
- **Core**: main business logic, primary features
- **Integration**: API connections, data flow, component wiring
- **Enhancement**: secondary features, optimizations
- **Validation**: testing infrastructure, E2E verification
