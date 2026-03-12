---
name: auto-todo
description: >
  Invoke for "autotodo", "auto-todo", "生成任务清单", "转成todolist", or any request to
  convert/decompose a requirement document into a todolist.md. This is the bridge between
  requirements and coding: it reads a requirement spec (markdown file) and outputs a phased,
  dependency-ordered todolist.md ready for auto-dev. Trigger whenever the user has a requirement
  doc and wants tasks extracted from it, asks "下一步" after auto-requirement, mentions breaking
  down features into a task list, or references generating todolist.md from any spec file.
  Do NOT trigger for writing requirements (use auto-requirement), running autodev pipelines,
  general coding, or documentation.
---

# Auto-Todo: Requirement → Executable Task List

Transform requirement documents into **auto-dev compatible todolist.md** files. This is not just mechanical task decomposition — the core job is **engineering completion**: filling in the glue layers that requirements assume but never state, resolving ambiguous definitions into concrete specs, and ensuring nothing falls through the cracks.

### Where this sits in the pipeline

```
auto-requirement  →  auto-todo  →  auto-dev
(what to build)      (task plan)    (write code)
```

auto-requirement decides *what* to build. auto-dev decides *how* to code it. **auto-todo owns the engineering middle layer** — the gap between product thinking and implementation. Requirements describe features in isolation; real systems need plumbing between them. auto-todo's job is to surface that plumbing and make it explicit.

## Three Core Capabilities

Every todolist.md must demonstrate these three qualities. They are the standard by which the output is judged.

### 1. Completeness — nothing from the requirement is lost

Every feature, constraint, and non-functional requirement must map to at least one task. The traceability matrix at the end enforces this. If a requirement is not covered, the todolist is broken.

### 2. Glue Layer Completion — the most important capability

Requirement documents describe individual features but **assume the connective tissue between them**. For example, a payment system requirement might list "payment processing" and "order management" as separate features, but never mention:

- The shared database schema they both read/write
- The event bus or message queue that connects payment status changes to order state
- The authentication/authorization layer every endpoint needs
- Error propagation paths between modules
- Data format contracts between frontend and backend
- Migration scripts for the database
- Configuration management for external service credentials

**auto-todo must identify and add these as explicit tasks.** Think top-down: what does the whole system need to function that no single feature requirement covers? These glue tasks often include:

- **Data layer foundation** — shared schema design, migration scripts, connection pooling
- **Integration plumbing** — event buses, message queues, API gateways, service discovery
- **Cross-cutting concerns** — auth middleware, error handling framework, logging infrastructure
- **Interface contracts** — API endpoint specs, data format definitions, protocol choices
- **Build/deploy infrastructure** — CI/CD, environment config, containerization

Mark glue tasks as `来源：[GLUE - 工程补全]` so they're distinguishable from requirement-derived tasks.

### 3. Ambiguity Resolution — vague requirements become concrete definitions

Requirements often describe *what* happens in business terms without specifying *how* at the engineering level. auto-todo must resolve these into concrete definitions in the task description. Examples:

**For existing codebases (upgrade requirements)** — check what's already defined before inventing new specs:
- If `src/api/v1/positions.py` already exists → task says "extend existing positions API with new field X"
- If `src/models/order.py` defines an `Order` model → task says "add `timeout_at` field to existing `OrderModel`"

**For greenfield projects** — define concrete specs from scratch:

| Requirement says | Task must define |
|-----------------|-----------------|
| "通过 API 将数据传递给前端" | `GET /api/v1/positions` → `{symbol, qty, cost, pnl}`, pagination |
| "实时推送行情数据" | WebSocket `ws://host/market/stream`, message format, heartbeat |
| "用户权限管理" | RBAC roles + permission matrix + JWT structure |

Use `[DECISION: rationale]` for engineering choices. Use `[NEEDS CONFIRMATION]` for high-stakes decisions.

## Workflow

Execute these 7 phases in order. The user must approve the task breakdown before any file is written — this review gate exists because task structure directly shapes how auto-dev generates code, so getting it wrong wastes significant downstream effort.

### Phase 1: Find the requirement document

Resolve the input:
- User provides a file path → use it directly
- User provides a project name → scan `docs/requirements/` for matching `*-requirement.md`
- Multiple matches → ask user to pick one
- Nothing found → suggest running auto-requirement first

Validate: abort on missing/empty file. Warn (but continue) if file >100KB or non-.md extension.

### Phase 2: Parse the requirement document

Detect the input format and parse accordingly:

| Format | How to detect | Parse strategy |
|--------|--------------|----------------|
| **auto-requirement output** | Has `SG-`, `CD-`, `FR-` IDs with hierarchy | Full structured parse — extract all nodes, priorities, dependencies |
| **Structured markdown** | Has headings + lists, but no FR IDs | Heuristic parse — infer hierarchy from headings, assign synthetic IDs. Show parse receipt for user confirmation |
| **Free-form markdown** | Minimal structure | LLM comprehension — extract requirements, mark all as `[INFERRED]`, require user confirmation |

