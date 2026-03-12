# Todolist.md Output Template

The generated todolist.md must be **directly consumable by auto-dev**. auto-dev maps `## Phase` → Groups and `### Task` → Cards.

---

## Template

```markdown
# {PROJECT_NAME} — 执行任务清单

> 唯一需求来源：`{requirement_doc_relative_path}`
> Generated: {YYYY-MM-DD HH:MM} by auto-todo
> Tech Stack: {detected_tech_stack} | Tasks: {total_task_count} | Phases: {phase_count}

---

## 设计文档

| 文件 | 路径 | 用途 |
|------|------|------|
| 需求文档 | `{requirement_doc_path}` | 产品需求与验收标准的唯一来源 |

---

## 测试命令

```bash
{detected_test_command}
```

> ⚠️ 如未检测到测试框架，写明：未检测到测试框架，请在使用 auto-dev 前手动指定测试命令。

---

## 约束

- {约束 1，从需求 NFR 提取}
- {约束 2}

---

## Phase A：{Phase Name}

### A-1：{Task Title}
- {任务描述：清晰说明要实现什么，包含足够上下文让 auto-dev 生成 Card}
- 依赖：{none | A-2, B-1}
- 来源：{FR-xxx, FR-yyy}
- 验证：
  - [ ] {验收标准 1}
  - [ ] {验收标准 2}

### A-2：{Task Title}
- {任务描述}
- 依赖：A-1
- 来源：{FR-xxx}
- 验证：
  - [ ] {验收标准}

---

## Phase B：{Phase Name}

### B-1：{Task Title}
...

---

## 可追溯性矩阵

| FR ID | FR Title | Task(s) | Priority | Status |
|-------|----------|---------|----------|--------|
| FR-001 | {title} | A-1 | Must | ✅ Covered |
| FR-002 | {title} | A-2, A-3 | Must | ✅ Covered |
| FR-003 | {title} | — | Should | ⚠️ UNCOVERED |

**Coverage:** {X}% ({covered}/{total} Must+Should FRs covered)

---

## 处理摘要

- **输入格式:** {Tier 1/2/3}
- **FR 数:** {count}
- **任务数:** {count} (直通: {n}, 合并: {n}, 拆分: {n})
- **推断项:** {count}
- **工程决策:** {count}
```

---

## Format Rules

1. **Phase IDs**: Letters with Chinese colon — `## Phase A：`, `## Phase B：`
2. **Task IDs**: `### {Letter}-{Number}：{Title}` — e.g., `### A-1：创建数据模型`
3. **Task body**: Bullet list. Keep flat and readable — auto-dev maps each `###` to a Card
4. **Dependencies**: `- 依赖：{task IDs}`
5. **Traceability**: `- 来源：{FR IDs}`
6. **Acceptance criteria**: Under `- 验证：` with `- [ ]` checkbox sublists
7. **Three required sections**: `设计文档`, `测试命令`, `约束` — auto-dev depends on these
8. **可追溯性矩阵**: Always at the end

## auto-dev Compatibility Map

| auto-todo output | auto-dev reads as |
|-----------------|-------------------|
| `## Phase A：Name` | Group → Phase A |
| `### A-1：Title` | Task → Card A.1 |
| `- 依赖：B-1` | Card ordering dependency |
| `- 验证：` + `- [ ]` | Card acceptance criteria |
| `## 测试命令` | `{TEST_CMD}` in autodev.sh |
| `## 设计文档` | `{SPEC_PATH}` in system_prompt.md |
| `## 约束` | system_prompt.md constraints |
| `> 唯一需求来源：` | `{TODOLIST_PATH}` reference |
