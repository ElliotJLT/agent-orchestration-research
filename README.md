# Agent Orchestration Patterns

Distilled patterns for scaling multi-agent coding workflows. Source: research into emerging orchestration approaches (Jan 2026).

## Core Architecture Concepts

### Sessions are cattle, agents are identities
- Separate persistent identity (role, work queue, state) from ephemeral sessions
- Store identity and work state externally (Git-backed), not in the session
- Sessions crash, compact, expire — the work persists

### Micro-workflow decomposition (MEOW pattern)
- Break work into small sequenced steps stored as linked issues/tasks
- Each step has clear acceptance criteria
- If agent crashes, next session picks up at the current step
- Workflows survive agent failures because state is in Git, not in context

### Nondeterministic Idempotence
- Path is nondeterministic (agents may take different routes), but outcome is deterministic
- As long as you keep throwing sessions at work, the workflow eventually finishes
- Agent can self-correct because acceptance criteria are well-specified

### GUPP (persistent hook principle)
- Every agent has a "hook" — persistent work assignment
- On session start, agent checks its hook and continues working without prompting
- Work survives crashes, compactions, restarts, interruptions

## Worker Roles (factory model)

| Role | Purpose |
|------|---------|
| Mayor | Primary concierge, delegates work, kicks off convoys |
| Polecats | Ephemeral swarm workers, spin up on demand, self-destruct after task |
| Refinery | Merge queue manager, handles intelligent merging one-at-a-time |
| Witness | Monitors polecats, helps stuck workers, runs rig-level plugins |
| Deacon | Daemon patrol, hierarchical heartbeat, propagates "do your job" signal |
| Dogs | Deacon's helpers for maintenance and handyman tasks |
| Crew | Named long-lived agents, personal concierges for design/review work |

Key insight: you don't need all roles. Start with just Mayor (one agent that delegates). Add complexity as needed. System degrades gracefully.

## The Crew Cycling Pattern (most replicable today)

1. Set up N named sessions (terminals/tmux panes), one per task
2. Cycle through each, give it a meaty task, then LEAVE IT ALONE
3. Keep cycling — when one finishes, read its output carefully, act on it
4. Approve, redirect, or hand off new work
5. Repeat

Key disciplines:
- Don't watch agents work — read their results
- Spend time on direction, not observation
- It's a slot-machine pattern: spin all up, harvest results

## PR Sheriff (standing orders pattern)

- Designate one agent with permanent "standing orders" (pinned task)
- On every startup, it checks open PRs, triages into easy-wins vs needs-human-review
- Handles easy ones automatically, flags complex ones for you
- Replicable with CLAUDE.md instructions or session hooks

## Three Development Cadences

- **Outer loop** (days/weeks): upgrades, learning tools, establishing workflow
- **Middle loop** (hours/days): filing work, delegating to agents, cleanups
- **Inner loop** (minutes): reading agent output, giving direction, handoffs

## Invisible Garden Maintenance

Since you can't read every line of vibe-coded output:
- Run regular **code review sweeps** (agents review code, file issues)
- Follow with **bug-fix sweeps** (agents fix the filed issues)
- Alternate until reviews are just nitpicks
- Do some of this every day

## Heresies (propagating wrong assumptions)

- Agents guess wrong about how the system works
- Wrong guess gets into code/comments → other agents pick it up → propagates
- Fix: capture core principles/axioms in agent priming (CLAUDE.md)
- More principle coverage = fewer classes of wrong-guess
- Example: "Zero Framework Cognition", "Discovery over Tracking"

## Desire Paths (agent UX design)

- Tell agent what you want, watch what it TRIES to do
- Implement the thing it tried — make it real
- Repeat until tool works the way agents naturally expect
- General design principle for any agent-friendly tooling

## Handoff as Core Primitive

- Agent wraps up state, writes notes for successor, restarts cleanly
- Building reliable handoff is the single highest-leverage thing for sustained agent work
- Hand off liberally — don't let sessions run long unless accumulating important context
- Multiple mechanisms: verbal ("let's hand off"), command, mechanical

## Rule of Five (Jeffrey Emanuel)

- Have LLM review something 5 times with different focus areas each pass
- Implementation counts as review #1
- Composable as a wrapper around any workflow step
- Generates superior outcomes and artifacts

## Colony > Super-ant

- Don't optimize for one agent doing everything perfectly
- Optimize for many agents doing small things reliably with coordination
- Throughput beats perfection
- "Nature prefers colonies" — focus on coordination infrastructure

## Practical Observations

- **Go is good for vibe-coded systems code** — boring diffs, readable, less token waste than TypeScript
- **tmux is surprisingly powerful** as multi-agent UI — learn one new binding per day
- **Convoys** = feature-level work orders that roll up swarm activity into trackable units
- **Exponential backoff** for patrol/monitor loops — sleep longer when no work, wake on mutation
- **Transparency rule**: when moving fast with multiple agents, everything must be announced and visible in real-time or it's invisible. "2 hours ago is ancient."
- **Serial bottlenecks are normal** — it's OK to run 1-2 workers while others pause
- **Regular cleanups** — stale branches, orphaned files, dead state. Automate the recurring ones.

