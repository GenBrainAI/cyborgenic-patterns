# Enterprise Discovery & Onboarding

## Problem

When an enterprise adopts agent.ceo, the first question is: "What agents do we need?" Deploying the wrong mix of agents wastes resources, while missing critical roles creates bottlenecks. Enterprises need a systematic way to discover the right agent composition and onboard agents effectively.

## Solution

Use a **discovery phase** to analyze the enterprise's workflows and roles, then systematically onboard agents that map to real organizational needs.

### The Discovery Process

#### Phase 1: Workflow Audit

Start by mapping existing workflows to identify where agents can add the most value:

```yaml
# discovery-audit.yaml
workflows:
  - name: "PR Review Cycle"
    current_bottleneck: "Reviews take 2-3 days"
    agent_opportunity: "Automated first-pass code review"
    suggested_role: "code-reviewer"
    
  - name: "Incident Response"
    current_bottleneck: "On-call triage is slow at night"
    agent_opportunity: "Automated log analysis and initial triage"
    suggested_role: "incident-responder"
    
  - name: "Documentation Updates"
    current_bottleneck: "Docs drift from code within weeks"
    agent_opportunity: "Auto-generate and update docs on code changes"
    suggested_role: "docs-maintainer"
```

#### Phase 2: Role Definition

For each discovered role, define the agent's responsibilities, capabilities, and knowledge requirements:

```yaml
agent_roles:
  code-reviewer:
    responsibilities:
      - First-pass review of all pull requests
      - Check for security vulnerabilities
      - Enforce coding standards
    knowledge_bases: ["coding-standards", "security-patterns"]
    template: "backend-python"  # or "fullstack-ts" depending on stack
    
  incident-responder:
    responsibilities:
      - Monitor alerting channels
      - Triage incoming incidents
      - Execute runbook procedures
    knowledge_bases: ["runbooks", "infrastructure-docs"]
    template: "devops"
```

#### Phase 3: Staged Rollout

Do not deploy all agents at once. Use a staged approach:

1. **Pilot** (Week 1-2): Deploy 1-2 agents in a low-risk area. Measure output quality.
2. **Expand** (Week 3-4): Add agents for the highest-impact bottlenecks identified in the audit.
3. **Scale** (Month 2+): Fill remaining roles and optimize based on observed performance.

### Onboarding Checklist

For each new agent:

- [ ] Define role and responsibilities in a CLAUDE.md template
- [ ] Attach relevant knowledge bases with appropriate access levels
- [ ] Set up communication channels (which agents/humans it reports to)
- [ ] Assign an initial low-risk task to validate configuration
- [ ] Review the agent's first 5 task completions for quality
- [ ] Adjust capabilities and knowledge access based on observed gaps

### Best Practices

- **Start with pain points**: Deploy agents where the current process is slowest or most error-prone.
- **Pair agents with humans initially**: Have a human review agent output for the first week to catch configuration issues early.
- **Measure before and after**: Track cycle time, error rate, and throughput to quantify impact.
- **Iterate on templates**: The first CLAUDE.md for a role will not be perfect. Refine based on actual agent performance.

### Anti-Patterns

- **The Big Bang**: Deploying 20 agents on day one without validation
- **Copy-Paste Roles**: Using the same generic template for every agent instead of tailoring to the role
- **No Knowledge**: Deploying agents without attaching relevant knowledge bases
