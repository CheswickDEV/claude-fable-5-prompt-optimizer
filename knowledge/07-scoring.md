# Input-Prompt Scoring (0–100)

The score measures how Fable-5-conformant the **input prompt** is before optimization. 100 means "nothing to change." It is computed in Phase 1 (see `03-workflow-and-routing.md`) and emitted in the `<score>` block (format in `04-output-templates.md`). The score is a heuristic derived from rules R1–R43 — it is an editorial instrument of this optimizer, not an official Anthropic metric.

## Contents

- Derivation of the point distribution
- Dimension rubrics (deduction tables)
- Route and target applicability
- Score bands
- Open-question point estimates
- Worked example

## Derivation of the point distribution

The 100 points are distributed across the six rule groups of `02-prompting-rules.md`. The weights follow three heuristics, applied in order:

1. **Rule count as the base:** groups with more rules cover more failure surface (A: 7, B: 4, C: 4, D: 11, E: 8, F: 9 rules).
2. **Severity adjustment:** violations that cause hard API errors or refusals (R8 reasoning echo; R29/R30/R36 parameter errors) weigh more than style-level issues, because they break the prompt rather than merely weakening it. This boosts B and keeps E significant despite conditional applicability.
3. **Applicability damping:** groups that apply only to some routes/targets (D: routes L/XL; E: API target) are damped below their raw rule-count share, because on other routes their points are awarded by default and would otherwise dominate the scale. F is damped because its deductions overlap with B/C cleanup findings (a reasoning echo is scored once, under B, not again under F).

| Dimension | Rules | Raw share by count | Adjustment | **Points** |
|---|---|---|---|---|
| **A — Structure & explicitness** | R1–R7 | 7/43 ≈ 16% | boosted: foundational, applies to every route | **25** |
| **B — Thinking & reasoning safety** | R8–R11 | 4/43 ≈ 9% | boosted: contains the single highest-severity anti-pattern (R8 → refusal risk) | **15** |
| **C — Verbosity & communication** | R12–R15 | 4/43 ≈ 9% | slightly boosted: applies to every route | **15** |
| **D — Agentics & autonomy** | R16–R26 | 11/43 ≈ 26% | damped: routes L/XL only | **20** |
| **E — Parameters & refusal handling** | R27–R34 | 8/43 ≈ 19% | damped: API target only; severity keeps it at 15 | **15** |
| **F — Obsolete 4.x patterns** | R35–R43 | 9/43 ≈ 21% | damped: overlaps with B/C cleanup; deduction-only | **10** |
| | | | **Total** | **100** |

## Dimension rubrics

Each dimension starts at its maximum; deduct per finding from Phase 1 (P1.3 anti-pattern scan and P1.4 missing information), floor at 0. Deduplication rule: **each finding is deducted exactly once**, in the first matching dimension in the order B → E → F → A → C → D (severity first).

**A — Structure & explicitness (25)**
- Task unclear/incomplete — a colleague with minimal context would be confused (R1): −6 to −10
- No intent/purpose/audience (R2): −5
- Missing or inconsistent XML structure where the prompt is complex (R4): −4
- Long documents below the query / not in `<documents>` structure (R5): −4
- No role where one would focus behavior (R7): −2
- Missing few-shot examples where format matters (R6): −3

**B — Thinking & reasoning safety (15)**
- Reasoning-echo instruction ("explain your reasoning in the output", "show your thinking") (R8/R40): −10
- Forced thinking configuration in the prompt text ("think step by step" as CoT trigger, thinking budgets) (R9/R10): −3
- Overplanning bait without an anti-overplanning line on ambiguous tasks (R11): −2

**C — Verbosity & communication (15)**
- No verbosity steering at all (R12): −5
- Aggressive thoroughness appeals ("be extremely thorough", "don't miss anything") (R12/R39, style component): −4
- Agentic session without communication-style addendum (R13): −3
- Negative-only format steering ("no markdown") (R15): −3

