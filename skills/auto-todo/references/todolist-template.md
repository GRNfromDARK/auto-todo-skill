# Todolist.md Output Template

This is the reference template for auto-todo's output. The generated todolist.md must be **directly consumable by auto-dev** — auto-dev reads this file to map Groups→Phases and Tasks→Cards.

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
| 需求文档 | `{requirement_doc_relative_path}` | 产品需求与验收标准的唯一来源 |
| {其他相关文档} | `{path}` | {用途说明} |

---

## 测试命令

```bash
{detected_test_command}
```

{如未检测到测试框架，写明：}
> ⚠️ 未检测到测试框架，请在使用 auto-dev 前手动指定测试命令。

---

## 约束

{从需求文档 NFR 和架构约束中提取，auto-dev 将这些写入 system_prompt。}

- {约束 1，如：向后兼容现有 API}
- {约束 2，如：所有接口需有错误处理}

---

## Phase A：{Phase Name}

### A-1：{Task Title}
- {任务描述：清晰说明要实现什么，包含足够上下文让 auto-dev 生成 Card}
- {实现细节或技术要点}
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
| FR-003 | {title} | B-1 | Should | ✅ Covered |
| FR-004 | {title} | — | Should | ⚠️ UNCOVERED |

**Coverage:** {X}% ({covered_count}/{total_must_should_count} Must+Should FRs covered)

---

## 处理摘要

- **输入格式:** {Tier 1: auto-requirement | Tier 2: structured markdown | Tier 3: free-form}
- **FR 数:** {count}
- **任务数:** {count} (直通: {n}, 合并: {n}, 拆分: {n})
- **推断项:** {count} ({list of [INFERRED] categories})
- **工程决策:** {count}
```

---

## Format Rules

1. **Phase IDs**: Use letters with Chinese colon: `## Phase A：`, `## Phase B：`
2. **Task IDs**: `### {Phase Letter}-{Sequence}：{Title}` — e.g., `### A-1：创建数据模型`
3. **Task body**: Bullet list description (not bold key-value metadata). Keep it flat and readable — auto-dev maps each `###` task to a Card
4. **Dependencies**: Inline as `- 依赖：{task IDs}` (not bold `**Depends on:**`)
5. **Traceability**: Inline as `- 来源：{FR IDs}` (not bold `**Traces to:**`)
6. **Acceptance Criteria**: Under `- 验证：` with `- [ ]` checkbox sublists
7. **测试命令 section**: REQUIRED — auto-dev uses this as the test verification command for every Card
8. **设计文档 section**: REQUIRED — auto-dev uses this path as "single source of truth" for Card generation
9. **约束 section**: REQUIRED — auto-dev extracts these into system_prompt constraints
10. **可追溯性矩阵**: Always appended at end

## auto-dev Compatibility Notes

The output format is designed to match auto-dev's expected input:

| auto-todo output | auto-dev reads as |
|-----------------|-------------------|
| `## Phase A：Name` | Group → Phase A |
| `### A-1：Title` | Task → Card A.1 |
| `- 依赖：B-1` | Card ordering dependency |
| `- 验证：` + `- [ ]` items | Card acceptance criteria |
| `## 测试命令` | `{TEST_CMD}` in autodev.sh |
| `## 设计文档` | `{SPEC_PATH}` in system_prompt.md |
| `## 约束` | system_prompt.md constraints section |
| `> 唯一需求来源：` | `{TODOLIST_PATH}` reference |
