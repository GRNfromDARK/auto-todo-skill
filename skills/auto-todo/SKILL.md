---
name: auto-todo
description: >
  Convert any requirement document, specification, brief, or accepted feature description into an
  implementation-ready Markdown `todolist.md` with task checkboxes and nested verification checkboxes.
  Trigger for "autotodo", "auto-todo", "生成任务清单", "转成 todolist", "拆成任务",
  "checkbox todo", or requests for a phased dependency-aware execution checklist. This skill is
  independent: it accepts requirements from any source and does not require or invoke another skill.
  Do not use it to write the requirement itself or to implement the tasks.
---

# Auto-Todo

Turn a requirement source into a durable, reviewable `todolist.md`. The output must be complete enough
for a human or coding agent to execute without losing scope, dependencies, verification, or important
engineering glue.

## Scope Boundary

This skill owns:

- extracting required behavior, constraints, and acceptance criteria from source material
- inspecting the existing codebase when implementation context is available
- decomposing requirements into dependency-aware, verification-bounded tasks
- adding necessary integration, migration, test, observability, and lifecycle work
- writing or revising a Markdown task list with checkboxes
- validating requirement and acceptance-criterion coverage

This skill does not:

- create or redesign the product requirement unless the source is unusably ambiguous
- implement code, run the implementation, commit, deploy, or publish
- require a particular requirement format or upstream skill
- generate provider-specific cards, prompts, shell runners, or model configuration

## Checkbox Contract

Every actionable task must start with a standard Markdown checkbox:

```markdown
- [ ] T-001: Task title
```

Every independently verifiable condition under a task must also use a nested checkbox:

```markdown
  - Verification:
    - [ ] The expected behavior is covered by an automated or explicit manual check.
```

Use only `[ ]` for open and `[x]` for completed. When revising an existing file, preserve completed
`[x]` tasks and their user notes unless the user explicitly asks to reset them.

## Operating Principles

1. **Any requirement source is acceptable.** Structured IDs are helpful but not required. Assign local
   source IDs when the input is free-form.
2. **Inspect before inventing.** In an existing repository, reference current modules, schemas, routes,
   tests, and conventions rather than planning duplicate replacements.
3. **One task, one verifiable outcome.** Task size is determined by behavioral and verification
   boundaries, not estimated human hours or a fixed number of files.
4. **Preserve traceability.** Every task names its source requirement, and every required acceptance
   criterion maps to at least one task verification checkbox.
5. **Add only necessary glue.** Include integration and cross-cutting tasks required for the requested
   system to work; do not add speculative infrastructure or unrelated improvements.
6. **Make uncertainty visible.** Use explicit assumptions or blockers instead of silently inventing
   high-impact contracts.
7. **Protect secrets and user work.** Never read secret-bearing files, and never discard completed
   checkboxes or unrelated edits in an existing task list.

## Workflow

### 1. Resolve the Requirement Source

- Use the path, content, ticket, or specification supplied by the user.
- If the user names a project but no file, search likely documentation locations and select a unique
  clear match.
- If multiple materially different sources match, ask which one is authoritative.
- If no written source exists but the user supplied a sufficiently clear accepted description, use that
  text and label the source `USER-BRIEF`.
- Reject missing or empty sources rather than fabricating a project.

Record all source paths or links at the top of the output.

### 2. Parse Requirements Without Requiring a Format

Extract:

- functional behavior and priorities
- acceptance criteria and examples
- non-functional constraints
- business rules, states, calculations, and data lifecycle
- explicit dependencies and external systems
- assumptions, decisions, non-goals, and open questions

If stable IDs exist, preserve them. Otherwise assign `SRC-001`, `SRC-002`, and so on for traceability.
Do not pretend inferred IDs were present in the source.

### 3. Inspect Existing Implementation Context

When a repository is available, inspect relevant:

- applicable `AGENTS.md` and repository documentation
- manifests and test configuration
- source modules, routes, interfaces, schemas, migrations, and config examples
- current tests and established validation commands
- architecture and naming patterns

