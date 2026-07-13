---
name: stress-test-plan
description: Stress-tests a business/product/project idea using CIA Red Team tradecraft (Key Assumptions Check, Pre-Mortem, Hostile Competitor, 1-Star Review) run in strict sequence against a one-paragraph idea. Use this whenever the user wants to "red team", "stress test", "pressure test", or find holes/blind spots in an idea, plan, startup concept, product pitch, or project before committing to it — including phrasing like "poke holes in this", "what could go wrong with this idea", "is this a good idea", "play devil's advocate", or "run the 4 prompts" / "destroy my idea". Do not use this for reviewing existing code or PRs (that's what code-review tooling is for) — this is for pre-build idea validation.
---

# Stress-Test a Plan

Run four adversarial prompts, in this exact order, against a one-paragraph
statement of the user's idea. Each prompt attacks the idea from a different
angle; later prompts build on earlier ones, so do not skip or reorder them.

## Step 0: Get the idea in one paragraph

Before running anything, get (or restate back for confirmation) a single
plain-language paragraph covering: what it is, who it's for, the goal, and
what success looks like in 6 months. If the user already gave this in the
conversation, reuse it — don't make them repeat themselves. If it's missing
key pieces (no target user, no success criteria), ask before proceeding;
the four prompts below are only as sharp as the paragraph they attack.

## Step 1: Key Assumptions Check

Adopt the persona instruction exactly, then answer it in full:

```
You are now a CIA Red Team analyst. Do not evaluate whether my
idea is good. Your only job is to audit the assumptions it is
built on.

1. List every assumption my plan depends on. Not just the
obvious ones. The hidden ones I probably have not noticed.
Give me at least 10.

2. Classify each one into three tiers:
- LOAD-BEARING: If wrong, the entire plan fails.
- IMPORTANT: If wrong, the plan is weakened but survives.
- MINOR: If wrong, barely affects the outcome.

3. For each LOAD-BEARING assumption, answer: What specific
evidence would prove it wrong? If I cannot point to that
evidence, I am operating on faith, not analysis.
```

Why this is first: you cannot attack a plan you don't understand yet. This
surfaces the hidden premises the later three prompts will exploit.

## Step 2: The Pre-Mortem

```
It is now 18 months from today. The idea I shared with you has
failed catastrophically. Not "did okay." Failed. Burned.
Embarrassing.

You are writing the honest post-mortem. Walk me through exactly
what went wrong, in chronological order, step by step.

Cover these stages:
- Month 1-3: The early warning signs we ignored
- Month 4-9: The decisions that made it worse
- Month 10-15: The point of no return
- Month 16-18: The collapse and what it cost

Be specific. Name the exact mistakes. Do not be vague.

End with one sentence: "The root cause was ___."
```

Write this as a past-tense narrative, not a bulleted risk list — the
value of a pre-mortem comes from simulating the failure as already having
happened, which produces sharper, more specific answers than "what could go
wrong" framing does. Feed in whatever load-bearing assumptions surfaced in
Step 1 as candidate failure points.

## Step 3: The Hostile Competitor

```
You are now a competitor with $100 million in funding, world-class
talent, and a personal motivation to crush the idea I just shared.
You have 90 days. You have unlimited budget. You hate me.

Write a 90-day attack plan to make my idea irrelevant.

Cover:
- Days 1-30: How you study, copy, and reposition
- Days 31-60: How you launch a better version
- Days 61-90: How you starve me of customers, attention, or talent
- What I am uniquely vulnerable to that I probably do not see

Be specific. Name tactics, not vague strategy.

End with one sentence: "The weakness that lets me win is ___."
```

Make the adversary fully concrete (funded, motivated, deadlined) — a vague
"a competitor" produces vague tactics. Reuse whatever vulnerabilities
surfaced in Steps 1-2 as the competitor's opening angle.

## Step 4: The 1-Star Review

```
You are now a customer who tried my idea and hated it. You spent
real money. You spent real time. You feel cheated.

Write the 1-star review that goes viral on Twitter and gets 10,000
likes. Be specific about what disappointed you. Be funny. Be brutal.
Use the voice of someone who is angry and articulate.

Then write 3 follow-up tweets from people quoting your review and
adding their own complaints.

End with one sentence: "The single thing that made me feel cheated was ___."
```

This is the demand-side check the first three prompts miss: not whether the
idea is logically sound, but whether it's emotionally honest — whether
there's a gap between what's promised and what's delivered.

## Step 5: Synthesize

After all four, pull the four closing lines together into one short summary:

- **Root cause** (from the pre-mortem)
- **Competitive weakness** (from the hostile competitor)
- **Trust gap** (from the 1-star review)
- **Load-bearing assumptions with no evidence** (from the assumptions check)

State plainly which of these, if any, look fixable in the near term vs.
which suggest the idea itself (not just its execution) needs to change.
Don't soften this — the point of the exercise is to find the thing that
would otherwise only surface after money and time are already spent.
