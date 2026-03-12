# Auto-Todo Skill

Intelligent requirement-to-task decomposition skill for [Claude Code](https://claude.com/claude-code). Goes beyond mechanical task splitting — fills in the **glue layers** that requirements assume but never state, resolves ambiguous definitions into concrete engineering specs, and ensures nothing falls through the cracks.

## Install

```bash
npx skills add GRNfromDARK/auto-todo-skill
```

Or install globally:

```bash
npx skills add GRNfromDARK/auto-todo-skill -g
```

## Usage

Trigger in Claude Code with any of these keywords:

```
autotodo: my-project
auto-todo: path/to/requirement.md
生成任务清单
转成todolist
```

- `autotodo: project-name` — auto-discovers requirement docs in `docs/requirements/`
- `autotodo: path/to/req.md` — uses the specified file directly
- Also triggers when you ask to decompose/break down a requirement doc into tasks

## Pipeline Positioning

```
auto-requirement → auto-todo → auto-dev
(产品决策层)         (工程任务层)   (代码实现层)
```

auto-requirement decides *what* to build. auto-dev decides *how* to code it. **auto-todo owns the engineering middle layer** — the gap between product thinking and implementation.

## Three Core Capabilities

### 1. Completeness — nothing from the requirement is lost

Every feature, constraint, and non-functional requirement maps to at least one task. Enforced by a traceability matrix with 100% Must+Should FR coverage target.

### 2. Glue Layer Completion — the most important capability

Requirements describe features in isolation but **assume the connective tissue between them**. auto-todo identifies and adds these as explicit tasks:

- **Data layer foundation** — shared schema design, migration scripts, connection pooling
- **Integration plumbing** — event buses, message queues, API gateways
- **Cross-cutting concerns** — auth middleware, error handling, logging infrastructure
- **Interface contracts** — API endpoint specs, data format definitions, protocol choices
- **Build/deploy infrastructure** — CI/CD, environment config, containerization

### 3. Ambiguity Resolution — vague requirements become concrete definitions

Requirements often describe *what* in business terms without specifying *how* at the engineering level. auto-todo resolves these into concrete specs:

| Requirement says | Task defines |
|-----------------|-------------|
| "通过 API 传递数据" | `GET /api/v1/positions` → `{symbol, qty, cost, pnl}` |
| "实时推送行情" | WebSocket `ws://host/market/stream`, message format, heartbeat |
| "用户权限管理" | RBAC roles + permission matrix + JWT structure |

### Codebase-Aware Decomposition

For upgrade requirements, auto-todo scans existing code (models, routes, config, types) and extends what's already there rather than reinventing. Greenfield projects get full specs defined from scratch.

## Key Features

- **Multi-format input** — Three-tier parsing: auto-requirement output (Tier 1), structured markdown (Tier 2), free-form markdown (Tier 3)
- **Existing codebase inventory** — Scans DB schema, API routes, config, types, and architecture patterns before generating tasks
- **S/M/L complexity scoring** — Heuristic scoring per task
- **Dependency-aware organization** — Topological sort, cycle detection, critical path identification
- **Phase grouping** — 3-7 tasks per phase, grouped by capability domain
- **Review gate** — User must approve task breakdown before file generation
- **Traceability matrix** — FR → Task mapping with coverage percentage

## How It Works

```
Find requirement → Parse document → Detect project context & codebase inventory
    → Decompose tasks + Glue layer + Ambiguity resolution
    → Organize by dependency → Review & approve → Generate todolist.md
```

## Output

Auto-dev compatible `todolist.md` with:

1. Header with source doc, tech stack, task/phase counts
2. Phase-grouped tasks with complexity tags and detailed engineering descriptions
3. Glue tasks marked as `[GLUE - 工程补全]`
4. Traceability matrix with FR coverage percentage

## Changelog

### v2.0 (2026-03-12)

- Added three core capabilities: completeness, glue layer completion, ambiguity resolution
- Added codebase-aware decomposition (Phase 3b inventory + Phase 4c Rule #1)
- Optimized skill description for better triggering accuracy
- Restructured SKILL.md with progressive disclosure to references/

### v1.1 (2026-03-03)

- Fixed 3 auto-dev compatibility issues in output format

### v1.0 (2026-03-03)

- Initial release with 7-phase workflow, three-tier parsing, dependency-aware organization

## Documentation

See [skills/auto-todo/README.md](skills/auto-todo/README.md) for detailed documentation.

## License

MIT