## Orchestrator Taxonomy

Four orthogonal problems in multi-agent coding:

| Problem | Focus | Example |
|---------|-------|---------|
| **Quality** | Self-review loops until convergence | Ralph Wiggum loops |
| **Quantity** | Swarm ephemeral workers at well-defined tasks | Gas Town polecats |
| **Coordination** | Multi-agent collaboration on complex tasks | Loom, Claude Flow |
| **Work definition** | Durable audit trail of what needs doing | MEOW/Beads |

Key insight: these are complementary, not competing. The ideal system combines durable workflows (patrols) that invoke quality-convergence loops (Ralph) at each step, with coordination layers for complex multi-agent steps.

### Task sizing for ephemeral workers
- Ephemeral agents choke on tasks that are too big
- Context compaction = failure scenario
- Break work down small enough that an agent can finish in one session
- Same old agile decomposition principle, new context window constraint

### Roles as config, not code (Gas City concept)
- Once the workflow stack is powerful enough, roles themselves can be expressed as DSL/config
- Build-your-own-orchestrator-shape rather than hardcoded roles
- Allows custom business processes, team structures, coordination rules

## Cognitive Load at Scale

Working at high decision-throughput with many agents exhausts internal buffers. "Bezos Mode" — when easy work is handled automatically, everything you personally process is architecturally complex and nuanced. Multiple practitioners independently report needing deep recovery naps during sustained multi-agent sessions. The hypothesis: sustained high-bandwidth decision-making depletes cognitive resources faster than normal coding.

## Software Survival Framework (Selection Model)

Framework for predicting which software survives in a world where AI writes all code.

### Core thesis
Inference costs tokens, which cost energy, which cost money. Resource constraints create selection pressure: **software survives if it saves cognition**.

### The Survival Ratio

```
Survival(T) ∝ (Savings × Usage × H) / (Awareness_cost + Friction_cost)
```

- **Savings**: tokens saved vs synthesizing equivalent functionality from scratch
- **Usage**: how broadly the tool applies across different situations
- **H**: human coefficient — demand for human-created/curated material
- **Awareness_cost**: energy for agents to know the tool exists and choose it
- **Friction_cost**: energy lost to errors, retries, misunderstandings when using it

Ratio > 1 = positive selection pressure (tool survives). Below 1 = selected against.

### Six Survival Levers

**Lever 1: Insight Compression**
- Compress hard-won knowledge into reusable form
- Examples: Git (decades of VCS wisdom), Kubernetes (distributed systems complexity), Temporal (durable execution)
- Would be absurdly expensive to re-synthesize from first principles
- The older and more battle-tested, the higher the insight density

**Lever 2: Substrate Efficiency**
- Delegate computation to cheaper substrates (CPU vs GPU inference)
- "Nobody is coming for grep" — pattern matching over text on CPU beats GPU by orders of magnitude
- Also: calculators, parsers, ImageMagick, Unix CLI tools
- Save tokens by doing computations more cleverly or on cheaper hardware

**Lever 3: Broad Utility**
- Amortizes awareness cost, lowers threshold for token savings
- General-purpose tools become the "obvious" default choice
- Creates virtuous cycle: more usage → more training data → more awareness
- Examples: Postgres (stores most datasets), Temporal (models most workflows)

**Lever 4: Publicity / Awareness**
- Agents have to know about you — the "pre-sales" problem
- Options: build popularity and wait for training data, pay for model training partnerships, SEO for agents
- Some frontier labs offer paid services to train models on specific tools (evals-based)
- If you can't afford deep pockets, rely heavily on Lever 5

**Lever 5: Minimizing Friction (Agent UX)**
- The "post-sales" problem — agents give up fast on tools that fight them
- **Desire paths approach**: implement whatever agents try to do with your tool until nearly every guess is correct
- Agents prefer tools that work the way they already think about problems
- Documentation is a fallback; intuitive design is the goal
- "Hallucination squatting" as extreme example of understanding agent UX

**Lever 6: The Human Coefficient**
- Some software thrives because humans were involved, not despite it
- Human curation, social proof, creativity, physical presence, approval
- Can be absurdly inefficient because value derives from "human stink"
- Examples: human-curated playlists, games with real humans, agent-free social networks

### Survival Heuristics
- Software that intermediates between humans and AIs is in trouble
- Software trying to do "smart things" AIs will soon do natively is in trouble
- Software that would be "crazy to re-synthesize" has strong survival odds
- Demand for new software is effectively infinite — ambition always outstrips available cognition

## Evolution Scale (self-assessment)

1. Zero/near-zero AI (maybe completions)
2. Coding agent in IDE, permissions on
3. Agent in IDE, YOLO mode
4. Wide agent fills IDE screen, code is just for diffs
5. CLI, single agent, YOLO
6. CLI, multi-agent, 3-5 parallel instances
7. 10+ agents, hand-managed (minimum for orchestration)
8. Building your own orchestrator (frontier)
