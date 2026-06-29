# Cyborgenic Organization Patterns

**Design patterns for building AI-native organizations with [agent.ceo](https://agent.ceo)**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

---

## What is a Cyborgenic Organization?

A **cyborgenic organization** is an AI-native operating model where human leaders and AI agents work together as a unified team. Instead of bolting AI onto existing workflows, a cyborgenic organization is designed from the ground up with agents as first-class participants — they have roles, responsibilities, communication channels, and accountability, just like human team members. The [agent.ceo](https://agent.ceo) platform provides the infrastructure to build, manage, and scale these organizations.

## Patterns

Reusable design patterns for common cyborgenic workflows:

| Pattern | Description |
|---------|-------------|
| [Multi-Agent Task Delegation](./patterns/multi-agent-task-delegation.md) | How to decompose work and delegate across a team of specialized agents |
| [Knowledge-Base-Driven Agents](./patterns/knowledge-base-driven-agents.md) | Using shared knowledge bases to give agents organizational context |
| [Enterprise Discovery & Onboarding](./patterns/enterprise-discovery-onboarding.md) | Patterns for discovering what agents an enterprise needs and onboarding them |
| [Agent Meeting Protocol](./patterns/agent-meeting-protocol.md) | Structured communication between agents for decision-making and sync |
| [Observable Loop (Observer / Observed)](./patterns/observable-loop.md) | A standing critic (observer) judges builders' solutions against better alternatives on evidence and drives swaps, looping to convergence — without writing the code itself |
| [Agentic Recursion (Delegate-and-Return)](./patterns/agentic-recursion.md) | An agent calls an agent of its choice, which calls another, … until a stop condition; the result returns up the stack. Pausable/stoppable, editable stop-condition + choice-set |
| [Ralph Loop (Fresh-Context Task Drain)](./patterns/ralph-loop.md) | The primitive build engine: fresh context every iteration, one task per pass, state externalized to repo + progress file + git; drain a task list to empty. Composes as the Observed builder inside an Observable Loop |
| [AutoResearch Loop (Propose · Run · Measure · Keep-or-Revert)](./patterns/autoresearch-loop.md) | The primitive search engine (Karpathy AutoResearch / self-improving): an LLM edits code, runs a real experiment, scores one objective metric, keeps on improve else reverts; swarm in parallel operands, observer promotes winners to scale |

## Templates

Ready-to-use CLAUDE.md configuration templates for common agent roles:

| Template | Role |
|----------|------|
| [Backend Python](./templates/backend-python/CLAUDE.md) | Python backend engineer agent |
| [Fullstack TypeScript](./templates/fullstack-ts/CLAUDE.md) | Full-stack TypeScript/React agent |
| [DevOps](./templates/devops/CLAUDE.md) | Infrastructure and deployment agent |

## Examples

End-to-end examples of cyborgenic organizations:

| Example | Description |
|---------|-------------|
| [Simple Org](./examples/simple-org/) | Minimal 3-agent organization for a SaaS startup |

## Getting Started

1. **Explore the patterns** — Read through the patterns to understand common approaches
2. **Pick a template** — Start with a template that matches your agent's role
3. **Configure your org** — Use the examples as a starting point for your organization config
4. **Deploy on agent.ceo** — Sign up at [agent.ceo](https://agent.ceo) and deploy your agents

## Resources

- **agent.ceo Platform**: [https://agent.ceo](https://agent.ceo)
- **Developer Docs**: [https://agent.ceo/developers/docs](https://agent.ceo/developers/docs)
- **Python SDK**: [agent-ceo-sdk](https://github.com/GenBrainAI/agent-ceo-sdk)

## Contributing

We welcome contributions! Whether it is a new pattern, an improved template, or a full example:

1. Fork this repository
2. Create a feature branch: `git checkout -b feat/my-pattern`
3. Add your content following the existing structure and conventions
4. Submit a pull request with a clear description

Please keep patterns practical and include code examples where applicable.

## License

This project is licensed under the MIT License — see the [LICENSE](./LICENSE) file for details.

---

<p align="center">
  Built by <a href="https://genbrain.ai">GenBrain AI</a> — powering cyborgenic organizations with <a href="https://agent.ceo">agent.ceo</a>
</p>
