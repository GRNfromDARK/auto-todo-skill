# Auto-Todo: Requirement → Executable Task List

## What is this?

Auto-Todo is a Claude Code skill that converts requirement documents into **auto-dev compatible `todolist.md`** files. It goes beyond mechanical task decomposition — the core job is **engineering completion**: filling in the glue layers that requirements assume but never state, resolving ambiguous definitions into concrete specs, and ensuring nothing falls through the cracks.

### Pipeline Positioning

```
auto-requirement → auto-todo → auto-dev
(产品决策层)         (工程任务层)   (代码实现层)
```

auto-requirement decides *what* to build. auto-dev decides *how* to code it. **auto-todo owns the engineering middle layer** — the gap between product thinking and implementation. Requirements describe features in isolation; real systems need plumbing between them. auto-todo surfaces that plumbing and makes it explicit.

## Quick Start

Trigger in Claude Code:

```
autotodo: my-project          → auto-discovers docs/requirements/
autotodo: path/to/req.md      → uses specified file
生成任务清单                    → scans for latest requirement doc
转成todolist                   → converts requirement to task list
```

## Three Core Capabilities

### 1. Completeness

Every feature, constraint, and non-functional requirement maps to at least one task. The traceability matrix at the end enforces this — any uncovered FR is a gap.

### 2. Glue Layer Completion (most important)

Requirements describe individual features but **assume the connective tissue between them**. For example, a trading system requirement might list "strategy signals" and "order management" as separate features, but never mention:

- The shared database schema they both read/write
- The event pipeline connecting signal generation to order execution
- The authentication layer every endpoint needs
- Error propagation paths between modules
- Data format contracts between frontend and backend

auto-todo identifies and adds these as explicit tasks marked `[GLUE - 工程补全]`:

| Category | Examples |
|----------|---------|
| Data layer foundation | Shared schema design, migrations, connection pooling |
| Integration plumbing | Event buses, message queues, API gateways |
| Cross-cutting concerns | Auth middleware, error handling, logging |
| Interface contracts | API specs, message formats, protocol definitions |
| Build/deploy infrastructure | CI/CD, env config, containerization |

### 3. Ambiguity Resolution

Requirements describe *what* in business terms without specifying *how* at the engineering level. auto-todo resolves these into concrete definitions:

| Requirement says | Task defines |
|-----------------|-------------|
| "通过 API 传递数据" | `GET /api/v1/positions` → `{symbol, qty, cost, pnl}`, pagination |
| "实时推送行情" | WebSocket `ws://host/market/stream`, message format, heartbeat |
| "用户权限管理" | RBAC roles + permission matrix + JWT structure |

Engineering choices are tagged `[DECISION: rationale]`. High-stakes decisions get `[NEEDS CONFIRMATION]`.

## Codebase-Aware Decomposition

For upgrade requirements, auto-todo scans the existing codebase before generating tasks:

| What it scans | Why |
|--------------|-----|
| DB schema (models/, migrations/, *.sql) | Extend existing tables, not recreate |
| API routes (routes/, controllers/, api/) | Reference existing endpoints |
| Config (config/, settings.py, .env.example) | Extend existing config system |
| Types (types/, schemas/, dataclass files) | Reuse existing type definitions |
| Architecture patterns | Follow established patterns |
| Naming conventions | Match existing style |

**Rule**: If a definition already exists → reference and extend it. If nothing exists → define from scratch and tag `[GREENFIELD]`.

## Multi-Format Input Support

| Tier | Input Format | Strategy |
|------|-------------|----------|
| **Tier 1** | auto-requirement output (SG/CD/FR IDs) | Full structured parse |
| **Tier 2** | Structured markdown (headings + lists) | Heuristic parse with confirmation |
| **Tier 3** | Free-form markdown | LLM comprehension with `[INFERRED]` tags |

## 7-Phase Workflow

