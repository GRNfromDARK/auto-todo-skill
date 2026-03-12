# Auto-Todo Decision Rules Reference

Read this file when performing Phase 4 (task decomposition) and Phase 5 (task organization). SKILL.md contains the summary — this file has the complete rules.

---

## 1. Granularity Classifier

For each FR, classify what to do based on three factors:

```
Step 1: Count acceptance criteria (ACs)
  - No ACs → infer from description length, tag [NO AC - INFERRED]

Step 2: Check for depth artifacts
  - Decision trees, state machines, computation specs, impact matrices
  - If any exist → "has depth artifacts"

Step 3: Apply the table below
```

| AC Count | Depth Artifacts? | Same-CD Small FRs? | → Action |
|----------|-----------------|---------------------|----------|
| ≤2 | No | Yes | **MERGE** — combine with other small FRs in same CD |
| ≤2 | No | No | **PASS-THROUGH** — 1 FR → 1 Task |
| 3-5 | No | — | **PASS-THROUGH** — 1 FR → 1 Task |
| 3-5 | Yes | — | **SPLIT** — by artifact boundary |
| >5 | No | — | **SPLIT** — by AC groups (cluster related ACs into 2-3 tasks) |
| >5 | Yes | — | **SPLIT** — by concern (data / logic / UI / integration) |

### Merge rules

- Same CD + same MoSCoW priority only
- Merged task title: descriptive name covering all source FRs
- Merged task `traces_to`: all source FR IDs
- Merged ACs: union of all source FRs
- Max 4 FRs per merge

### Split rules

- Each subtask needs clear boundaries and must be independently testable
- Subtask IDs: append suffix (FR-005 → T-05a, T-05b)
- All subtasks share `traces_to` to the source FR
- Max 1 additional split round — if still too large, tag `[LARGE - consider manual split]`

### Granularity presets

The user can request a different granularity:
- **Fine**: bias toward splitting, tasks target 2-4 hours, merge only ≤1 AC FRs
- **Standard** (default): balanced, tasks target 2-8 hours, table above as-is
- **Coarse**: bias toward merging, tasks target 4-12 hours, merge up to ≤4 AC FRs

---

## 2. Dependency Rewrite Rules

When FRs have `depends_on` relationships, translate them to task-level after merge/split:

| FR-A and FR-B relationship | What happens to the dependency |
|---------------------------|-------------------------------|
| Both pass-through | Direct inherit: Task-A → Task-B |
| A was split → T-A1, T-A2 | Earliest subtask (T-A1) inherits upstream dep |
| B was split → T-B1, T-B2 | Task-A depends on latest subtask (T-B2, because all parts must complete) |
| Both merged into same task | Dependency eliminated (now internal) |
| A merged with C → T-AC | T-AC inherits A's dependency on B |
| B merged with D → T-BD | Task-A depends on T-BD |

### Cycle detection

After rewriting all dependencies:
1. Build directed graph (nodes = tasks, edges = depends_on)
2. Run DFS-based cycle detection
3. If cycle found → abort with error showing the cycle path

### Implicit dependencies

Beyond explicit `depends_on`, detect:
- Task modifies a file that another task reads → ordering dependency
- Task creates an API that another task consumes → ordering dependency
- Tag as `[IMPLICIT]` for user awareness

---

## 3. Complexity Scoring

```
score = AC_count × 1.0 + depth_artifact_count × 3.0 + dependency_fan_out × 0.5
```

| Score | Size | Meaning |
|-------|------|---------|
| < 3 | **S** | Straightforward |
| 3-7 | **M** | Moderate complexity |
| > 7 | **L** | Significant, may need experienced dev |

Edge cases:
- No ACs: infer from description — <100 words → S, 100-300 → M, >300 → L
- Score exactly at boundary (3.0 or 7.0): round up
- Merged task: sum ACs from all source FRs
- Split task: divide original AC count proportionally
- All scores tagged `[AI estimate]`

---

## 4. Confidence Signals

| Input quality | Confidence |
|--------------|------------|
| auto-requirement output + Must FR + complete ACs | **High** |
| auto-requirement + Should/Could FR | **High** |
| Structured markdown + inferred IDs | **Medium** |
| Free-form or `[INFERRED]` ACs | **Low** |
| Any FR with `[NO AC - INFERRED]` | **Low** |

Low confidence tasks get highlighted in the Phase 6 review summary.

---

## 5. Phase Grouping

### Strategy selection (in order of preference)

1. **By CD domain** — when CDs are well-defined, tasks from same CD go to same phase
2. **By architecture layer** — Foundation → Core Logic → Integration → Validation
3. **By dependency cluster** — group tasks with dense internal dependencies

### Constraints

- 3-7 tasks per phase (hard limits)
- <3 tasks → merge with closest related phase
- >7 tasks → split by sub-concern
- No task depends on a task in a later phase
- Phase names must be descriptive (not "Phase 1" or "Miscellaneous")

### Standard phase names

Use when CD-derived names aren't suitable:
- **Foundation**: setup, config, schema, infrastructure
- **Core**: main business logic, primary features
- **Integration**: API connections, data flow, component wiring
- **Enhancement**: secondary features, optimizations
- **Validation**: testing infrastructure, E2E verification
