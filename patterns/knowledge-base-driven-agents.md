# Knowledge-Base-Driven Agents

## Problem

Agents need organizational context — coding standards, architecture decisions, runbooks, domain knowledge — to produce quality work. Without this context, agents make decisions in a vacuum, leading to inconsistency and rework.

## Solution

Use **shared knowledge bases** to give agents access to curated organizational knowledge. Knowledge bases act as a persistent, searchable memory that agents can query before making decisions.

### Architecture

```
  [Knowledge Bases]
   /       |       \
  KB:      KB:      KB:
  Docs   Runbooks  ADRs
   \       |       /
    [Agent Pool]
```

Each agent can have one or more knowledge bases attached with different access levels:
- **read** — The agent can query the KB for context
- **write** — The agent can add new entries (e.g., post-incident reports)
- **admin** — The agent can manage KB structure and permissions

### How It Works

1. **Curate**: Populate knowledge bases with relevant documents — architecture decision records, API docs, coding standards, incident post-mortems.
2. **Attach**: Attach relevant KBs to each agent role. A backend agent gets the API docs and coding standards; a DevOps agent gets runbooks and infrastructure docs.
3. **Query**: Agents automatically query attached KBs for context when working on tasks. The platform uses vector search to surface relevant documents.
4. **Update**: Agents with write access contribute back — adding post-mortems, documenting new patterns, or updating outdated docs.

### Implementation

```python
from agent_ceo import AgentCEOClient

async def setup_agent_knowledge(client: AgentCEOClient, org_id: str):
    # List available knowledge bases
    kbs = await client.list_knowledge_bases(org_id)
    
    # Attach the coding standards KB to the backend agent (read-only)
    await client.attach_kb(
        org_id,
        agent_id="backend-agent",
        kb_id="kb-coding-standards",
        access="read",
    )

    # Attach the post-mortems KB with write access so the agent
    # can contribute new post-mortems after incidents
    await client.attach_kb(
        org_id,
        agent_id="backend-agent",
        kb_id="kb-post-mortems",
        access="write",
    )
```

### Knowledge Base Types

| Type | Contents | Typical Access |
|------|----------|---------------|
| **Coding Standards** | Style guides, linting rules, architectural patterns | Read for all dev agents |
| **Architecture Decision Records** | Past decisions with rationale and trade-offs | Read for all agents |
| **Runbooks** | Step-by-step operational procedures | Read for DevOps, write for incident responders |
| **Domain Knowledge** | Business rules, domain models, glossaries | Read for all agents |
| **Post-Mortems** | Incident analyses and lessons learned | Write for incident agents, read for all |

### Best Practices

- **Keep KBs focused**: One knowledge base per domain. Do not dump everything into a single KB.
- **Review regularly**: Stale knowledge is worse than no knowledge. Schedule periodic reviews.
- **Use access levels intentionally**: Not every agent needs write access to every KB.
- **Let agents contribute**: Agents that solve novel problems should document the solution back into the KB.

### When to Use

- Any organization with more than two agents that need to stay consistent
- When onboarding new agents that need to learn the codebase quickly
- When the same questions keep coming up across different agents
