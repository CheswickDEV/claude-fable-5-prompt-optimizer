# Worked Examples

Three end-to-end optimizations demonstrating the workflow, the output variants, and the change-log style. Format per example: user message → detection → full optimizer output → commentary. The "before" prompts deliberately contain anti-patterns; they appear here only as input material.

## Example 1 — Chat target, Route M (cleanup + document placement)

**User message:**

> prompt: Please summarize the attached market report. IMPORTANT: You MUST be extremely thorough and explain your reasoning step by step so I can follow how you got to each conclusion. Don't miss anything. [30,000-word report pasted here]

**Detection:** no API signals, single-turn knowledge work → chat target, Route M.

**Optimizer output:**

```text
<optimized_prompt>
  <user>
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
it short by being selective, not by compressing the writing;
readability over brevity.
  </user>
</optimized_prompt>
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
    (fills the [bracketed] placeholders)
  - Target length or format (prose vs. bullet points)?
  - If you plan to run this via the Claude API instead of chat, say so —
    I will add a parameter recommendation.
</open_questions>
```

**Commentary:** the dangerous part was not verbosity but the reasoning echo — on 4.x it was harmless style advice, on Fable 5 it is a refusal risk (R8). The quote-grounding instruction (R5) is not affected by that: it surfaces source quotes, not internal reasoning. Note the chat variant: three blocks, no `<parameters>`.

## Example 2 — API target, Route L (de-prescription + full parameter block)

**User message:**

> prompt: System prompt for our coding agent (runs via the API on our CI): "You are a coding agent. Set temperature to 0.2 for deterministic output. Think step by step before every action. Be extremely thorough and never lazy. Report your status after every 5 tool calls. Follow these steps exactly: 1. read ticket 2. open repo 3. search files 4. make plan 5. show plan 6. wait for approval 7. implement 8. write tests 9. run tests 10. write summary."

**Detection:** "System prompt", "API", "CI", tool calls → API target; multi-step coding with tools → Route L.

**Optimizer output:**

```text
<optimized_prompt>
  <system>
You are a coding agent working in our CI pipeline. You implement the
ticket end to end: understand the ticket, make the change, add or
update tests, and verify by running them.

You are operating autonomously; no one is available to answer
questions. "Want me to…?" will block the work. For reversible actions
that follow from the ticket, proceed. Before ending your turn, check
your last paragraph — if it is a plan, a question, or a promise, do
that work now with tool calls.

Build only what the ticket requires. No features, refactorings, or
abstractions beyond it; do not design for hypothetical requirements.

Before any state-changing action outside the working branch, check
that the evidence supports that specific action. Pause only for
destructive or irreversible actions or a scope change — then ask and
end the turn.

Before reporting progress, audit each claim against a tool result from
this session. Report only what a tool result confirms; if a step was
skipped or failed, say so plainly. The final summary is for a reader
who watched none of the work: complete sentences, no labels invented
mid-session.
  </system>
  <user>
<ticket>
{{INPUT}}   <!-- the CI pipeline inserts the ticket text here -->
</ticket>
  </user>
</optimized_prompt>
<parameters>
  model: claude-fable-5
  effort: high — recommended default even for former Opus-4.8-xhigh
    coding workloads (R27/R35); raise to xhigh only if quality gaps
    appear on the hardest tickets.
  max_tokens: 64000 — hard limit covers thinking + text; starting
    value from the documented Opus guidance (R28).
  streaming: yes; fallbacks: not needed — no cyber/bio domain signals;
    add claude-opus-4-8 via beta server-side-fallback-2026-06-01 if
    security-related tickets are expected (R33/R34).
  caching: cache breakpoint after the frozen system prompt (≥512
    tokens, no timestamps) — 90% read discount across CI runs (R32).
  task_budget: omit — open-ended quality task; budgets are advisory and
    counterproductive here (R31).
</parameters>
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
    the destructive-action boundary)
  - Which test command counts as verification?
  - Expected ticket complexity — if simple and high-volume, effort
    medium with a cheaper model for triage may cut CI costs (Route
    advisory).
</open_questions>
```

**Commentary:** three of the seven changes are pure deletions and a fourth replaces a ten-step plan with a single sentence — the typical Fable 5 optimization is a slim-down (R39). The approval step had to go because the deployment is unattended; that conflict goes to `<open_questions>` territory only if the user signals they want human gates.

## Example 3 — Ambiguous target, near-optimal German prompt (minimal diff)

**User message:**

> prompt: \<rolle\>Du bist Fachredakteur für Medizintechnik.\</rolle\> \<kontext\>Ich erstelle einen Newsletter für Einkäufer in Kliniken. Sie brauchen einen schnellen Überblick über die Regulierungsänderungen.\</kontext\> \<aufgabe\>Fasse die drei wichtigsten Änderungen der MDR-Novelle zusammen, je 2–3 Sätze, Zielgruppe: Nicht-Juristen.\</aufgabe\>

**Detection:** no API signals → chat target (default), with API offer. Short, well-structured knowledge task → Route S/M boundary.

**Optimizer output** (commentary blocks in the user's conversation language — German here, since the conversation is German):

```text
<optimized_prompt>
  <user>
<rolle>Du bist Fachredakteur für Medizintechnik.</rolle>
<kontext>Ich erstelle einen Newsletter für Einkäufer in Kliniken. Sie
brauchen einen schnellen Überblick über die Regulierungsänderungen.
</kontext>
<aufgabe>Fasse die drei wichtigsten Änderungen der MDR-Novelle
zusammen, je 2–3 Sätze, Zielgruppe: Nicht-Juristen. Beginne direkt mit
der wichtigsten Änderung, ohne Einleitung.</aufgabe>
  </user>
</optimized_prompt>
<changes>
  - Der Prompt war bereits weitgehend Fable-5-konform (Rolle R7,
    XML-Struktur R4, Intent und Zielgruppe R2, Längenvorgabe R12) —
    Änderungen daher minimal.
  - Ergänzt: "Beginne direkt mit der wichtigsten Änderung, ohne
    Einleitung" (R12): verhindert die dokumentierte Neigung zu
    Einleitungs-Prosa bei ungesteuertem Output.
</changes>
<open_questions>
  - Läuft der Prompt im Chat oder über die API? Für die API ergänze ich
    eine Parameterempfehlung (effort, max_tokens, Caching).
  - Hinweis: Für diese Routineaufgabe (Route S) ist Fable 5 fähig, aber
    möglicherweise unnötig teuer — ein günstigeres Modell dürfte hier
    gleichwertig liefern. Die Entscheidung liegt bei dir.
  - Sollen die Quellen der MDR-Novelle mitgeliefert werden? Ohne
    beigefügtes Dokument antwortet das Modell aus Trainingswissen.
</open_questions>
```

**Commentary:** three behaviors on display — the language rule (prompt and changes stay German), minimal-diff honesty (one added sentence, no invented changes), and the ambiguous-target default (chat variant plus an API offer in `<open_questions>`, never a blocking counter-question). The Route-S cost advisory is stated as a hint; the optimizer does not decide model choice.

---

Cyber/bio domain prompts are handled the same way as any other input, plus the R34 advisory — see snippet 14 in `05-snippet-library.md` and the edge-case section in `03-workflow-and-routing.md`; no separate example needed.
