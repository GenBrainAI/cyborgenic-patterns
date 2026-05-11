# Simple Organization Example

A minimal cyborgenic organization for a SaaS startup with three agents.

## Organization Structure

```
        CEO (Human)
            |
       CTO (Agent)
        /       \
  Backend     Fullstack
  (Agent)      (Agent)
```

## Agents

| Agent | Role | Responsibilities |
|-------|------|-----------------|
| **CTO** | Technical Lead | Architecture decisions, PR reviews, task coordination |
| **Backend** | Backend Engineer | API development, database design, testing |
| **Fullstack** | Fullstack Developer | UI implementation, API integration, E2E tests |

## Knowledge Bases

| KB | Contents | Attached To |
|----|----------|-------------|
| `coding-standards` | Style guides, patterns | All agents (read) |
| `architecture-docs` | ADRs, system design | CTO (read/write), others (read) |
| `runbooks` | Operational procedures | CTO (read) |

## Usage

1. Review `org-config.yaml` below
2. Customize the agent names, templates, and knowledge bases
3. Deploy via the agent.ceo dashboard or use the SDK:

```python
import yaml
from agent_ceo import AgentCEOClient

async def deploy_org(client: AgentCEOClient):
    with open("org-config.yaml") as f:
        config = yaml.safe_load(f)

    for agent_cfg in config["agents"]:
        agent = await client.create_agent(
            org_id=config["org"]["id"],
            role=agent_cfg["role"],
            name=agent_cfg["name"],
            template=agent_cfg["template"],
        )
        # Attach knowledge bases
        for kb in agent_cfg.get("knowledge_bases", []):
            await client.attach_kb(
                org_id=config["org"]["id"],
                agent_id=agent["id"],
                kb_id=kb["id"],
                access=kb["access"],
            )
```
