# Auto-Todo Skill

Independent requirement-to-checklist skill. It accepts any requirement document, specification,
ticket, brief, or accepted feature description and produces an implementation-ready Markdown
`todolist.md`.

It does not require a particular upstream skill and does not implement the generated tasks.

## Install

```bash
npx skills add GRNfromDARK/auto-todo-skill
```

Global installation:

```bash
npx skills add GRNfromDARK/auto-todo-skill -g
```

## Usage

Typical triggers:

```text
autotodo: path/to/requirement.md
auto-todo: my-project
把这个需求拆成带 checkbox 的 todolist
生成任务清单
```

## Checkbox Contract

Every task uses a standard Markdown checkbox:

```markdown
- [ ] T-001: Implement the required behavior
  - Source: FR-001
  - Dependencies: None
  - Verification:
    - [ ] Targeted behavior check passes
    - [ ] Relevant regression check passes
```

New items use `[ ]`; completed items use `[x]`. When an existing task list is revised, completion state,
stable task IDs, and user notes are preserved.

## Key Capabilities

- accepts structured or free-form requirement sources
- inspects existing code before planning replacement work
- decomposes by independently verifiable outcomes rather than time estimates
- adds necessary data, integration, migration, test, and operational glue
- orders tasks with an acyclic dependency graph
- maps every required acceptance criterion to a verification checkbox
- records assumptions and high-impact blockers explicitly
- produces a final traceability table

## Repository Structure

```text
skills/auto-todo/
├── SKILL.md
├── README.md
└── references/
    └── todolist-template.md
```

See [skills/auto-todo/SKILL.md](skills/auto-todo/SKILL.md) for the workflow and
[todolist-template.md](skills/auto-todo/references/todolist-template.md) for the output format.

## License

MIT
