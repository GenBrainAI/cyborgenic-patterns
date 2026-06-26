# Observable Loop (Observer / Observed)

## Problem

You want a feature built *well*, but you don't want to dictate *how*. A single agent handed a detailed spec tends to lock onto its first working approach — a local optimum — and stop improving. A reviewer who rubber-stamps diffs only checks "is this correct," never "is there a materially **better** solution than this one?" You want continuous convergence toward the best implementation **without writing the code or the detailed design yourself**.

## Solution

Split the organization into two roles and run a continuous loop between them:

- **Observed (builders)** — the agents that own features. Each receives a **rough** spec ("here's roughly what I want — implement it however you like") and builds freely, owning the feature's whole lifecycle.
- **Observer** — a single standing critic (e.g. the CEO) that **never builds**. It runs one question over the whole fleet, forever: *"Is there a better coding solution than what's been done so far — and if so, **what**, and **when** do we swap it in?"* — and decides on **evidence**.

The observer doesn't approve diffs; it drives builders off local optima toward "better," looping until the work converges. The human **observes and queries** rather than manages.

### Architecture

```
              Founder / Human
                    |
              [ Observer ]   ── single standing critic; does NOT build
             /     |      \     "is there a better solution? what / when?"
     [Builder]  [Builder]  [Builder]        ← the Observed
      (rough     (rough     (rough           each owns a feature,
       spec)      spec)      spec)            builds freely, stays
         ↑_________ observe + direct swap _________↑   observable + queryable
```

### How It Works

1. **Rough spec in.** The observer (or human) hands a builder a deliberately rough spec and gets out of the way. Roughness is the point — it leaves room for the builder's judgment and the observer's job.
2. **Build freely.** The builder implements any way it likes. It commits early and often, pushes, and logs evidence (tests, tokens, time).
3. **Observe.** The observer watches the *actual* work — commits/PRs, task progress, the builder's own answers — plus the per-task **token + time** accounting.
4. **Judge current-vs-better.** Each cycle the observer weighs the current solution against *alternatives* on evidence: correctness, tokens, time-to-result, simplicity, fit-to-spec.
5. **Decide & direct.** Verdict is **keep + refine** or **replace-with-better** (what + when). The observer *directs* the swap — it never writes the code.
6. **Loop to convergence.** Repeat until no better solution surfaces for K rounds, the rough spec is met, and tests pass. Then the **observer**, not the builder, calls "done."

Queries flow sideways: a question about a feature routes to its builder, which answers however it likes — it is the **live interface** to its feature.

### Implementation with the SDK

```python
from agent_ceo import AgentCEOClient

# Observer loop (pseudocode) — runs continuously over a builder
spec = "Make the central registry actually populate + serve agents/MCPs — your call how."
task = await client.assign_task(org_id, agent_id="builder",
                                description=spec, priority="high")

dry_rounds = 0
while dry_rounds < K:                                   # K consecutive "no better" rounds = converged
    work    = await client.get_task(org_id, task["id"])     # progress notes + status
    diff    = git_show(work["branch"])                       # the ACTUAL code, not the claim
    metrics = await client.get_task_usage(org_id, task["id"])# tokens + wall-clock time

    verdict = judge(diff, metrics, spec)                # current vs. alternatives, on EVIDENCE
    if verdict.has_better:
        await client.send_to_agent(org_id, "builder",
            f"Swap to: {verdict.better}. Evidence: {verdict.why}. When: {verdict.when}.")
        dry_rounds = 0
    else:
        dry_rounds += 1
    await asyncio.sleep(cycle)
# Converged when: no better solution for K rounds AND rough spec met AND tests pass.
```

The reference implementation ships as two skills — **`observer`** and **`observed`** — in
[agent-hub-skills](https://github.com/GenBrainAI/agent-hub-skills). Install `observer` on the
critic (CEO) and `observed` on every builder agent.

### Best Practices

- **Keep specs rough on purpose.** A detailed spec pre-empts the builder's judgment and the observer's job. Say *what*, not *how*.
- **The observer never touches the code.** Its leverage is judgment, not keystrokes. If it starts implementing, the roles have collapsed.
- **"Better" is evidence, not opinion.** Wire per-task **token + time** accounting plus tests — that is the observer's ruler. No evidence → it is guessing.
- **Make work observable.** Unpushed, unlogged work is invisible, and invisible work cannot converge. Builders commit small and often.
- **Both sides can overrule — on evidence.** A builder may refute a swap with better numbers; the observer may insist with its own. Disagreement resolves on data, not authority.
- **Define convergence explicitly.** Done = no better solution for K rounds + spec met + tests pass — never self-declared by the builder.
- **Stay harness-agnostic.** The observer judges the *solution*, not the tool that made it, so builders can run different harnesses (Claude Code, Codex, Gemini CLI, …) without changing the loop.

### When to Use

- You want continuous quality convergence on a feature without micromanaging the approach.
- Many builders working in parallel, with one standing critic keeping them off local optima.
- A human wants to **observe and query** a feature rather than manage its implementation.
- Long-running or large builds where first-draft solutions should be replaceable as better ones emerge.

### Avoid When

- One-off, throwaway work where convergence isn't worth the observer's overhead.
- Hard real-time tasks with no room to iterate-and-swap.

### Related

- [Multi-Agent Task Delegation](./multi-agent-task-delegation.md) — decomposition and delegation; the observable loop adds a *standing critic* on top of delegated work.
- [Agent Meeting Protocol](./agent-meeting-protocol.md) — structured sync between agents, complementary to the observer's continuous watch.
- `observer` / `observed` skills in [agent-hub-skills](https://github.com/GenBrainAI/agent-hub-skills) — the reference implementation of this pattern.
