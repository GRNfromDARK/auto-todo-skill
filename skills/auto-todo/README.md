# auto-todo

Independent requirement-to-checklist Skill.

It accepts any requirement document, specification, ticket, brief, or accepted user description and
produces a dependency-aware Markdown `todolist.md`.

Every task is a checkbox:

```markdown
- [ ] T-001: Implement the required behavior
```

Every independently verifiable condition is a nested checkbox. Existing `[x]` completion state is
preserved when a task list is revised.

Typical triggers:

- `生成任务清单`
- `转成 todolist`
- `拆成带 checkbox 的任务`
- `autotodo`
- `auto-todo`

The Skill does not write requirements, implement tasks, commit, deploy, or depend on another Skill.
