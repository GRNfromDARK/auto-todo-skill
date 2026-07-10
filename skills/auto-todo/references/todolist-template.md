# `todolist.md` Template

Every actionable task and every independently verifiable condition uses a standard Markdown checkbox.
Adapt sections to the project, but preserve the task structure.

```markdown
# <Project or Feature> — Todo List

**Status:** Draft | Ready
**Generated:** YYYY-MM-DD
**Requirement sources:**

- `<path or URL>`
- `USER-BRIEF` — if the accepted user description is the only source

## Context

- **Repository state:** Existing system | Greenfield | Unknown
- **Detected stack:** ...
- **Relevant test commands:** ...
- **Important constraints:** ...
- **Evidence gaps:** ...

## Open Decisions

- **DEC-001 — <decision>:** why it matters, affected tasks, owner if known

## Assumptions

- **A-001:** assumption — evidence/confidence — impact if wrong

## Phase 1: <Meaningful phase name>

- [ ] T-001: <Action-oriented task title>
  - Source: FR-001, NFR-001 | SRC-001 | [GLUE]
  - Objective: <single observable outcome>
  - Scope: `<known paths, components, or interfaces>`
  - Dependencies: None | T-xxx
  - Parallel with: None | T-xxx
  - Risk: Low | Medium | High
  - Deliverables:
    - <code, schema, migration, documentation, configuration, or test artifact>
  - Verification:
    - [ ] <targeted automated check or explicit observable verification>
    - [ ] <relevant regression or compatibility check>
  - Notes: <constraints, assumptions, approvals, or rollback considerations>

- [ ] T-002: <Action-oriented task title>
  - Source: FR-002
  - Objective: ...
  - Scope: ...
  - Dependencies: T-001
  - Parallel with: None
  - Risk: Medium
  - Deliverables:
    - ...
  - Verification:
    - [ ] ...

## Phase 2: <Meaningful phase name>

- [ ] T-003: ...
  - Source: [GLUE] — connects FR-001 and FR-003
  - Objective: ...
  - Dependencies: T-001, T-002
  - Verification:
    - [ ] ...

## Final Validation

- [ ] VAL-001: All required targeted tests pass.
- [ ] VAL-002: Relevant regression checks pass.
- [ ] VAL-003: Requirement and acceptance-criterion coverage is complete.
- [ ] VAL-004: No unresolved blocker is incorrectly marked complete.

## Traceability

| Source requirement | Priority | Task(s) | Acceptance criteria covered | Status |
|--------------------|----------|---------|------------------------------|--------|
| FR-001 | Must | T-001, T-003 | AC-1, AC-2 | Covered |
| NFR-001 | Must | T-001 | latency check | Covered |

## Superseded or Deferred Tasks

Preserve completed historical tasks here when requirements change and deleting them would lose useful
project history.

- [x] T-900: <previously completed task> — superseded by <decision or requirement change>
```

## Checkbox Rules

- New task: `- [ ] T-001: ...`
- Completed task: `- [x] T-001: ...`
- New verification item: `    - [ ] ...`
- Completed verification item: `    - [x] ...`
- Do not use nonstandard checkbox states such as `[-]`, `[~]`, or `[!]`.