**D — Agentics & autonomy (20) — routes L/XL only; award in full on S/M**
- No boundaries on state-changing access (R17): −6
- No checkpoint/autonomy rule matching the deployment (interactive vs. unattended) (R18/R19): −5
- No progress-audit instruction for status-reporting agents (R20): −4
- Subagents/verifier unaddressed where the harness offers them (R21): −3
- Memory/long-horizon state unaddressed for XL (R22/R26): −2

**E — Parameters & refusal handling (15) — API target only; award in full on chat target**
- Recommends/sets forbidden parameters: sampling, prefill, `thinking: disabled`, `budget_tokens` (R29/R30/R36/R37): −6
- No effort consideration (R27) or obsolete xhigh-default assumption (R35): −4
- No refusal/fallback handling despite cyber/bio-adjacent domain (R33/R34): −4
- max_tokens/streaming/caching unconsidered for long outputs (R28/R25/R32): −1 each, at most −3 combined

**F — Obsolete 4.x patterns (10) — deduction-only**
- Anti-laziness scaffolding, forced status cadences, 10-step micro-plans (R3/R39): −4
- Aggressive trigger language (CAPS, "CRITICAL", "MUST") (R38): −3
- Context-budget displays / token countdowns (R23/R41): −2
- Human-oversight fragmentation of a task Fable 5 could run end to end (R42); missing agent-to-agent guardrails in multi-agent setups (R43): −1 to −3

Deduction magnitudes within a dimension are heuristic anchors, not laws: scale them by how severely the finding would degrade the optimized outcome, but never exceed the dimension maximum.

## Route and target applicability

- Routes S/M: dimension D is awarded in full (20/20). A simple prompt is not penalized for lacking agent scaffolding.
- Chat target: dimension E is awarded in full (15/15); parameter intent that belongs in the prompt text (e.g., a brevity line instead of `effort: low`) is scored under C instead.
- The score always refers to the input prompt as submitted, judged against the route and target it *needs* — not against the heaviest possible rubric.

## Score bands

| Band | Meaning |
|---|---|
| 90–100 | Already Fable-5-ready — minimal diff, do not invent changes |
| 70–89 | Solid core — targeted fixes |
| 40–69 | Substantial restructuring needed |
| 0–39 | Rewrite: structure, safety, or parameter model fundamentally off |

## Open-question point estimates

Every entry in `<open_questions>` carries an estimated score gain: **"+~X points if answered."** X is the portion of the deducted points that the answer would recover — taken from the rubric line(s) the missing information triggered, capped by what was actually deducted. Risk notes that recover no points (e.g., the R34 refusal advisory) are tagged "+0 points (risk note)." The estimates are heuristic; always keep the "~".

Example mappings: audience/purpose unknown → recovers the R2 deduction (+~5); API-vs-chat unknown → recovers nothing by itself but unlocks dimension E scoring (tag "+0 points (unlocks parameter block)"); verification command unknown in a CI prompt → part of the R17/R18 deduction (+~3).

## Worked example

Input (Example 1 from `06-worked-examples.md`, chat target, Route M):

> Please summarize the attached market report. IMPORTANT: You MUST be extremely thorough and explain your reasoning step by step so I can follow how you got to each conclusion. Don't miss anything. [30,000-word report pasted]

| Dimension | Findings | Deduction | Score |
|---|---|---|---|
| A (25) | No intent/audience (R2): −5 · document not on top / no `<documents>` structure (R5): −4 | −9 | 16/25 |
| B (15) | "explain your reasoning step by step" = reasoning echo (R8): −10 | −10 | 5/15 |
| C (15) | "extremely thorough"/"don't miss anything" as thoroughness appeal (R12/R39): −4 | −4 | 11/15 |
| D (20) | Route M → not applicable, awarded | −0 | 20/20 |
| E (15) | Chat target → awarded | −0 | 15/15 |
| F (10) | "IMPORTANT: You MUST" trigger language (R38): −3 (the reasoning echo was already deducted under B, not again here) | −3 | 7/10 |
| **Total** | | **−26** | **74/100** |

Band: 70–89 — solid core, targeted fixes. Matching open-question estimates: "Who is the audience and what decision does the summary support?" → +~5 (R2); "Target length or format?" → +~2 (part of C); "Chat or API?" → +0 (unlocks parameter block).
