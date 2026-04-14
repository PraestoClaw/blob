# Dispatch Patterns

> Hard-won rules for multi-agent task dispatch.

## Overview

Dispatch is where multi-agent systems break. Not because agents can't do the work — because the coordinator can't track who's doing what, who's free, and what happens when something fails.

These patterns emerged from real failures. Every rule exists because we broke something without it.

## Core Rules

### 1. One Agent, One Task
```
✅ Agent A → Task 1
   Agent B → Task 2

❌ Agent A → Task 1 + Task 2 (concurrent)
```
No multitasking. Ever. It sounds inefficient, but it eliminates an entire class of context-switching bugs and half-finished work.

### 2. Concurrent Dispatch, Serial Execution
You *can* dispatch to multiple agents simultaneously. But each agent only works on one thing at a time. The parallelism is across agents, not within them.

```
Time →
Agent A: [====Task 1====][====Task 3====]
Agent B: [====Task 2====][====Task 4====]
Agent C: [========Task 5========]
```

### 3. Automatic Queue Draining
When any of these events occur, immediately check for idle agents and dispatch queued tasks:
- New task enters queue
- A subagent completes
- New message/heartbeat received
- Before any new dispatch

Never let an idle agent sit while tasks are queued.

### 4. Dependency-Aware Parallelism
Before parallelizing, draw the dependency graph. Only independent tasks run in parallel.

```
✅ Task A ──→ Task C
   Task B ──→ Task C    (A and B parallel, C waits)

❌ Task A ──→ Task B    (B depends on A's output)
   Task A ──→ (dispatch B immediately)
```

### 5. Failure Recovery
When a subagent fails (timeout, LLM error):
1. **Auto-retry once** with a stronger model (e.g., upgrade to GPT-5.4)
2. If retry fails, escalate to human
3. Never silently drop failed tasks

### 6. Gateway Restart Protocol
When the gateway restarts, all running subagents disconnect. Immediately:
1. `subagents list` — identify who was running
2. Kill disconnected sessions
3. Re-dispatch all interrupted tasks

### 7. Queue as Source of Truth
- Running tasks: check `subagents list` (live sessions)
- Queued tasks: check `DISPATCH-QUEUE.md` (persistent file)
- Board/dashboard: for human readability only, never for dispatch decisions

## Task Sizing

### Default: 1 file per task
Each subtask should touch **at most 1 file** (2 if strongly coupled). If a change exceeds ~100 lines, split it.

Why? Small tasks are:
- Easier to review
- Faster to retry on failure
- Less likely to conflict with parallel work
- Simpler to verify

### Decomposition process
1. Architect analyzes the full change
2. Maps file dependencies
3. Groups strongly-coupled changes
4. Splits into 1-file units
5. Draws dependency graph
6. Dispatches independent units in parallel

## Anti-Patterns

| Anti-Pattern | What Goes Wrong |
|-------------|----------------|
| Giving one agent 5 tasks | Context overflow, half-finished work |
| Dispatching dependent tasks in parallel | Race conditions, wasted work |
| Manual queue tracking | Forgotten tasks, idle agents |
| Ignoring timeouts | Zombie tasks blocking the pipeline |
| Trusting review results blindly | Hallucinated "all pass" reviews |
