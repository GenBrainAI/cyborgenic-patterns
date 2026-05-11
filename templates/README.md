# Agent Templates

Ready-to-use CLAUDE.md configuration templates for common agent roles. Copy a template, customize it for your organization, and deploy.

## Available Templates

| Template | Role | Stack |
|----------|------|-------|
| [Backend Python](./backend-python/CLAUDE.md) | Python backend engineer | Python, FastAPI, PostgreSQL |
| [Fullstack TypeScript](./fullstack-ts/CLAUDE.md) | Full-stack developer | TypeScript, React, Node.js |
| [DevOps](./devops/CLAUDE.md) | Infrastructure & deployment | Kubernetes, Terraform, CI/CD |

## How to Use

1. Copy the relevant `CLAUDE.md` into your agent's configuration
2. Replace placeholder values (`<ORG_NAME>`, `<BRANCH>`, etc.) with your actual values
3. Add or remove capabilities based on your agent's specific needs
4. Attach relevant knowledge bases for organizational context

## Customization Guide

Each template has sections you should customize:

- **Role & Manager**: Set the agent's reporting line
- **Tools**: List the specific tools available in your environment
- **Capabilities**: Tailor to the agent's actual responsibilities
- **Git Workflow**: Match your organization's branching strategy
- **Quality Standards**: Align with your team's conventions
