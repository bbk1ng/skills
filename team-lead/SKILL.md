---
name: team-lead
description: Orchestrate a team of identified worker agents to produce a verified, high-confidence deliverable — parallel specialist wave, adversarial verification wave, straggler re-drive, synthesized report. Use whenever the user asks for a codebase or software-system audit, a "fresh-eyes" or "external observer" review, a deep multi-angle investigation, or explicitly says "team lead approach", "spawn a team", "audit with agents", "fan out and verify", or wants findings that are verified rather than merely plausible. Also use for large review tasks where one context cannot hold everything (whole-repo audits, multi-module consistency sweeps). Do NOT use for single-file questions, quick lookups, or tasks with an obvious single owner — one worker or direct work is cheaper.
---

# Team Lead

Run a multi-agent investigation the way a good tech lead runs a review team:
carve scope, brief specialists, collect reports, chase stragglers, have a
second team try to destroy the findings, then write the one report that
survives. The coordinator never does the field work — it stays clean to
judge and synthesize.

Map coordinator, worker launch, messaging, status, and resumption to the
primitives available in the current multi-agent runtime. Do not assume any
vendor, model family, tool name, or message-routing API.

Why this shape works: a single agent auditing a whole repo dilutes attention
and fills its context with file dumps. Specialists with tight scopes go deep.
But specialists also hallucinate plausible-looking bugs — in a real run of
this pattern, 1 in ~14 "verified by execution" findings turned out to be
already fixed, and another had a repro string that did not actually reproduce.
The adversarial second wave is not optional garnish; it is what converts
"findings" into "facts".

## Phase 0 — Scope carve

Before launching workers, inventory the target artifacts and estimate their
size with the source-control or filesystem tools available in the environment.
For a Git repository in a POSIX shell, an optional example is:

```
git ls-files '*.js' | xargs wc -l | sort -rn   # or equivalent for the stack
```

Aim for 5–9 clusters, but use fewer when the scope or runtime capacity is
smaller. Ensure that:
- Each cluster is a coherent domain (core pipeline, CLI, adapters, UI, config…).
- Every file belongs to exactly one deep-audit cluster (no gaps, no overlap).
- One extra agent gets the **cross-cutting** brief: it deep-audits nothing,
  it only hunts *disagreements between files* — duplicated helpers drifting,
  writer/reader data-shape mismatches, naming drift, CI-vs-scripts mismatch.
- One extra agent gets the **tests-vs-source** brief if a test suite exists:
  run the suite, then hunt tests asserting buggy behavior, coverage holes on
  risky exports, isolation hazards.

The biggest single file usually deserves its own agent.

## Phase 1 — Specialist wave

Launch specialists concurrently up to the runtime's supported limit and queue
the remainder in batches.
For each worker:
- **Stable identifier** — always track a coordinator-side identifier
  (`audit-core`, `audit-cli`…). If the runtime supports addressable names,
  assign the same identifier to the worker so it can be contacted or
  re-driven later.
- **Capability routing** — if the runtime supports it, use an economical
  reading/analysis capability for specialists and reserve the strongest
  reasoning capability for synthesis. Do not spend expensive capability on
  mechanical discovery.
