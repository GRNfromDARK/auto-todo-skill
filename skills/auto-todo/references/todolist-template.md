# Todolist.md Output Template

This is the reference template for auto-todo's output. The generated todolist.md must be compatible with auto-dev's LLM-based parsing.

---

## Template

```markdown
# {PROJECT_NAME} — 执行任务清单

> Generated: {YYYY-MM-DD HH:MM} by auto-todo
> Source: `{requirement_doc_relative_path}`
> Tech Stack: {detected_tech_stack_or_"not detected"}
> Tasks: {total_task_count} | Phases: {phase_count} | Critical Path: {critical_path_length} tasks

---

## Key Constraints

{Extract from requirement doc's NFRs and constraints. These become system_prompt constraints in auto-dev.}

- {Constraint 1}
- {Constraint 2}

---

## Phase A: {Phase Name}

### A-1: {Task Title} [{S|M|L}]
- **Traces to:** {FR-xxx, FR-yyy}
- **Depends on:** {none | task IDs like A-2, B-1}
- **Confidence:** {High|Medium|Low}
- **Description:**
  {Clear description of what to implement. Include enough context for auto-dev to generate a Card.}
- **Acceptance Criteria:**
  - [ ] {Given X, when Y, then Z}
  - [ ] {Given A, when B, then C}
- **Notes:** {Optional: engineering decisions, [INFERRED] tags, warnings}

### A-2: {Task Title} [{S|M|L}]
- **Traces to:** {FR-xxx}
- **Depends on:** A-1
- **Confidence:** High
- **Description:**
  {Description}
- **Acceptance Criteria:**
  - [ ] {criterion}

---

## Phase B: {Phase Name}

### B-1: {Task Title} [{S|M|L}]
...

---

## Traceability Matrix

| FR ID | FR Title | Task(s) | Priority | Status |
|-------|----------|---------|----------|--------|
| FR-001 | {title} | A-1 | Must | ✅ Covered |
| FR-002 | {title} | A-2, A-3 | Must | ✅ Covered |
| FR-003 | {title} | B-1 | Should | ✅ Covered |
| FR-004 | {title} | — | Should | ⚠️ UNCOVERED |

**Coverage:** {X}% ({covered_count}/{total_must_should_count} Must+Should FRs covered)

---

## Processing Summary

- **Input format:** {Tier 1: auto-requirement | Tier 2: structured markdown | Tier 3: free-form}
- **FRs parsed:** {count}
- **Tasks generated:** {count} (Pass-through: {n}, Merged: {n}, Split: {n})
- **Inferred items:** {count} ({list of [INFERRED] categories})
- **Engineering decisions:** {count}
```

---

## Format Rules

1. **Phase IDs**: Use letters: Phase A, Phase B, Phase C...
2. **Task IDs**: `{Phase Letter}-{Sequence Number}` — e.g., A-1, A-2, B-1
3. **Complexity tags**: Always in square brackets after title: `[S]`, `[M]`, `[L]`
4. **Traces to**: Always present — links back to source FR(s)
5. **Depends on**: Always present — use `none` if no dependencies
6. **Acceptance Criteria**: Use checkbox format `- [ ]` for auto-dev compatibility
7. **Metadata header**: Always include Generated, Source, Tech Stack, counts
8. **Traceability matrix**: Always appended at end
9. **Key Constraints section**: Extract from requirement NFRs — auto-dev uses these for system_prompt

## Compatibility Notes

- auto-dev uses LLM to parse this file, so format is **convention** not strict schema
- Keep structure consistent but don't obsess over exact whitespace
- Phase names must be human-readable (auto-dev displays them in pipeline output)
- Task descriptions should be detailed enough for auto-dev to generate a complete Card
- Include file paths, API endpoints, or data structures when known from requirements