**Filling gaps:** When information is missing (no priority, no acceptance criteria, no dependencies), infer reasonable defaults and tag them `[INFERRED]`. Batch all inferences for the review summary — don't interrupt the user with individual questions.

**Large documents:** <10 FRs → single pass. 10-30 FRs → phased processing. >30 FRs → dispatch sub-agents per capability domain and merge.

### Phase 3: Detect project context and existing codebase

This phase has two jobs: understand the tech environment AND inventory what already exists. Many requirements are upgrades to existing systems — the todolist must build on what's there, not reinvent it.

**Security:** Never read `.env`, `credentials.*`, `*secret*`, or other sensitive files.

#### 3a. Tech stack detection

Read manifest files (`package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, etc.) to detect language, framework, and test runner. Detect the test command — this populates `## 测试命令` for auto-dev.

#### 3b. Existing codebase inventory (CRITICAL for upgrade requirements)

Use Glob and Read to scan for existing definitions. The goal is to understand what's already built so the todolist extends rather than replaces:

1. **Database schema** — scan for `models/`, `migrations/`, `schema.py`, `*.sql`, ORM model files. If tables already exist, the task should say "extend the `orders` table with field X" not "create an `orders` table".
2. **API endpoints** — scan for `routes/`, `controllers/`, `api/`, `openapi.json`, FastAPI/Express/Gin router files. If endpoints exist, reference them: "add `DELETE /api/v1/orders/{id}` alongside existing order endpoints".
3. **Config structure** — scan for `config/`, `settings.py`, `.env.example`, YAML configs. If a config system exists, extend it rather than creating a new one.
4. **Type definitions** — scan for `types/`, `models/`, `interfaces/`, `schemas/`, dataclass/Pydantic files. Reuse existing types in task descriptions.
5. **Architecture patterns** — identify the existing patterns (event bus? service layer? repository pattern?). New tasks must follow established patterns.
6. **Naming conventions** — observe file naming, variable naming, module structure. New code should match.

**Show a brief inventory receipt:**
```
📂 Existing Codebase Inventory
─────────────────────────────
DB Schema: 12 tables found (orders, fills, positions... in src/models/)
API Routes: 8 endpoints found (in src/api/v1/)
Config: pydantic-settings based (src/config/settings.py)
Types: Pydantic models (src/schemas/)
Pattern: Event-driven + Repository pattern
Test framework: pytest (tests/ directory, 47 existing tests)
```

If the project directory is empty or contains no code, note `[GREENFIELD PROJECT]` and proceed — in this case, auto-todo defines everything from scratch.

#### 3c. Engineering decisions

Based on requirement NFRs + project context + existing codebase:
- "前后端分离" → create separate Frontend/Backend phases
- Database FRs + existing schema → extend/migrate tasks, not recreate
- Tag decisions as `[DECISION: reason]`
- Flag anything that contradicts existing code or the requirement doc

### Phase 4: Task Decomposition + Glue Layer + Ambiguity Resolution

This is the core phase. It has three sub-steps that must ALL happen:

#### 4a. Decompose features into tasks

Each task should represent roughly one auto-dev Card (2-8 hours of work). Apply the granularity classifier from `references/decision-rules.md`:

- **Small features** (≤2 acceptance criteria, same domain) → **merge** into one task
- **Medium features** (3-5 ACs, no complex artifacts) → **pass through** as-is (1 FR = 1 task)
- **Large features** (>5 ACs or complex artifacts like state machines) → **split** by concern boundary

Every task must have a `traces_to` field linking back to source requirement(s).

**Complexity scoring:** Each task gets an S/M/L size estimate. See `references/decision-rules.md` for the formula.

#### 4b. Glue layer analysis (CRITICAL)

After decomposing all FRs, step back and look at the system as a whole. Ask:

1. **Data flow:** How does data move between features? Do they share a database? Do they need an event bus? Add data layer and integration tasks.
2. **Common infrastructure:** What does every feature need but no feature explicitly requests? (auth, error handling, logging, config management) Add foundation tasks.
3. **Interface contracts:** Where do features interact? Define the APIs, message formats, and protocols between them. Add interface definition tasks or specify contracts within existing tasks.
4. **Deployment plumbing:** Does the system need database migrations, environment configs, CI/CD setup, containerization? Add infrastructure tasks.
5. **Missing lifecycle steps:** Is there initialization/setup that must happen before features work? Teardown/cleanup? Seed data? Add lifecycle tasks.

For each glue task, write a clear description of what it connects and why it's needed. Mark `来源：[GLUE - 工程补全]`.

#### 4c. Resolve ambiguity in task descriptions

Go through every task and check: is the description concrete enough for an engineer (or auto-dev) to implement without guessing?

**Rule #1: Check existing code first.** Before defining any interface, schema, or config, consult the Phase 3 codebase inventory:

- If the definition **already exists** → reference it: "扩展现有 `OrderModel`（`src/models/order.py`）增加 `timeout_at` 字段"
- If a **pattern** exists → follow it: "参照现有 `UserAPI`（`src/api/v1/users.py`）路由模式，新增订单路由"
- If **nothing exists** (greenfield or new domain) → define from scratch with full specs, tag `[GREENFIELD]`

**Rule #2: Make vague requirements concrete** (after checking existing code):

- Vague verbs like "处理"、"管理"、"支持" → replace with specific operations
- "通过 API" → if endpoints exist, say which to extend; if not, define endpoint/method/schema
- "数据存储" → if tables exist, say which to alter + migration; if not, define full schema
- "通知用户" → specify channel, message format, trigger conditions
- "权限控制" → if auth exists, extend it; if not, define auth model

**Codebase-aware vs codebase-blind example:**

| Scenario | BAD (codebase-blind) | GOOD (codebase-aware) |
|----------|---------------------|----------------------|
| 升级：新增订单超时 | "设计 orders 表: id, symbol..." | "在现有 `src/models/order.py` 的 `OrderModel` 增加 `timeout_at` 字段，新增 Alembic 迁移" |
| 全新项目 | 同右 | "定义 orders 表 `[GREENFIELD]`: id UUID PK, symbol VARCHAR(32)..." |

Use `[DECISION: rationale]` for engineering choices. Use `[NEEDS CONFIRMATION]` for high-stakes decisions you're not confident about.

### Phase 5: Organize tasks

**5a. Dependencies** — Translate requirement-level dependencies to task-level. Glue tasks often become dependencies for multiple feature tasks. See `references/decision-rules.md` for the full impact matrix. Run cycle detection.

**5b. Topological sort** — Order tasks by dependency graph. Break ties by priority (Must > Should > Could). Identify the critical path and parallelizable tasks.

**5c. Phase grouping** — Group tasks into phases of 3-7 tasks each. Glue/foundation tasks typically go in Phase A. Prefer grouping by capability domain; fall back to architecture layers (Foundation → Core → Integration → Validation). No task may depend on a task in a later phase.

### Phase 6: Present review summary

Show a concise summary for user approval. Keep it under 20 lines:

```
📋 Task Breakdown Summary
─────────────────────────
Source: [requirement doc name]
Tech Stack: [detected or "not detected"]
Total: [N] tasks ([M] from requirements + [K] glue tasks) across [P] phases
Critical path: [C] tasks
Complexity: S×[a] M×[b] L×[c]

Phase 1: [Name] ([n] tasks)
Phase 2: [Name] ([n] tasks)
...

🔧 Glue tasks added: [count] (data layer, integration, infrastructure, etc.)
📐 Ambiguities resolved: [count] (API contracts, protocol choices, etc.)
⚠️ Needs confirmation: [count items flagged NEEDS CONFIRMATION]
```

Then ask: "要继续生成 todolist.md 吗？你也可以要求查看详情、修改任务分组或重新生成。"

If the user requests changes, apply them, revalidate dependencies, confirm, and loop back. Only proceed to file generation after explicit approval.

### Phase 7: Generate todolist.md

**Before writing:** If `todolist.md` already exists, create a timestamped backup and ask the user whether to overwrite, create a versioned file, or view the diff.

**Output format:** Follow `references/todolist-template.md` exactly. The three sections below are required because auto-dev reads them directly:

| Section | What auto-dev does with it |
|---------|---------------------------|
| `## 设计文档` | Uses as `{SPEC_PATH}` — the source of truth for generating Cards |
| `## 测试命令` | Uses as `{TEST_CMD}` — runs this after every Card implementation |
| `## 约束` | Writes into `system_prompt.md` — constraints every Card must follow |

**Task description quality:** Each task's bullet-point description should contain enough engineering detail for auto-dev to generate a Card without needing to guess. This means:
- Specific technologies and libraries to use
- Data structures and schemas where relevant
- API endpoints and request/response formats where relevant
- Error handling expectations
- Performance constraints from NFRs that apply to this task

**Traceability matrix:** Append at the end, mapping every FR to its task(s). Include glue tasks in a separate section. Flag any Must/Should FR without coverage. Target: 100% coverage of Must + Should FRs.

**After writing, confirm:**
```
✅ todolist.md generated
   Path: [relative path]
   Tasks: [N] ([M] requirement + [K] glue) across [P] phases
   Coverage: [X]% of Must+Should FRs
   Glue tasks: [list of glue task IDs and what they connect]

Next step: Run `autodev: [project-name]` to start coding.
```

## Example: Glue Layer in Action

Given a requirement with these features:
- FR-001: 策略信号生成
- FR-002: 订单管理
- FR-003: 持仓计算

A naive decomposition just creates 3 tasks. But the **glue layer analysis** reveals:

```
🔧 Glue tasks identified:
- [GLUE] 策略-订单映射层：定义 Signal → Order 转换接口
  SignalHandler.on_signal(signal: Signal) → List[Order]
  Signal schema: {strategy_id, symbol, direction, quantity, order_type, price}

- [GLUE] 成交回报处理管道：连接订单成交到持仓更新
  FillHandler.on_fill(fill: Fill) → PositionUpdate
  Event flow: Exchange → FillHandler → PositionManager.update()

- [GLUE] 共享数据层：设计统一的数据库 schema
  Tables: orders, fills, positions, strategies
  Constraints: 所有金额字段使用 Decimal
```

These glue tasks are what makes a set of features into a working system.
