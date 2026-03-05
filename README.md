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

## Evolution Scale (self-assessment)

1. Zero/near-zero AI (maybe completions)
2. Coding agent in IDE, permissions on
3. Agent in IDE, YOLO mode
4. Wide agent fills IDE screen, code is just for diffs
5. CLI, single agent, YOLO
6. CLI, multi-agent, 3-5 parallel instances
7. 10+ agents, hand-managed (minimum for orchestration)
8. Building your own orchestrator (frontier)
