# AutoResearch Loop (Propose · Run · Measure · Keep-or-Revert)

## Problem

You have a goal that's measured by a **number** — make this model train better, make this query faster, make this prompt convert higher — and a huge space of possible changes, most of which won't help. You can't enumerate the options in advance (the good ideas are *code*, not a dropdown of hyperparameters), and you can't eyeball "better" — you have to *run it and measure*. You want an agent to search that space empirically, overnight, keeping only what actually moves the metric.

## Solution

Give the agent a small but **real** harness it can run, and an **objective metric**. Then loop: the agent proposes an idea, edits the code, runs the experiment under a fixed budget, reads the metric, and **keeps the change only if the metric improved — otherwise it reverts to the previous state** and tries something else. Winners accumulate in git history; losers leave no trace.

This is Andrej Karpathy's *AutoResearch*: an LLM performs the search **by editing the code directly** (open-ended), rather than selecting from a predefined space like classical AutoML. In his run, ~630 lines of training code + one markdown charter produced **700 experiments in 2 days and ~20 real optimizations**.

### Architecture

```
   charter (program.md, immutable)  ── "research org code, in English"
            │
   loop:    ▼
   ┌──────────────────────────────────────────────┐
   │ 1. PROPOSE  an idea (novel code change)        │
   │ 2. EDIT     train.py / the target              │
   │ 3. RUN      experiment, fixed time budget       │
   │ 4. MEASURE  one objective metric                │
   │ 5. KEEP if improved → commit                    │
   │    else REVERT (git reset) to last good state   │
   └──────────────────────────────────────────────┘
        winners → git history     stop: target / budget / K dry rounds
```

### How It Works

1. **Immutable charter.** A natural-language spec (`program.md`) defines the goal, the metric, and the rules. The agent reads it every pass but **cannot edit it** — the charter keeps the search honest.
2. **Propose by editing code.** The agent doesn't pick from a menu; it writes the change — a new architecture tweak, a different schedule, a refactor. The LLM *is* the search operator.
3. **Run in a real, time-boxed harness.** Each experiment actually executes, under a per-run compute/time cap so a bad idea fails fast.
4. **Score on one objective metric.** Extract a single number from the run (e.g. validation bits-per-byte, latency, accept-rate). One metric = an unambiguous keep/discard decision.
5. **Keep-or-revert (hill-climb).** Improved → commit and build on it. Worse-or-equal → reset to the last good state. The repo always reflects the current best.
6. **Stop sensibly.** Halt on: metric target reached · budget exhausted · **no improvement for K consecutive rounds** (loop-until-dry). Otherwise it explores indefinitely.

### Pitfalls

- **Gameable / proxy metric.** If the metric doesn't capture what you actually want, the loop will optimize the proxy and degrade the real goal. Choose the metric like it's a contract.
- **Unbounded compute.** Each experiment costs real money; without a per-run budget *and* a loop-level cap, a swarm gets expensive fast.
- **Overfitting to the eval.** Hold out a separate check; periodically validate winners on data the loop never optimized against.
- **Not AutoML.** Don't constrain the agent to a fixed search space — the value is open-ended code edits. (But keep the harness small and fast so iterations are cheap.)

### Relationship to other patterns

- AutoResearch is the **primitive search/optimize engine** — and the concrete form of a **self-improving loop**: a component that changes, runs, and tests *itself* against a metric.
- Put an [Observable Loop](./observable-loop.md) **over a swarm** of AutoResearch loops and the observer **promotes winning experiments to larger scale** — Karpathy's own stated future ("a swarm of agents, promote the most promising ideas to increasingly larger scales"). Engine × transmission.
- Complements [Ralph](./ralph-loop.md): Ralph *drains a known list*; AutoResearch *discovers what's worth doing* by measurement.

### Running it as a managed loop (agent.ceo)

A pattern plugin on the shared loop runtime. Each experiment runs in its **own isolated operand** (sandboxed, time-boxed, independently billed) — so the swarm is just N operands exploring in parallel, with results shared and winners promoted. The spec adds three fields: `objective_metric` (how to score a run), `keep_rule` (improve→keep / else→revert), and `experiment_budget` (per-run cap). Per-loop and per-experiment budgets are first-class stop reasons, which is what makes an unattended overnight search safe to sell.
