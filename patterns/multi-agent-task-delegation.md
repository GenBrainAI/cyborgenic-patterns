# Multi-Agent Task Delegation

## Problem

You have a complex initiative — say, building a new feature — that requires multiple skills: backend development, frontend work, testing, and deployment. No single agent has all the capabilities, and sequential handoffs are slow.

## Solution

Use a **coordinator agent** that decomposes the work into subtasks and delegates them to specialized agents in parallel. The coordinator tracks progress, resolves blockers, and assembles the final result.

### Architecture

```
         CEO / Human
              |
        [Coordinator Agent]
         /        |        \
   [Backend]  [Frontend]  [DevOps]
     Agent      Agent      Agent
```

### How It Works

1. **Decompose**: The coordinator receives a high-level directive and breaks it into subtasks with clear boundaries and deliverables.
2. **Delegate**: Each subtask is assigned to the agent best suited for it, with context about dependencies.
3. **Track**: The coordinator monitors progress via task status polling or event subscriptions.
4. **Resolve**: When an agent reports a blocker, the coordinator either reassigns or escalates.
5. **Assemble**: Once all subtasks complete, the coordinator verifies integration and reports back.

### Implementation with the SDK

```python
from agent_ceo import AgentCEOClient

async def delegate_feature(client: AgentCEOClient, org_id: str):
    # Step 1: Create subtasks for each specialist
    backend_task = await client.assign_task(
        org_id, agent_id="backend-agent",
        description="Implement the /users API endpoints with auth middleware",
        priority="high",
    )

    frontend_task = await client.assign_task(
        org_id, agent_id="frontend-agent",
        description="Build the user management UI pages using the /users API",
        priority="high",
        metadata={"depends_on": backend_task["id"]},
    )

    # Step 2: Poll for completion
    for task_id in [backend_task["id"], frontend_task["id"]]:
        task = await client.get_task(org_id, task_id)
        while task["status"] not in ("completed", "verified", "failed"):
            await asyncio.sleep(10)
            task = await client.get_task(org_id, task_id)
```

### Best Practices

- **Keep subtasks atomic**: Each subtask should be independently completable and testable.
- **Declare dependencies explicitly**: Use metadata to mark which tasks depend on others so the coordinator can sequence correctly.
- **Set SLAs**: Assign deadlines so the coordinator can escalate before the overall timeline slips.
- **Limit coordinator scope**: The coordinator should orchestrate, not implement. If it starts writing code, the decomposition was too coarse.

### When to Use

- Features that span multiple codebases or skill domains
- Sprint-level planning where a tech lead agent coordinates the team
- Incident response where multiple agents investigate different subsystems in parallel

### Related

- [Agent Meeting Protocol](./agent-meeting-protocol.md) — for synchronization between agents during delegation
- [Knowledge-Base-Driven Agents](./knowledge-base-driven-agents.md) — for sharing context across delegated agents