Never read `.env`, credentials, tokens, private keys, secret files, or unrelated personal data.

Summarize only context that changes the task plan. Existing systems should be extended rather than
recreated unless replacement is an explicit requirement.

### 4. Build a Coverage Ledger

Before decomposition, create an internal ledger of:

- each source requirement
- each acceptance criterion
- each non-functional constraint
- each explicit non-goal
- each unresolved blocking decision

This ledger drives the final traceability table. Requirement-level coverage alone is insufficient when
one requirement contains backend, frontend, migration, and operational acceptance criteria.

### 5. Decompose Into Verification-Bounded Tasks

Create a separate task when work has a distinct observable outcome or can fail independently. Keep work
together when splitting would create artificial handoffs around one behavior.

Each task must contain:

- unique `T-xxx` ID and action-oriented title
- source requirement IDs
- objective and behavioral boundary
- likely scope or affected areas, when known
- dependencies on other task IDs
- expected deliverables
- nested verification checkboxes
- relevant constraints, assumptions, risks, or approval needs

Do not use arbitrary 2-8 hour estimates, fixed phase sizes, or formulaic S/M/L scores. Mark risk as
Low/Medium/High only when it helps execution ordering or review.

### 6. Add Necessary Engineering Glue

After requirement-derived tasks are drafted, check whether the requested behavior also needs:

- shared data contracts or migrations
- API, event, or frontend/backend integration
- authentication and authorization integration
- error propagation, retries, idempotency, or recovery
- configuration and secret references without exposing values
- logging, metrics, auditability, or operational readiness required by the source
- test fixtures, compatibility checks, or migration verification
- initialization, backfill, cleanup, or rollback steps

Add a glue task only when repository evidence or a source requirement makes it necessary. Mark its
source as `[GLUE]` and explain what requirement paths it connects.

### 7. Order Dependencies and Phases

- Build a directed dependency graph using task IDs.
- Reject or resolve cycles before writing the final file.
- Order tasks topologically; use product priority and risk to break ties.
- Group tasks into phases only when phases communicate meaningful execution boundaries.
- Do not impose a fixed number of tasks per phase.
- Mark tasks that can safely run in parallel, but never make one task depend on a later phase.

### 8. Handle Decisions and Blockers

- For low-impact reversible choices, state a conservative assumption and its evidence.
- For high-impact API, data-loss, security, compliance, migration, or public-contract choices, add a
  blocker under `Open Decisions` instead of inventing an answer.
- A blocker may prevent a dependent task while the rest of the task list remains usable.

### 9. Write or Revise `todolist.md`

Use [references/todolist-template.md](references/todolist-template.md).

- Use the user-specified output path when supplied; otherwise use `<project-root>/todolist.md`.
- When creating a new file, all task and verification checkboxes start unchecked.
- When updating an existing file, preserve completed `[x]` state, user notes, and stable task IDs.
- If a requirement was removed, do not silently delete a completed historical task; move it to a
  clearly labeled superseded section or ask when deletion would lose useful history.
- Do not require a separate approval merely to write the file when the user already asked to generate
  or revise it.

### 10. Validate the Output

Verify all of the following:

- every task begins with `- [ ] T-xxx:` or preserved `- [x] T-xxx:`
- every task has at least one nested verification checkbox
- task IDs are unique and dependencies reference existing IDs
- the dependency graph is acyclic
- all Must and Should requirements are covered
- every source acceptance criterion maps to a task verification checkbox
- glue tasks identify what they connect and why they are necessary
- constraints and non-goals have not been converted into contradictory tasks
- blockers and inferred assumptions are explicit
- completed checkbox state from an existing file is preserved
- the file contains no implementation code, hidden credential values, or provider-specific runner setup

## Delivery Report

After writing, report:

- output path
- total, open, and completed task counts
- phase count, if phases were useful
- requirement and acceptance-criterion coverage
- glue tasks added
- blockers and assumptions
- areas that could not be verified from the available repository context

Do not start implementing the checklist unless the user separately asks for implementation.
