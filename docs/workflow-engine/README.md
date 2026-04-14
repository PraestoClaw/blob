# Workflow Engine

> Declarative YAML-based workflow orchestration for multi-agent teams.

## Overview

When you have 10+ agents, you can't coordinate them with ad-hoc instructions. You need structure — but not so rigid that it can't handle reality.

Our workflow engine solves this with:
- **YAML workflow definitions** — Human-readable, version-controlled
- **DAG topology execution** — Steps run in dependency order
- **State persistence** — Workflows survive crashes and restarts
- **Loop limits** — Prevent infinite review cycles (default max: 10)
- **Automatic escalation** — When loops hit limits, escalate to human

## How It Works

```
Trigger (/实现 xxx)
    │
    ▼
Load YAML definition
    │
    ▼
Generate execution plan (DAG)
    │
    ▼
Execute step by step
    │  ├── Dispatch to agent
    │  ├── Collect output
    │  ├── Check gate conditions
    │  └── Loop if review fails
    │
    ▼
PR ready → Human review
    │
    ▼
Retrospective
```

## Workflow Types

| Command | Pipeline | Agents Involved |
|---------|----------|----------------|
| `/实现` | PRD → Wireframe → Visual Design → Architecture → Dev → Review | All |
| `/测试` | Test Plan → Evidence → Execute → Fix → Review | QA, GUI, Dev, PM |
| `/修复` | Decompose → Fix → Screenshot → Internal Review → Cross Review | Architect, Dev, GUI |
| `/视觉审查` | Evidence → Visual Audit → Cross Review → Fix → Review | Designer, PM, GUI |
| `/产品审查` | Evidence → Product Audit → Cross Review → Fix → Review | PM, Designer, GUI |
| `/功能审查` | Evidence → Functional Audit → Cross Review → Fix → Review | QA, PM, GUI |
| `/全审查` | Evidence → 3-way Parallel Audit → Merge → Fix → Review | All reviewers |

## Key Design Decisions

### Why YAML?
- Git-diffable, human-readable
- Easy to add new workflow types
- Separates workflow definition from execution logic

### Why loops with limits?
Real reviews rarely pass on the first try. But infinite loops waste resources. Default limit of 10 iterations balances thoroughness with practicality. If 10 rounds can't resolve it, a human needs to look.

### Why automatic escalation?
Agent teams can get stuck in review loops where each round introduces new issues. Automatic escalation prevents this by surfacing stuck workflows to human decision-makers.

## Schema

See [SCHEMA.md](./SCHEMA.md) for the full YAML schema specification.

## Examples

See the `examples/` directory for sample workflow definitions.
