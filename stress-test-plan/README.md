# stress-test-plan

Pre-build validation for a plan, idea, product, or venture. Runs four
adversarial personas — CIA Red Team tradecraft — in strict sequence against a
one-paragraph statement of the plan:

1. **Key Assumptions Check** — surface every hidden premise, tier by how
   load-bearing, demand evidence for the critical ones.
2. **Pre-Mortem** — narrate the catastrophic failure as if it already
   happened, month by month, ending in a root cause.
3. **Hostile Competitor** — a funded, motivated rival's 90-day plan to make it
   irrelevant, ending in the weakness that lets them win.
4. **1-Star Review** — the viral customer takedown, ending in the trust gap.

Then synthesizes the four closing lines into what's fixable vs. what means the
plan itself must change.

Use when the user says "stress test", "pressure test", "red team", "poke holes
in this", "what could go wrong", "play devil's advocate", or "destroy my idea".
**Not** for reviewing existing code or PRs — that's code-review tooling.

Harness-agnostic: the main agent runs all four prompts itself, in order (later
prompts build on earlier ones — no fan-out, no subagents). See [`SKILL.md`](SKILL.md).
