---
name: team-lead
description: Orchestrate a team of named subagents to produce a verified, high-confidence deliverable — parallel specialist wave, adversarial verification wave, straggler re-drive, synthesized report. Use whenever the user asks for a codebase/system audit, a "fresh-eyes" or "external observer" review, a deep multi-angle investigation, or explicitly says "team lead approach", "spawn a team", "audit with agents", "fan out and verify", or wants findings that are verified rather than merely plausible. Also use for large review tasks where one context can't hold everything (whole-repo audits, multi-module consistency sweeps). Do NOT use for single-file questions, quick lookups, or tasks with an obvious single owner — a lone subagent or direct work is cheaper.
---

# Team Lead

Run a multi-agent investigation the way a good tech lead runs a review team:
carve scope, brief specialists, collect reports, chase stragglers, have a
second team try to destroy the findings, then write the one report that
survives. You (the main session) never do the field work — you stay clean to
judge and synthesize.

Why this shape works: a single agent auditing a whole repo dilutes attention
and fills its context with file dumps. Specialists with tight scopes go deep.
But specialists also hallucinate plausible-looking bugs — in a real run of
this pattern, 1 in ~14 "verified by execution" findings turned out to be
already fixed, and another had a repro string that did not actually reproduce.
The adversarial second wave is not optional garnish; it is what converts
"findings" into "facts".

## Phase 0 — Scope carve

Before spawning anything, get the terrain map cheaply:

```
git ls-files '*.js' | xargs wc -l | sort -rn   # or equivalent for the stack
```

Split the codebase into 5–9 clusters such that:
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

Spawn ALL specialists at once (one parallel batch), each as an independent
subagent with:
- **A stable name/handle** — always set it (`audit-core`, `audit-cli`…). A
  named agent can be messaged later; an anonymous one that goes idle without
  reporting is lost. (No naming/re-messaging in your harness? See the mapping
  note at the end.)
- **Capability tier** — route by task: a cheap/fast tier for reading and
  finding (specialists), reserve the strongest tier for synthesis (you).
  Don't burn a premium tier on grep. Use whatever tiers your harness exposes;
  if it has only one, the split still applies to attention, not cost.
- **A briefing prompt** containing, in this order:
  1. Role framing: *"external auditor, ZERO prior project knowledge, judge
     only what the code says, do NOT read docs/README/CLAUDE.md, no
     assumptions from project lore, READ-ONLY."* Fresh eyes are the point —
     project lore is where stale assumptions hide.
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

Reports arrive as teammate messages, asynchronously, interleaved with idle
notifications. Discipline:

- **The moment a report arrives, write it verbatim to a scratchpad file**
  (`<scratchpad>/findings-<name>.md`). Your context may be summarized before
  synthesis; the files survive. Never rely on conversation memory for raw
  findings.
- **Idle without a report** happens regularly — the agent finished but its
  final text never became a message to you. Message it (whatever your harness
  calls sending to a live agent): *"You went idle without delivering. Send
  your complete findings now to the lead. If unfinished, finish first, then
  send."*
- **Died on a rate/usage limit**: the failure notice usually includes a reset
  time. Note which agents died, wait for reset (schedule a wakeup or continue
  other work), then re-message the same agent — if your harness keeps the
  agent's transcript, it resumes where it stopped. Do not respawn fresh; you'd
  pay for the whole audit again. (If dead agents can't resume, respawn with the
  same briefing and note the lost work.)
- Track an explicit checklist of who has reported (task list or a status line
  in your updates). With 8+ agents you will lose count otherwise.

## Phase 3 — Adversarial verification wave

When all specialist reports are stashed, spawn a **second, smaller wave**
(group ~2 report files per verifier) whose brief is to **refute**:

- *"Try to REFUTE each finding. Read the cited code yourself. Run empirical
  checks where cheap (`node -e` imports, dry-run commands, real regex runs) —
  scratch files outside the repo only, repo stays read-only."*
- Point them at the stashed findings **files**, not pasted text — verifiers
  should read the full raw claims, and you shouldn't burn your context
  re-serializing them.
- Name the specific claims to attack hardest (the HIGHs, and anything whose
  repro can be executed cheaply).
- Required verdict per finding: **CONFIRMED / REFUTED (with disproof) /
  ADJUSTED (right defect, wrong detail — give the correction)** + the
  severity the verifier would assign.

Expect three verifier outputs, all valuable:
- REFUTED → drop the finding, but record it in the final report's "ruled out"
  appendix with the disproof (e.g. "already fixed in commit X") so nobody
  re-discovers the stale claim.
- ADJUSTED → the most common gold: corrected repro strings, softened
  overclaims ("every sibling catch" → "every non-redundant catch"),
  severity moves in both directions.
- CONFIRMED with new detail → verifiers often find the bug is *worse* than
  reported (traced further downstream).

Append each verifier's verdicts to the corresponding stash file. Re-drive
idle/dead verifiers exactly as in Phase 2.

## Phase 4 — Synthesize

You write the final report yourself — this is the judgment call the strong
model in the main session exists for. Structure that has worked:

1. **Method note** — how findings were produced and verified, what was
   dropped and why. Readers trust a report that shows its error bar.
2. **Findings by severity** (HIGH/MEDIUM/LOW), each with stable IDs (A1, B3…)
   so the fix session can reference them, and each carrying WHAT + MECHANISM
   + FIX, with corrections from verification folded in (use the verifier's
   corrected repro, not the specialist's broken one).
3. **Implementation order** — batch findings into fix-session-sized PRs,
   grouped to keep concurrent work off shared files; note which batches need
   a design decision vs. which are mechanical.
4. **Verified-clean appendix** — everything checked and found sound, plus
   refuted findings with their disproofs. This is what makes the report
   *terminal*: the next session starts from it instead of re-walking the repo.

Deliver the report as a file (user's requested location), and lead your final
message with the tally: N findings confirmed, M refuted, top 3 headliners.

## Costs and calibration

- A full run (8 specialists + 4 verifiers, mid-tier models) is a serious
  token spend — fit for "audit the codebase", not "check this function".
  Scale waves down for smaller scopes: 3 specialists + 1 verifier is a valid
  small team.
- Verification wave costs ~30% of the specialist wave and routinely changes
  the report. If forced to cut, cut specialist count, not verification.
- All agents read-only on the repo. Scratch/empirical work goes to the
  scratchpad. If a finding needs a mutation to prove, it ships as PLAUSIBLE,
  not CONFIRMED.

## Harness mapping

This skill is written in capability terms; map them to your tools:
- *spawn a named subagent, parallel batch* → e.g. Claude Code's Agent tool with
  `name:`/`subagent_type:`; other frameworks: their task-spawn primitive.
- *message / re-message a live agent* → e.g. SendMessage; otherwise a fresh
  agent seeded with the prior briefing.
- *capability tier* → per-agent model selection where available, else one model
  for all and the strong, clean context reserved for the lead's synthesis.
- *scratchpad* → any writable dir outside the repo under review.

If your harness has no parallel agents at all, run specialists sequentially —
the shape (deep pass → adversarial refute pass → synthesis) is what matters,
not the concurrency.
