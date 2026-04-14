# Review Standards

> Multi-perspective cross-review checklist system.

## Overview

One reviewer catches some bugs. Three reviewers with different perspectives catch almost all of them.

Our review system requires **three independent passes** before any change is considered ready:

| Reviewer | Perspective | Checklist Items |
|----------|-------------|----------------|
| Product Manager (奶茶) | Requirements, interaction, edge cases | 28 items |
| Visual Designer (可乐) | Layout, color, typography, responsive | 26 items |
| QA Engineer (牛奶) | Coverage, regression, performance, a11y | 33 items |

**Total: 87 items across 3 perspectives.**

## Process

```
Code Change
    │
    ├──→ Product Review (28 items)
    ├──→ Visual Review (26 items)
    └──→ QA Review (33 items)
           │
           ▼
    Cross-review phase
    (each reviewer checks others' findings)
           │
           ▼
    Issue merge & dedup (by Architect)
           │
           ▼
    Unified fix list with priority
```

## Key Rules

### Three-way or no merge
All three reviewers must independently pass. No exceptions. No "it's a small change."

### Unified issue schema
All review findings follow the same format:
```yaml
issue_id: VR-001
page_or_module: Home Screen
severity: P1
tags: [layout, responsive]
suggested_fix: Adjust grid breakpoint at 375px
blocker: true
```

### Issue merge node
After all reviews complete, the Architect deduplicates issues with the same root cause, unifies priority, and produces a single fix list. This prevents engineers from fixing the same underlying problem three times.

### Full coverage, not sampling
- Every page must be scrolled top to bottom
- Every clickable element must be clicked
- Every triggerable state must be triggered
- Code layer + render layer + operation layer

### Review against specs, not just checklists
Checklists catch generic issues. But reviewers must also verify against:
- Product design documents
- Visual design documents  
- Architecture design documents

The checklist catches "is this button aligned?" The spec check catches "is this the right button?"