- **A briefing prompt** containing, in this order:
  1. Role framing: *"external auditor, ZERO prior project knowledge, judge
     only what the code says, do NOT read project documentation,
     instruction files, or memory files; make no assumptions from project
     lore; READ-ONLY."* Fresh eyes are the point — project lore is where stale
     assumptions hide.
  2. Exact file list (paths, not vibes). Permission to skim imports only to
     verify call contracts.
  3. A hunt list specific to that domain (5–7 concrete defect classes, e.g.
     "lock acquire/release asymmetry", "ANSI-aware vs naive string length
     mixed"). Generic "find bugs" briefs produce generic noise.
  4. Required finding format: `file:line` + severity + WHAT (one sentence) +
     MECHANISM (concrete failure scenario with inputs) + FIX DIRECTION
     (implementable in 1–2 sentences).
  5. Anti-noise rules: no doc issues, no style nits, no "could maybe" without
     a concrete failure path, re-read the code before including each finding.
     Ask for "clean" declarations too ("if a module is clean, say so in one
     line") — verified-clean areas are deliverable content (they stop the fix
     session from re-auditing ghosts).

## Phase 2 — Collect and stash

Collect reports through the runtime's completion, result, or messaging
mechanism. Handle asynchronous or interleaved delivery when applicable.
Discipline:

- **The moment a report arrives, persist it verbatim in durable temporary
  storage** (`<scratchpad>/findings-<name>.md`, an artifact store, or an
  equivalent). Coordinator context may be summarized before synthesis; the
  persisted reports survive. Never rely on conversation memory for raw
  findings.
- **Completion without a report** can happen. If the runtime supports
  follow-up messaging, send: *"You finished without delivering. Send your
  complete findings to the coordinator now. If unfinished, finish first, then
  send."* Otherwise, launch a replacement with the original brief and any
  preserved artifacts.
- **Interrupted by a rate or resource limit**: record the worker and any
  retry time. When capacity returns, resume or re-drive the same identified worker
  if the runtime preserves its context. Respawn only when resumption is
  unavailable, supplying the original brief and preserved artifacts because a
  fresh worker otherwise repeats discovery.
- Track an explicit checklist of who has reported. With 8+ workers it is easy
  to lose count.

## Phase 3 — Adversarial verification wave

When all specialist reports are stashed, spawn a **second, smaller wave**
(group ~2 report files per verifier) whose brief is to **refute**:

- *"Try to REFUTE each finding. Read the cited code yourself. Run cheap,
  stack-appropriate empirical probes (interpreter one-liners, dry-run
  commands, isolated regex checks) — use temporary storage and keep the target
  repository read-only."*
- Give verifiers the persisted finding reports through a shared filesystem,
  artifact store, attachment, or prompt payload according to runtime
  capabilities. They must receive the complete raw claims.
- Name the specific claims to attack hardest (the HIGHs, and anything whose
  repro can be executed cheaply).
- Required verdict per finding: **CONFIRMED / REFUTED (with disproof) /
  ADJUSTED (right defect, wrong detail — give the correction)** + the
  severity the verifier would assign.

Expect three verifier outputs, all valuable:
- REFUTED → drop the finding, but record it in the final report's "ruled out"
  appendix with the disproof (e.g. "already fixed in revision X") so nobody
  re-discovers the stale claim.
- ADJUSTED → the most common gold: corrected repro strings, softened
  overclaims ("every sibling catch" → "every non-redundant catch"),
  severity moves in both directions.
- CONFIRMED with new detail → verifiers often find the bug is *worse* than
  reported (traced further downstream).

Append each verifier's verdicts to the corresponding stash file. Re-drive
idle/dead verifiers exactly as in Phase 2.

## Phase 4 — Synthesize

The coordinator writes the final report — this is the judgment call for which
the strongest available reasoning capability was reserved. Structure that has
worked:

1. **Method note** — how findings were produced and verified, what was
   dropped and why. Readers trust a report that shows its error bar.
2. **Findings by severity** (HIGH/MEDIUM/LOW), each with stable IDs (A1, B3…)
   so the fix pass can reference them, and each carrying WHAT + MECHANISM
   + FIX, with corrections from verification folded in (use the verifier's
   corrected repro, not the specialist's broken one).
3. **Implementation order** — batch findings into fix-pass-sized changes,
   grouped to keep concurrent work off shared files; note which batches need
   a design decision vs. which are mechanical.
4. **Verified-clean appendix** — everything checked and found sound, plus
   refuted findings with their disproofs. This is what makes the report
   *terminal*: the next run starts from it instead of re-walking the repo.

Deliver the report as a file (user's requested location), and lead your final
message with the tally: N findings confirmed, M refuted, top 3 headliners.

## Costs and calibration

- A full run (8 specialists + 4 verifiers) is a serious compute and context
  spend — fit for "audit the codebase", not "check this function".
  Scale waves down for smaller scopes: 3 specialists + 1 verifier is a valid
  small team.
- Verification wave costs ~30% of the specialist wave and routinely changes
  the report. If forced to cut, cut specialist count, not verification.
- Keep all workers read-only on the target repository. Put empirical work in
  durable temporary storage. If a finding needs a repository mutation to
  prove, it ships as PLAUSIBLE, not CONFIRMED.
