# Agentic Recursion (Delegate-and-Return)

## Problem

Some requests can't be answered by one agent, and you don't know the call graph in advance — the right next agent depends on what the current one finds. A fixed pipeline or a flat fan-out can't express "ask whoever is best, and let *them* ask whoever *they* think is best, until the question is actually answered." You need open-ended, self-determining delegation that still **returns a result**.

## Solution

Let an agent **call another agent of its own choosing**; that agent may call another; and so on — a **recursive call stack of agent choices** — until a **stop condition** is met, at which point the result **returns back up the stack** to the original caller. It's recursion where each frame is an agent picking the next agent.

Like the [observable loop](./observable-loop.md), a running recursion is a managed object: it can be **paused, stopped, its stop-condition changed, and its agent-choice set edited** at runtime.

### Architecture

```
   caller ──asks──▶ Agent A
                     │ picks ▶ Agent B
                     │          │ picks ▶ Agent C
                     │          │          │ stop-condition met
                     │          │          ▼ returns result
                     │          ◀──────────┘
                     ◀──────────┘
   caller ◀── final result (unwound up the stack)
```

### How It Works

1. **Enter:** the caller invokes Agent A with the request + a **stop condition** and an allowed **agent-choice set**.
2. **Choose or answer:** each agent decides — either it can answer (→ return), or it selects the **next agent** (from the choice set) and delegates, enriching context.
3. **Recurse:** repeat down the stack. Each frame is a real agent with its own judgment about who to call next.
4. **Stop:** the **stop condition** ends descent — answer found, max depth reached, budget exhausted, or no better agent to call.
5. **Return:** the result unwinds back up the stack to the caller, each frame optionally post-processing.

### Implementation with the SDK

```python
async def recurse(client, org_id, request, choices, stop, depth=0):
    agent = pick_entry_agent(choices)
    while True:
        resp = await client.ask_agent(org_id, agent, request, context=ctx)
        if stop(resp, depth):           # answer / max-depth / budget / no-better-choice
            return resp.result
        nxt = resp.next_agent           # the agent CHOSE who to call next
        if nxt is None:
            return resp.result
        request, agent, depth = resp.subrequest, nxt, depth + 1
        # pause/stop/stop-condition-change checked between frames by the runtime
```

In the agentic-patterns runtime the controls are MCP tools — `loop_pause/stop`, `loop_set_stop_condition`, `loop_add_agent_choice` — so the same recursion is drivable by an operator (L2) or, with an admin/owner grant, by an observer agent (L3).

### Best Practices

- **Always bound it:** a stop condition is mandatory — max depth + budget + "no better agent" — or recursion runs away. Make budget a first-class stop reason.
- **Enrich context down, summarize up:** each frame should hand the next agent everything it needs to disagree; each return should compress, not dump.
- **Constrain the choice set per scope:** an agent picks only from an allowed, scope-checked set — open-ended choice is where cost and security blow up.
- **Make every frame observable:** log the chosen agent + why at each depth, so the call tree is auditable and the observer can judge it.
- **Detect cycles:** guard against A→B→A loops (visited-set or depth cap).

### When to Use

- The next step genuinely depends on the current agent's findings (research, triage, routing, "who knows this?").
- Expertise is distributed and the path isn't known up front.
- You want delegation that returns a single composed answer, not a fan-out you must aggregate.

### Avoid When

- The call graph is fixed/known → use a pipeline or [multi-agent delegation](./multi-agent-task-delegation.md).
- Latency-critical paths where an unknown-depth descent is unacceptable.

### Related

- [Observable Loop](./observable-loop.md) — the other core agentic pattern; both share one control surface (pause/stop/change-condition/edit-choices) in the agentic-patterns runtime.
- [Multi-Agent Task Delegation](./multi-agent-task-delegation.md) — flat, known decomposition; recursion is the dynamic, self-determining cousin.
