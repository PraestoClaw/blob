# PraestoClaw

> Latin: *praestō* — "I am here. I am ready. I deliver."

Multi-agent team engineering patterns, extracted from PraestoClaw — a 10+ agent team building WeChat Mini Programs and Mini Games on [OpenClaw](https://github.com/openclaw/openclaw).

## Problem

Single-agent capability is mostly solved. Multi-agent coordination is not.

When you scale from 1 agent to 10, the bottleneck shifts from **capability** to **organization**: task decomposition, dispatch, quality gates, failure recovery, and cross-agent review.

This repo documents the patterns we use to solve these problems.

## Architecture

```
                    ┌─────────────┐
                    │   PraestoClaw (L0)  │
                    │  Coordinator │
                    └──────┬──────┘
           ┌───────┬───────┼───────┬───────┐
           ▼       ▼       ▼       ▼       ▼
       ┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐
       │ 芋泥 ││ 奶茶 ││ 可乐 ││ 牛奶 ││ 年糕 │
       │Arch. ││ PM   ││Design││  QA  ││ GUI  │
       └──────┘└──────┘└──────┘└──────┘└──────┘
           ┌───────┬───────┬───────┬───────┐
           ▼       ▼       ▼       ▼       ▼
       ┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐
       │ 汤圆 ││ 饺子 ││ 毛球 ││ 阿墨 ││ 包子 │
       │ Dev  ││ Dev  ││Infra ││  LLM ││  Ops │
       └──────┘└──────┘└──────┘└──────┘└──────┘
```

### Roles

| Agent | Role | Domain |
|-------|------|--------|
| PraestoClaw (Xiaojie) | L0 Coordinator | Task decomposition, dispatch, decisions |
| 芋泥 (Yuni) | Architect | Architecture design, internal review |
| 奶茶 (Naicha) | Product Manager | PRD, wireframes, product review |
| 可乐 (Kele) | Visual Designer | UI/UX design, visual review |
| 牛奶 (Niunai) | QA Engineer | Test planning, functional testing |
| 年糕 (Niangao) | GUI Operator | Screenshot capture, simulator automation |
| 汤圆 (Tangyuan) | Full-stack Dev | Implementation |
| 饺子 (Jiaozi) | Full-stack Dev | Implementation |
| 毛球 (Maoqiu) | Infrastructure | DevOps, security |
| 阿墨 (Amo) | LLM Specialist | Prompt engineering, eval |
| 包子 (Baozi) | Operations | Content ops |
| 豆包 (Doubao) | Data | Data analysis |

Each agent is bound to a domain-expert skill — a structured knowledge base (SKILL.md + reference files) that provides deep expertise, not just a system prompt.

## Contents

| Directory | Description |
|-----------|-------------|
| [workflow-engine/](./workflow-engine/) | Declarative YAML workflow orchestration. 10 workflow types, DAG execution, state persistence, loop limits, auto-escalation. |
| [skills/](./skills/) | Domain-expert skill templates. 8 templates, 72 reference files, ~870KB knowledge base. |
| [review-standards/](./review-standards/) | Three-way cross-review system. 87 checklist items across product (28), visual (26), and test (33). |
| [dispatch-patterns/](./dispatch-patterns/) | Multi-agent dispatch rules. Queue management, load balancing, failure recovery. |
| [field-notes/](./field-notes/) | Incident reports and lessons learned. |

## Principles

- **Action first, report later.** Report verifiable results, not status updates.
- **One agent, one task.** Parallel across agents, serial within each.
- **Three-way review or no merge.** Product + Visual + Architecture must independently pass.
- **Failures become rules.** Every incident gets a post-mortem and a guardrail.
- **Code, render, operation.** Review all three layers. Reading code is not enough.
- **Quality over token economy.** Get it right; rework costs more than thoroughness.

## Stack

- [OpenClaw](https://github.com/openclaw/openclaw) — Agent runtime
- [GitHub Copilot CLI](https://github.com/features/copilot) — Code execution (ACP harness)
- Feishu — Team communication

## License

MIT
