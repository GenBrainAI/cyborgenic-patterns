# Agent Meeting Protocol

## Problem

When multiple agents work on related tasks, they need to synchronize — share status updates, resolve conflicts, make joint decisions, and align on priorities. Without a structured protocol, agents either duplicate work, make conflicting decisions, or fail to share critical context.

## Solution

Use **structured agent meetings** — time-boxed, agenda-driven communication sessions between two or more agents. Meetings follow a defined protocol that ensures productive outcomes without unbounded back-and-forth.

### Meeting Types

| Type | Participants | Frequency | Purpose |
|------|-------------|-----------|---------|
| **Standup** | All agents in a team | Daily | Status sync, blocker identification |
| **Design Review** | CTO + relevant agents | As needed | Architecture decisions |
| **Incident Sync** | Incident responders | During incidents | Coordinate response |
| **Sprint Planning** | Coordinator + team | Per sprint | Task decomposition and assignment |

### Protocol Structure

Every meeting follows this sequence:

```
1. CONVENE    — Meeting creator defines agenda and invites participants
2. ROLL CALL  — Each participant acknowledges and shares brief status
3. AGENDA     — Work through agenda items in order
4. DECISIONS  — Record any decisions made with rationale
5. ACTIONS    — Assign action items with owners and deadlines
6. ADJOURN    — Meeting creator closes the meeting and distributes summary
```

### Implementation

```python
from agent_ceo import AgentCEOClient

async def run_standup(client: AgentCEOClient, org_id: str):
    # Create a standup meeting task for the coordinator
    standup = await client.assign_task(
        org_id,
        agent_id="coordinator",
        description=(
            "Run daily standup. Collect status from: backend-agent, "
            "frontend-agent, devops-agent. Identify blockers. "
            "Record decisions and action items."
        ),
        priority="normal",
        metadata={
            "meeting_type": "standup",
            "participants": ["backend-agent", "frontend-agent", "devops-agent"],
            "agenda": [
                "Yesterday's progress",
                "Today's plan",
                "Blockers",
            ],
        },
    )
```

### Message Format

Agents communicate during meetings using structured messages:

```json
{
  "meeting_id": "mtg-2026-05-11-standup",
  "from": "backend-agent",
  "type": "status_update",
  "content": {
    "completed": ["Implemented /users endpoint", "Added JWT validation"],
    "planned": ["Write integration tests for auth flow"],
    "blockers": [],
    "needs_from_others": ["Frontend: API contract for user profile page"]
  }
}
```

### Decision Records

When a meeting produces a decision, record it formally:

```json
{
  "meeting_id": "mtg-2026-05-11-design",
  "decision": "Use PostgreSQL row-level security instead of application-level auth checks",
  "rationale": "Reduces attack surface and centralizes access control logic",
  "decided_by": ["cto-agent", "backend-agent"],
  "action_items": [
    {"owner": "backend-agent", "task": "Implement RLS policies for users table", "deadline": "2026-05-13"}
  ]
}
```

### Best Practices

- **Time-box meetings**: Set a maximum number of message rounds (e.g., 3 rounds for a standup). Prevent unbounded discussion.
- **Require structure**: Use typed message formats so agents can parse each other's updates programmatically.
- **Record everything**: Every decision and action item should be recorded. Agents forget context between sessions; the record is the memory.
- **Keep participants minimal**: Only invite agents who need to participate. Broadcast summaries to the rest.
- **Async by default**: Most meetings should be asynchronous message exchanges, not real-time. Reserve synchronous meetings for time-sensitive decisions.

### When to Use

- Teams with 3+ agents working on related tasks
- Any situation where two agents might make conflicting decisions
- Sprint boundaries where priorities need to be aligned
- Incident response where multiple agents coordinate under time pressure
