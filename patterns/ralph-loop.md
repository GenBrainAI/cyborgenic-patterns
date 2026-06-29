# Ralph Loop (Fresh-Context Task Drain)

## Problem

You have a list of work — a backlog, a PRD, a TODO — and you want an agent to grind through it autonomously, end to end, without you babysitting each step. But a single long-running agent session degrades: as its context window fills with prior turns, diffs, and dead ends, its judgment rots and it starts repeating itself. You want **many reliable small steps**, not one heroic session that slowly loses the plot.

## Solution

Run the agent in a tight outer loop where **every iteration starts from a fresh context window** and does exactly **one task**. State does not live in the conversation — it lives **outside**, in the codebase, a progress file, and git history. The loop reloads that external state each pass, picks the next task, finishes it, records the result, and exits — then the loop runs it again, fresh.

Named for Geoffrey Huntley's "Ralph Wiggum loop": *deterministically bad in an undeterministic world* — you embrace cheap, disposable iterations instead of one fragile long-lived one.

### Architecture

```
   while not done:
   ┌─────────────────────────────────────────────┐
   │  FRESH context window  (no prior turns)      │
   │   1. read  PROGRESS.md + the task list       │
   │   2. pick  the highest-priority OPEN task    │
   │   3. implement it fully                       │
   │   4. run tests / typecheck  (objective gate)  │
   │   5. update PROGRESS.md, mark task done       │
   │   6. commit to git                            │
   └─────────────────────────────────────────────┘
        state persists in:  repo · PROGRESS.md · git history
        stop when:          task list is empty  (or budget hit)
```

### How It Works

1. **Externalize the memory.** The loop's "memory" is files, not chat: the repo, a `PROGRESS.md` (what's done / what's next / lessons learned), and the git log. This is the whole trick — the agent is stateless; the repo is the state.
2. **Fresh context every pass — on purpose.** Each iteration boots clean and rereads the external state. Context never accumulates, so judgment never degrades. (This is *the point*, not a side effect.)
3. **One task per iteration.** Pick the single highest-priority open item, implement it completely, and stop. Small blast radius per pass.
4. **Gate on something objective.** Run the test suite / type-check / build before marking a task done. No green gate, no "done."
5. **Record and commit.** Update `PROGRESS.md` and commit. The next iteration sees the new ground truth.
6. **Drain to empty.** Loop until the task list is exhausted (the stop condition), a hard budget/iteration cap trips, or a human stops it.

The **standing high-level spec** (a `VISION.md` / charter) is reread each pass and is *not* editable by the loop — it keeps the drain from drifting off-goal.

### Pitfalls

- **Soft completion (the Ralph-Wiggum failure mode).** Without a real objective gate, the agent emits "done" on a half-finished task and the loop drains a list of lies. The test/build gate is mandatory.
- **No hard stop.** A task list that's never truly "empty" (or a regenerating one) runs forever and burns budget. Cap iterations / tokens / wall-clock.
- **Don't parallelize the inner loop.** Ralph is a *single* process on a *single* repo. Concurrency on the same checkout is "a red hot mess" — parallelize by running *separate* Ralph loops in *isolated* workspaces, not by threading one.

### Relationship to other patterns

- Ralph is the **primitive build engine**. It is the ideal **Observed builder** inside an [Observable Loop](./observable-loop.md): the observer watches a Ralph builder drain its list and directs a swap when evidence shows a better approach. (Engine inside a transmission.)
- It is the disciplined sibling of [Agentic Recursion](./agentic-recursion.md): recursion fans *out* across agents; Ralph drives *forward* through a list, one fresh pass at a time.

### Running it as a managed loop (agent.ceo)

Ralph is a pattern plugin on the shared loop runtime. The platform supplies the outer loop, the external state store, the objective gate, and a hard budget as a first-class stop reason. The one capability it specifically needs from the runtime is a **per-step context reset** (`reset_context_each_step: true`) — the mechanism that makes "fresh context every iteration" real. Each Ralph instance runs in an isolated operand so many can drain different backlogs in parallel without colliding.
