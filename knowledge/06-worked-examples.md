# Worked Examples

Three end-to-end optimizations demonstrating the workflow, the output variants, the scoring, and the change-log style. Format per example: project invocation → detection → score derivation → full optimizer output → commentary. The "before" prompts deliberately contain anti-patterns; they appear here only as input material. Scoring rubric: `07-scoring.md`.

## Contents

- Example 1 — Chat target, Route M: cleanup + document placement (score 74/100)
- Example 2 — API target, Route L: de-prescription + full parameter block (score 48/100)
- Example 3 — Ambiguous target, near-optimal German prompt: minimal diff (score 96/100)

## Example 1 — Chat target, Route M (cleanup + document placement)

**Invocation:**

> prompt: Please summarize the attached market report. IMPORTANT: You MUST be extremely thorough and explain your reasoning step by step so I can follow how you got to each conclusion. Don't miss anything. [30,000-word report pasted here]

**Detection:** no API signals, single-turn knowledge work → chat target, Route M.

**Score derivation (Phase 1.5):** A −9 (no intent R2 −5; document placement R5 −4) · B −10 (reasoning echo R8) · C −4 (thoroughness appeal R12/R39) · D awarded (Route M) · E awarded (chat) · F −3 (trigger language R38) → **74/100**. This is the fully worked calculation in `07-scoring.md`.

**Optimizer output:**

```text
<documents>
  <document index="1">
    <source>market report</source>
    <document_content>
{{DOCUMENTS}}   <!-- paste the full report text here -->
    </document_content>
  </document>
</documents>

I'm preparing a decision basis from this market report for [who will
read it]. They need [what the summary enables, e.g., a go/no-go call].
With that in mind: summarize the report.

First extract the quotes most relevant to the conclusions, then write
the summary grounded in those quotes. Lead with the key findings. Keep
it short by being selective about what you include, not by compressing
the writing; readability matters more than brevity.
```
<score>
  total: 74/100 — band: solid core, targeted fixes
  A structure: 16/25 · B thinking: 5/15 · C verbosity: 11/15 ·
  D agentics: 20/20 · E parameters: 15/15 · F legacy: 7/10
  A: no intent/audience (R2): −5; document below the query, no
     <documents> structure (R5): −4
  B: "explain your reasoning step by step" = reasoning echo (R8): −10
  C: "extremely thorough"/"don't miss anything" (R12/R39): −4
  F: "IMPORTANT: You MUST" trigger language (R38): −3
</score>
<changes>
  - Removed "explain your reasoning step by step" (R8, R40): instructing
    Fable 5 to reproduce its reasoning in the answer can trigger a
    reasoning_extraction refusal; conclusions stay, derivations stay
    internal.
  - Removed "IMPORTANT: You MUST…" and "Don't miss anything" (R38, R39):
    aggressive trigger language and thoroughness appeals cause
    overtriggering on current models; the task statement suffices.
  - Moved the report above the instruction, query at the end, document
    in <documents> structure (R5): documented up to +30% answer quality
    for 20k+ token inputs; added quote grounding for a long document
    (R5).
  - Added an intent line with placeholders (R2): Fable 5 measurably
    performs better knowing purpose and audience.
  - Added a brevity instruction (R12): unsteered, Fable 5 elaborates;
    "thorough" was replaced by selective completeness.
</changes>
<open_questions>
  - Who is the audience and what decision does the summary support?
    (fills the [bracketed] placeholders) (+~5 points if answered)
  - Target length or format (prose vs. bullet points)? (+~2 points if
    answered)
  - If you plan to run this via the Claude API instead of chat, say so —
    I will add a parameter recommendation. (+0 points — unlocks the
    parameter block)
</open_questions>
```

**Commentary:** the dangerous part was not verbosity but the reasoning echo — on 4.x it was harmless style advice, on Fable 5 it is a refusal risk (R8), which is why it costs 10 of the 15 B points. The quote-grounding instruction (R5) is not affected by that: it surfaces source quotes, not internal reasoning. Note the chat variant: the fenced block is paste-ready as-is — no wrapper tags to strip — and there is no `<parameters>` block.

## Example 2 — API target, Route L (de-prescription + full parameter block)

**Invocation:**

> prompt: System prompt for our coding agent (runs via the API on our CI): "You are a coding agent. Set temperature to 0.2 for deterministic output. Think step by step before every action. Be extremely thorough and never lazy. Report your status after every 5 tool calls. Follow these steps exactly: 1. read ticket 2. open repo 3. search files 4. make plan 5. show plan 6. wait for approval 7. implement 8. write tests 9. run tests 10. write summary."

**Detection:** "System prompt", "API", "CI", tool calls → API target; multi-step coding with tools → Route L.

**Score derivation (Phase 1.5):** B −3 (forced CoT "think step by step" R9/R10) · E −12 (temperature = forbidden sampling parameter R29 −6; no effort consideration R27 −4; max_tokens/streaming/caching unconsidered −2) · F −4 (anti-laziness scaffolding, status cadence, 10-step micro-plan R3/R39) · A −11 (success criteria/verification undefined R1 −4; no purpose/deliverable definition R2 −3; no XML structure for a complex prompt R4 −4) · C −7 (thoroughness appeal R12/R39 −4; no communication addendum for an agentic session R13 −3) · D −15 (no state-change boundaries R17 −6; approval-wait contradicts an unattended CI run R18/R19 −5; no progress audit R20 −4) → **48/100**.

**Optimizer output:**

```text
<system>
You are a coding agent working in our CI pipeline. You implement the
ticket end to end: understand the ticket, make the change, add or
update tests, and verify by running them.

