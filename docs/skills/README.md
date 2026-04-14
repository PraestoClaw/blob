# Expert Skills

> Domain-expert skill templates for specialized AI agents.

## Overview

A generic agent with a good system prompt can do many things adequately. A specialized agent with deep domain knowledge can do one thing *well*.

Our skill system gives each agent a structured knowledge base:

```
skill/
├── SKILL.md          # Routing rules, output templates, activation triggers
└── references/       # Deep knowledge base files (loaded on demand)
    ├── 01-principles.md
    ├── 02-methods.md
    ├── ...
    └── 08-advanced.md
```

## Design Principles

**Separation of concerns**: Each agent is an expert in one domain. They don't try to do everything.

**On-demand loading**: Reference files are loaded only when the skill activates. This keeps context windows efficient.

**Knowledge synthesis**: Each skill fuses 2-3 authoritative sources into a coherent methodology. Not a textbook dump — a synthesized, opinionated framework.

## Available Templates

| Skill | Domain | Key Sources |
|-------|--------|------------|
| Architecture Master | System design, tech decisions, ADRs | Martin Fowler + Uncle Bob + Google SWE Book |
| Fullstack Master | Code quality, engineering practices | Pragmatic Programmer + Kent Beck + Google Style Guides |
| Product Master | Product analysis, PRD, user value | 俞军 + Marty Cagan + 张小龙 |
| Visual Design Master | UI/UX, visual hierarchy, interaction | Dieter Rams + Don Norman + 原研哉 |
| QA Master | Test strategy, coverage, defect analysis | James Bach + Kent Beck TDD + Google Testing |
| Prompt Master | Prompt engineering, agent design, eval | Anthropic + Lilian Weng + OpenAI Cookbook |
| WeChat Ops Master | Content operations, growth | 粥左罗 + 郭静 + B2B SaaS patterns |

## How to Use

1. Pick a skill template relevant to your agent's role
2. Customize the `references/` files for your domain context
3. Set the agent's `AGENTS.md` to force-load the skill
4. The agent now has deep domain expertise, not just generic capability

## Stats

- 8 skill templates
- 72 reference files
- ~870KB total knowledge base
