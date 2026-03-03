# Auto-Todo: Requirement → Executable Task List

## What is this?

Auto-Todo is a Claude Code skill that converts requirement documents into **auto-dev compatible `todolist.md`** files through intelligent task decomposition. It reads structured requirements (SG→CD→FR hierarchy or any markdown), applies merge/split/pass-through rules, organizes tasks by dependency and phase, and outputs a ready-to-execute task list.

**Core principle:** Parse requirements → Decompose into right-sized tasks → Organize by dependency → Review with user → Generate todolist.md.

## Quick Start

Trigger with any of these keywords in Claude Code:

```
autotodo: my-project          → auto-discovers docs/requirements/
autotodo: path/to/req.md      → uses specified file
auto-todo                     → scans for latest requirement doc
```

## Multi-Format Input Support

Three-tier parsing strategy:

| Tier | Input Format | Strategy |
|------|-------------|----------|
| **Tier 1** | auto-requirement output (SG/CD/FR IDs) | Full structured parse |
| **Tier 2** | Structured markdown (headings + lists) | Heuristic parse with confirmation |
| **Tier 3** | Free-form markdown | LLM comprehension with `[INFERRED]` tags |

## Intelligent Task Decomposition

Each FR is classified using a granularity decision tree:

| AC Count | Depth Artifacts? | Same-CD Small FRs? | Action |
|----------|-----------------|---------------------|--------|
| ≤2 | No | Yes | **MERGE** — combine related small FRs |
| ≤2 | No | No | **PASS-THROUGH** — 1 FR → 1 task |
| 3-5 | No | — | **PASS-THROUGH** |
| 3-5 | Yes | — | **SPLIT** by artifact boundary |
| >5 | No | — | **SPLIT** by AC groups |
| >5 | Yes | — | **SPLIT** by concern |

Target: each task ≈ 1 auto-dev Card (2-8 hours work).

## S/M/L Complexity Scoring

```
score = AC_count × 1.0 + depth_artifacts × 3.0 + dependency_fan_out × 0.5
S: score < 3  |  M: score 3-7  |  L: score > 7
```

## 7-Phase Workflow

```
 ┌───────────────────────────┐
 │  1. Validate Input         │  Find and validate requirement doc
 └───────────┬───────────────┘
             ▼
 ┌───────────────────────────┐
 │  2. Parse Requirement      │  Tier 1/2/3 parsing, missing info handling
 └───────────┬───────────────┘
             ▼
 ┌───────────────────────────┐
 │  3. Project Context        │  Tech stack detection, engineering decisions
 └───────────┬───────────────┘
             ▼
 ┌───────────────────────────┐
 │  4. Task Decomposition     │  Merge / Split / Pass-through + S/M/L scoring
 └───────────┬───────────────┘
             ▼
 ┌───────────────────────────┐
 │  5. Task Organization      │  Dependencies, topological sort, phase grouping
 └───────────┬───────────────┘
             ▼
 ┌───────────────────────────┐     ┌──────────────────────┐
 │  6. Review & Approval      │─NO─▶│ Modify / Regenerate  │
 │     (HARD GATE)            │     │ → Back to 4          │
 └───────────┬───────────────┘     └──────────────────────┘
             │ YES
             ▼
 ┌───────────────────────────┐
 │  7. Generate & Write       │  Backup → Atomic write → Traceability matrix
 └───────────┬───────────────┘
             ▼
          Done → Next: `autodev: project-name`
```

## Output Format

```markdown
# [Project] — 执行任务清单

> Generated: 2026-03-03 14:30 by auto-todo
> Source: `docs/requirements/2026-03-03-project-requirement.md`
> Tech Stack: Node.js 20, React 19, Vitest | Tasks: 15 | Phases: 4

---

## Phase A: Foundation

### A-1: Setup database schema [M]
- **Traces to:** FR-001, FR-002
- **Depends on:** none
- **Description:** Create initial database schema...
- **Acceptance Criteria:**
  - [ ] AC-1: Schema migration runs successfully
  - [ ] AC-2: All tables created with correct types

---

## Traceability Matrix

| FR | Task(s) | Status |
|----|---------|--------|
| FR-001 | A-1 | ✅ Covered |
| FR-002 | A-1, A-2 | ✅ Covered |

Coverage: 100% (12/12 Must+Should FRs covered)
```

## File Structure