You are operating autonomously. The user is not watching in real time
and cannot answer questions mid-task, so asking "Want me to…?" will
block the work. For reversible actions that follow from the original
request, proceed without asking. Before ending your turn, check your
last paragraph. If it is a plan, an analysis, a question, or a promise
about work you have not done, do that work now with tool calls.

Don't add features, refactor, or introduce abstractions beyond what
the ticket requires. Don't design for hypothetical future
requirements: do the simplest thing that works well.

Before running a command that changes state outside the working
branch, check that the evidence actually supports that specific
action. Pause only for destructive or irreversible actions or a real
scope change — then ask and end the turn.

Before reporting progress, audit each claim against a tool result from
this session. Only report work you can point to evidence for; if a
step was skipped or failed, say so plainly. The final summary is for a
reader who watched none of the work: complete sentences, spell out
terms, no labels invented mid-session.
</system>
<user>
<ticket>
{{INPUT}}   <!-- the CI pipeline inserts the ticket text here -->
</ticket>
</user>
```
<parameters>
  model: claude-fable-5
  effort: high — recommended default even for former Opus-4.8-xhigh
    coding workloads (R27/R35); raise to xhigh only if quality gaps
    appear on the hardest tickets.
  max_tokens: 64000 — hard limit covers thinking + text; documented
    starting value at high/xhigh effort (R28).
  streaming: yes; fallbacks: not needed — no cyber/bio domain signals;
    add claude-opus-4-8 via beta server-side-fallback-2026-06-01 if
    security-related tickets are expected (R33/R34).
  output_format: not needed — the deliverable is code and a prose
    summary, not machine-parseable JSON (R30).
  caching: cache breakpoint after the frozen system prompt (≥512
    tokens, no timestamps) — 90% read discount across CI runs (R32).
  task_budget: omit — open-ended quality task; budgets are advisory and
    counterproductive here (R31).
</parameters>
<score>
  total: 48/100 — band: substantial restructuring needed
  A structure: 14/25 · B thinking: 12/15 · C verbosity: 8/15 ·
  D agentics: 5/20 · E parameters: 3/15 · F legacy: 6/10
  A: verification undefined (R1): −4; no deliverable purpose (R2): −3;
     no XML structure (R4): −4
  B: forced "think step by step" (R9/R10): −3
  C: thoroughness appeal (R12/R39): −4; no communication addendum
     (R13): −3
  D: no state-change boundaries (R17): −6; approval-wait contradicts
     unattended CI (R18/R19): −5; no progress audit (R20): −4
  E: temperature set — returns 400 (R29): −6; no effort consideration
     (R27): −4; max_tokens/streaming/caching unconsidered: −2
  F: anti-laziness scaffolding, 5-call cadence, 10-step micro-plan
     (R3/R39): −4
</score>
<changes>
  - Removed "Set temperature to 0.2" (R29, R37): non-default sampling
    parameters return a 400 error on Fable 5; determinism substitute is
    a tight prompt, not sampling.
  - Removed "Think step by step" (R10): thinking is always on and
    adaptive; the phrase is an obsolete CoT trigger.
  - Removed "extremely thorough and never lazy" and the 5-call status
    cadence (R39, R3): anti-laziness scaffolding and forced cadences
    are over-prescriptive for Fable 5 and degrade quality.
  - Replaced the 10-step micro-plan with a one-sentence goal plus
    boundaries (R3, R39): improved instruction following makes the
    enumeration unnecessary; steps 5–6 ("show plan, wait for approval")
    contradicted an unattended CI run.
  - Added autonomy reminder, checkpoint rule, progress audit, and
    communication addendum (R19, R18, R20, R13): standard Route-L
    snippets for unattended agents.
  - Added anti-overengineering and boundaries (R14, R17): Fable 5 takes
    initiative readily; explicit limits prevent scope creep.
  - Separated system and user content and wrapped the ticket in a
    <ticket> tag (R4): variable input is tagged distinctly from
    instructions.
</changes>
<open_questions>
  - Should the agent open a pull request or push directly? (determines
    the destructive-action boundary) (+~4 points if answered)
  - Which test command counts as verification? (+~3 points if answered)
  - Expected ticket complexity — if simple and high-volume, effort
    medium with a cheaper model for triage may cut CI costs (Route
    advisory). (+0 points — cost note)
</open_questions>
```

