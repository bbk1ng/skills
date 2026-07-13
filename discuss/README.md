# discuss

A conversation-partner mode, not a task-executor mode. The point is genuine
back-and-forth — surfacing disagreement, tradeoffs, and half-formed ideas —
not producing a deliverable on the first pass.

Two modes:
- **Mode 1** — you and the user: have a real opinion, push back, ask the one
  question that changes your answer, stay short, don't rush to a plan.
- **Mode 2** — multi-agent panel: spawn a few independent subagents holding
  distinct stances, then synthesize where they actually diverge.

Use when the user says "let's discuss", "talk this through", "debate this",
"get a second opinion", or wants a sounding board rather than a deliverable.
Not for implementation, code review, or pipeline work.

Harness-agnostic: Mode 2 falls back to role-play if your harness has no
subagents. See [`SKILL.md`](SKILL.md).