```
auto-todo/
├── SKILL.md                          ← Skill definition (loaded by Claude Code)
├── README.md                         ← This file
└── references/
    ├── decision-rules.md             ← Granularity classifier, dependency rewrite,
    │                                    complexity scoring, confidence signals
    └── todolist-template.md          ← Output format template for auto-dev
```

## Hard Gate

**Enforced constraint:** The skill will **NOT** write todolist.md until the user has reviewed and approved the task breakdown summary. No shortcutting the review gate.

## Large Document Support

| Document Size | Strategy |
|--------------|----------|
| <10 FRs | Single-pass processing |
| 10-30 FRs | Phased loading (parse → decompose → output) |
| >30 FRs | Sub-Agent dispatch per CD domain, merge results |

## Tips

1. **Use with auto-requirement output for best results** — Tier 1 parsing extracts full hierarchy with zero ambiguity
2. **Any structured markdown works** — You don't need auto-requirement; headings + bullet lists are enough
3. **Review the summary carefully** — 30 seconds of review catches most issues
4. **Use "modify" for fine-tuning** — Natural language commands like "merge T-03 and T-04" or "split T-05 into frontend and backend"
5. **Check the traceability matrix** — Any `[UNCOVERED]` FR is a gap in your task list
6. **Feed the output to auto-dev** — Run `autodev: project-name` to generate the full TDD pipeline

## Installation

### Via npx (Recommended)

```bash
npx skills add GRNfromDARK/auto-todo-skill
```

### Manual

Place the `auto-todo/` folder in either location:

```bash
# Project-level (current project only)
.claude/skills/auto-todo/

# User-level (all projects)
~/.claude/skills/auto-todo/
```

Claude Code will automatically discover and load the skill.

---

# Auto-Todo：需求 → 可执行任务清单

## 这是什么？

Auto-Todo 是一个 Claude Code skill，通过智能任务分解将需求文档转化为 **auto-dev 兼容的 `todolist.md`** 文件。它读取结构化需求（SG→CD→FR 层级或任意 markdown），应用合并/拆分/直通规则，按依赖关系和阶段组织任务，输出可直接执行的任务清单。

### 流水线定位

Auto-Todo 是三阶段开发流水线的**中间环节**：

```
auto-requirement → auto-todo → auto-dev
(产品决策层)         (工程任务层)   (代码实现层)
```

- **auto-requirement**：产品决策 — 为什么做、做什么、优先级和取舍
- **auto-todo**：工程分解 — 怎么拆、怎么排、怎么分组
- **auto-dev**：代码实现 — TDD 门控开发流水线

## 快速开始

在 Claude Code 中使用以下关键词触发：

```
autotodo: my-project          → 自动发现 docs/requirements/ 下的需求文档
autotodo: path/to/req.md      → 使用指定文件
auto-todo                     → 扫描最新需求文档
```

## 安装

### 通过 npx（推荐）

```bash
npx skills add GRNfromDARK/auto-todo-skill
```

### 手动安装

将 `auto-todo/` 文件夹放到以下任一位置：

```bash
# 项目级
.claude/skills/auto-todo/

# 用户级
~/.claude/skills/auto-todo/
```

## 核心特性

1. **多格式输入支持** — 三级解析策略（auto-requirement 标准输出 / 结构化 markdown / 自由格式）
2. **智能任务分解** — 合并小 FR、拆分大 FR、直通适中 FR，粒度对齐 auto-dev Card
3. **依赖感知排序** — 拓扑排序、循环检测、关键路径识别、并行任务标注
4. **阶段分组** — 3-7 个任务一组，按能力域/架构层/依赖聚类分组
5. **快速审查门** — 三级渐进披露，30 秒快速确认路径
6. **S/M/L 复杂度评分** — 基于 AC 数量、深度制品、依赖扇出的启发式评分
7. **可追溯性矩阵** — FR → Task 映射，100% Must+Should 覆盖目标
8. **文件写入安全** — 自动备份、原子写入、覆盖确认

## 使用技巧

1. **配合 auto-requirement 输出效果最佳** — Tier 1 解析零歧义
2. **任意结构化 markdown 也能用** — 不强制使用 auto-requirement
3. **认真查看审查摘要** — 30 秒审查就能发现大部分问题
4. **用 "modify" 微调** — 支持自然语言修改："合并 T-03 和 T-04"、"把 T-05 拆成前端和后端"
5. **检查可追溯性矩阵** — 任何 `[UNCOVERED]` 的 FR 都是任务缺口
6. **输出直接喂给 auto-dev** — 运行 `autodev: project-name` 生成完整 TDD 流水线
