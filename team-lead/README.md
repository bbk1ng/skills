# team-lead

Run a multi-agent investigation the way a tech lead runs a review team: carve
scope, brief specialists in parallel, collect their reports, then send a
second **adversarial wave** to try to refute every finding before you
synthesize the one report that survives. The main session stays clean to judge
and synthesize — it never does field work.

Use for codebase/system audits, fresh-eyes reviews, deep multi-angle
investigations, or any review too big for one context. Not for single-file
questions or quick lookups — a lone agent is cheaper.

Harness-agnostic: written in capability terms (spawn a named subagent, message
a live agent, pick a model tier). See the **Harness mapping** section at the
bottom of [`SKILL.md`](SKILL.md) to map those onto your tools.