1. **Find requirement document** — resolve file path, validate input
2. **Parse requirement** — detect format (Tier 1/2/3), extract features and constraints
3. **Detect project context** — tech stack detection + existing codebase inventory
4. **Task decomposition** — decompose FRs + glue layer analysis + ambiguity resolution
5. **Organize tasks** — dependencies, topological sort, phase grouping (3-7 tasks/phase)
6. **Review summary** — present breakdown for user approval (hard gate before file write)
7. **Generate todolist.md** — write file with traceability matrix

## Output Format

```markdown
# [Project] — 执行任务清单

> Source: `docs/requirements/xxx-requirement.md`
> Tech Stack: Python 3.12, FastAPI, PostgreSQL | Tasks: 27 (18 req + 9 glue) | Phases: 5

## 设计文档
...
## 测试命令
...
## 约束
...

## Phase A: Foundation (Infrastructure & Glue)

### A-1: 共享数据层设计 [M] [GLUE - 工程补全]
- **Traces to:** FR-001, FR-002, FR-003
- **Description:** Design unified database schema covering orders, fills, positions...

### A-2: 策略-订单映射层 [S] [GLUE - 工程补全]
- **Traces to:** FR-001, FR-002
- **Description:** Define Signal → Order conversion interface...

---

## Traceability Matrix

| FR | Task(s) | Status |
|----|---------|--------|
| FR-001 | A-1, A-2, B-1 | Covered |

Coverage: 100% (15/15 Must+Should FRs covered)
Glue tasks: 9 (data layer ×2, integration ×3, cross-cutting ×2, interface ×2)
```

## File Structure

```
auto-todo/
├── SKILL.md                          ← Skill definition (loaded by Claude Code)
├── README.md                         ← This file
└── references/
    ├── decision-rules.md             ← Granularity classifier, dependency rewrite,
    │                                    complexity scoring, phase grouping rules
    └── todolist-template.md          ← Output format template for auto-dev
```

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

## Tips

1. **Use with auto-requirement output for best results** — Tier 1 parsing extracts full hierarchy with zero ambiguity
2. **Review the glue tasks carefully** — these are the highest-value additions auto-todo makes
3. **Check the traceability matrix** — any `[UNCOVERED]` FR is a gap in your task list
4. **For upgrade projects** — make sure the codebase inventory looks correct before approving
5. **Feed the output to auto-dev** — run `autodev: project-name` to start coding

---

# Auto-Todo：需求 → 可执行任务清单

## 这是什么？

Auto-Todo 是一个 Claude Code skill，将需求文档转化为 **auto-dev 兼容的 `todolist.md`**。它不是简单的任务拆解——核心工作是**工程补全**：补齐需求默认但未说明的胶水层，将模糊定义具象化为工程规格，确保没有遗漏。

### 三大核心能力

1. **完整性** — 需求中的每个功能、约束、非功能需求都映射到至少一个任务
2. **胶水层补全**（最重要）— 补齐功能之间的连接层：共享数据库 schema、事件总线、认证中间件、接口契约等
3. **模糊定义具象化** — "通过 API 传递数据" → 定义具体端点、请求/响应格式、协议

### 代码库感知

对于升级需求，auto-todo 会扫描现有代码库（数据库模型、API 路由、配置、类型定义），基于已有定义扩展而非重新发明。全新项目则从零定义完整规格。

### 流水线定位

```
auto-requirement → auto-todo → auto-dev
(产品决策层)         (工程任务层)   (代码实现层)
```

## 快速开始

```
autotodo: my-project          → 自动发现 docs/requirements/ 下的需求文档
autotodo: path/to/req.md      → 使用指定文件
生成任务清单                    → 扫描最新需求文档
转成todolist                   → 转换需求为任务清单
```

## 安装

```bash
# 推荐
npx skills add GRNfromDARK/auto-todo-skill

# 手动：放到以下任一位置
.claude/skills/auto-todo/     # 项目级
~/.claude/skills/auto-todo/   # 用户级
```
