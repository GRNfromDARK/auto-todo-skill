# Auto-Todo Skill

Intelligent task decomposition skill for [Claude Code](https://claude.com/claude-code). Convert requirement documents into auto-dev compatible `todolist.md` files through structured decomposition, dependency analysis, and phase grouping.

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
auto-todo
```

- `autotodo: project-name` → auto-discovers requirement docs in `docs/requirements/`
- `autotodo: path/to/req.md` → uses the specified file directly
- `auto-todo` (no args) → scans for the latest requirement document

## Pipeline Positioning

```
auto-requirement → auto-todo → auto-dev
(产品决策层)         (工程任务层)   (代码实现层)
```

This skill handles **engineering decomposition** — converting product decisions into executable task lists. Macro architecture and code implementation are handled by upstream/downstream skills.

| Layer | Owner | Examples |
|-------|-------|---------|
| Macro Architecture | auto-requirement | "Use microservices", "PostgreSQL", "frontend-backend separation" |
| Engineering Details | **auto-todo** | "Need DB migration task", "Split frontend/backend phases", "API routes by module" |
| Code Implementation | auto-dev | Function design, variable naming, algorithm selection |

## Key Features

- **Multi-format input** — Three-tier parsing: auto-requirement output (Tier 1), structured markdown (Tier 2), free-form markdown (Tier 3)
- **Intelligent decomposition** — Merge/split/pass-through rules based on AC count, depth artifacts, and domain grouping; target: each task ≈ 1 auto-dev Card (2-8 hours)
- **Dependency-aware organization** — Topological sort, circular dependency detection, critical path identification
- **Phase grouping** — 3-7 tasks per phase, grouped by capability domain or architecture layer
- **Quick review gate** — Three-level progressive disclosure with HARD GATE before file write
- **S/M/L complexity scoring** — Heuristic scoring: `AC_count × 1.0 + depth_artifacts × 3.0 + dep_fan_out × 0.5`
- **Traceability matrix** — FR → Task mapping with 100% Must+Should coverage target
- **File write safety** — Automatic backup, atomic write, overwrite confirmation

## How It Works

```
Validate input → Parse requirement → Detect project context → Decompose tasks
    → Organize by dependency → Review & approve (HARD GATE) → Generate todolist.md
```

## Output

Auto-dev compatible `todolist.md` with:

1. Header with source doc, tech stack, task/phase counts
2. Phase-grouped tasks with S/M/L complexity tags
3. Per-task traces, dependencies, acceptance criteria
4. Traceability matrix with FR coverage percentage

## Changelog

### v1.1 (2026-03-03)

- Fixed 3 auto-dev compatibility issues in output format (test command section, constraint section, spec doc reference)

### v1.0 (2026-03-03)

- Initial release with 7-phase workflow, three-tier parsing, intelligent decomposition, dependency-aware organization, and traceability matrix

## Documentation

See [skills/auto-todo/README.md](skills/auto-todo/README.md) for detailed documentation.

## License

MIT