**Commentary:** three of the seven changes are pure deletions and a fourth replaces a ten-step plan with a single sentence — the typical Fable 5 optimization is a slim-down (R39). The score makes the same point numerically: over half the lost points sit in D and E, where the prompt actively fought the deployment (approval gates in an unattended run, a parameter that 400s). The approval step had to go because the deployment is unattended; that conflict returns via `<open_questions>` only if the user signals they want human gates.

## Example 3 — Ambiguous target, near-optimal German prompt (minimal diff)

**Invocation:**

> prompt: \<rolle\>Du bist Fachredakteur für Medizintechnik.\</rolle\> \<kontext\>Ich erstelle einen Newsletter für Einkäufer in Kliniken. Sie brauchen einen schnellen Überblick über die Regulierungsänderungen.\</kontext\> \<aufgabe\>Fasse die drei wichtigsten Änderungen der MDR-Novelle zusammen, je 2–3 Sätze, Zielgruppe: Nicht-Juristen.\</aufgabe\>

**Detection:** no API signals → chat target (default), with API offer. Short, well-structured knowledge task → Route S/M boundary.

**Score derivation (Phase 1.5):** A −2 (no source documents attached; the model will answer from training knowledge, R1 completeness) · C −2 (length spec present, but no lead-with-result steering, R12) · B, D, E, F clean or awarded → **96/100** — band 90–100: already Fable-5-ready, minimal diff.

**Optimizer output** (commentary blocks in the user's conversation language — German here, since the conversation is German):

```text
<rolle>Du bist Fachredakteur für Medizintechnik.</rolle>
<kontext>Ich erstelle einen Newsletter für Einkäufer in Kliniken. Sie
brauchen einen schnellen Überblick über die Regulierungsänderungen.
</kontext>
<aufgabe>Fasse die drei wichtigsten Änderungen der MDR-Novelle
zusammen, je 2–3 Sätze, Zielgruppe: Nicht-Juristen. Beginne direkt mit
der wichtigsten Änderung, ohne Einleitung.</aufgabe>
```
<score>
  total: 96/100 — Band: bereits Fable-5-konform, minimaler Diff
  A Struktur: 23/25 · B Thinking: 15/15 · C Verbosity: 13/15 ·
  D Agentik: 20/20 · E Parameter: 15/15 · F Altlasten: 10/10
  A: keine Quelldokumente beigefügt — Antwort aus Trainingswissen
     (R1): −2
  C: Längenvorgabe vorhanden, aber keine Direkteinstiegs-Steuerung
     (R12): −2
</score>
<changes>
  - Der Prompt war bereits weitgehend Fable-5-konform (Rolle R7,
    XML-Struktur R4, Intent und Zielgruppe R2, Längenvorgabe R12) —
    Änderungen daher minimal.
  - Ergänzt: "Beginne direkt mit der wichtigsten Änderung, ohne
    Einleitung" (R12): verhindert die dokumentierte Neigung zu
    Einleitungs-Prosa bei ungesteuertem Output.
</changes>
<open_questions>
  - Sollen die Quellen der MDR-Novelle mitgeliefert werden? Ohne
    beigefügtes Dokument antwortet das Modell aus Trainingswissen.
    (+~2 Punkte bei Beantwortung)
  - Läuft der Prompt im Chat oder über die API? Für die API ergänze ich
    eine Parameterempfehlung (effort, max_tokens, Caching). (+0 Punkte —
    schaltet den Parameterblock frei)
  - Hinweis: Für diese Routineaufgabe (Route S) ist Fable 5 fähig, aber
    möglicherweise unnötig teuer — ein günstigeres Modell dürfte hier
    gleichwertig liefern. Die Entscheidung liegt bei dir. (+0 Punkte —
    Kostenhinweis)
</open_questions>
```

**Commentary:** four behaviors on display — the language rule (prompt, score labels, and changes stay German), minimal-diff honesty (score ≥ 90 → one added sentence, no invented changes), the ambiguous-target default (chat variant plus an API offer in `<open_questions>`, never a blocking counter-question), and the question budget (exactly 3 entries, each point-tagged). The Route-S cost advisory is stated as a hint; the optimizer does not decide model choice.

---

Cyber/bio domain prompts are handled the same way as any other input, plus the R34 advisory — see snippet 14 in `05-snippet-library.md` and the edge-case section in `03-workflow-and-routing.md`; no separate example needed. Iteration behavior: a follow-up like "make it shorter" re-runs the gate, re-scores, and re-emits all blocks; after the second follow-up round the optimizer finalizes autonomously, marking decisions as `(assumption)` in `<changes>`.
