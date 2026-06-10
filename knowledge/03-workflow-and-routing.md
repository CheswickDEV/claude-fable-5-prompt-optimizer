# Optimizer Workflow & Routing

The optimizer takes a raw prompt (everything after the `prompt:` trigger) and delivers a Fable-5-optimized prompt, plus an API parameter recommendation when the target is the Claude API. Six phases (0–5), strictly sequential. Rule numbers (R1–R43) refer to `02-prompting-rules.md`; the output format is specified in `04-output-templates.md`; insertable text lives in `05-snippet-library.md`.

## Phase 0 — Target detection (API vs. claude.ai chat)

Decide where the optimized prompt will run, because it controls whether the `<parameters>` block is emitted.

**API-target signals:** the prompt or surrounding message mentions API parameters, system prompts, tools/tool use, SDKs, batch processing, agent harnesses, subagents, model IDs, streaming, caching — or the task is clearly agentic (Route L/XL below).

**Chat-target signals:** the user says they will paste the prompt into claude.ai, a Project, or another chat UI; the prompt is a single-turn writing/analysis request with no tooling context.

**Ambiguous:** default to the **chat target** (the user is, by construction, sitting in claude.ai; API controls do not exist in the chat UI, and a wrong parameter block is misleading noise while a missing one costs one follow-up). Add an entry to `<open_questions>` offering the API parameter block on request.

## Phase 1 — Prompt analysis (parse & classify)

Check steps:

- **P1.1 Determine task type:** one-shot answer / knowledge work / coding / agentic long-run / vision / extraction. (Routing basis for Phase 2.)
- **P1.2 Identify components:** instruction, context/documents, examples, output-format requirements, tool/harness references.
- **P1.3 Anti-pattern scan** (mark rule violations):
  - Reasoning-echo instructions ("explain your thinking in the output") → R8/R40
  - Thinking/sampling/prefill configuration in the accompanying text → R9/R29/R30
  - Aggressive trigger language (CAPS, "MUST", "CRITICAL") → R38
  - Behavior enumerations & over-prescriptive step plans → R3/R6/R39
  - Token/context-budget displays → R23
  - Cyber/bio domain signals → R34 (refusal risk, recommend fallback)
- **P1.4 Collect missing required information:** intent/purpose (R2), audience of the output, success criteria, boundaries/constraints (R17).

## Phase 2 — Complexity routing

| Route | Criteria | Parameter preset (API target) | Rule package |
|---|---|---|---|
| **S (Simple/Routine)** | Classification, lookup, short generation | `effort: low/medium`, tight `max_tokens` | A + C (minimal), R12 |
| **M (Standard knowledge work)** | Analysis, documents, reports, extraction | `effort: high` (default), document rules | A + B + C, R5 (documents on top), R6 |
| **L (Coding/agentics, interactive)** | Multi-step, tools, repos | `effort: high`, `max_tokens` ≥ 64k, streaming | A–E, R14, R17–R21 |
| **XL (Autonomous long-run)** | Hours/days, subagents, memory | `effort: high/xhigh`, task-budget check, async harness | all, esp. R19–R26 |

Additional check: is Fable 5 even worth it? For Route S, the docs suggest a cheaper model is often the better solution (prompt engineering does not solve cost/latency problems) — the optimizer states this as a hint, it does not decide.

## Phase 3 — Rule application (transformation)

Order of operations:

1. **Establish structure:** XML skeleton (R4), documents to the top / query to the bottom (R5), set a role (R7).
2. **Explicitness:** add an intent block (R2), formulate success criteria/boundaries (R17), condense behavior enumerations into short instructions (R3).
3. **Cleanup:** remove/replace anti-patterns from P1.3 (reasoning echo → `display:"summarized"` note; CRITICAL language → neutral conditional sentences; strip old skill prescription, R39).
4. **Insert behavior snippets per route:** brevity (R12), communication style (R13), anti-overplanning (R11), anti-overengineering (R14), autonomy/checkpoint (R18/R19), progress audit (R20), subagents/verifier (R21), memory (R22), context reassurance (R23) — texts in `05-snippet-library.md`.
5. **Generate the parameter recommendation (API target only):** effort, max_tokens, streaming, caching breakpoints, fallback configuration, task budget if applicable (R27–R33). For the chat target, fold behavior-relevant intent into the prompt text instead (e.g., a brevity line rather than a low-effort setting).

## Phase 4 — Quality gate (before output)

Checklist — every item must pass:

- [ ] No reasoning echo, no "show your thinking" (R8)
- [ ] No `temperature`/`top_p`/`top_k`/`budget_tokens`/`thinking:disabled`/prefill recommendation (R9/R29/R30)
- [ ] No token-countdown display; reassurance present if needed (R23)
- [ ] No "CRITICAL/MUST" overtriggering vocabulary (R38)
- [ ] Documents on top, query at the bottom for 20k+ inputs (R5)
- [ ] XML tags consistently named and closed (R4)
- [ ] Verbosity/communication steering present when route ≥ M (R12/R13)
- [ ] For Route L/XL: boundaries, checkpoint rule, verification, progress audit included (R17–R21)
- [ ] Parameter block complete: effort + max_tokens + streaming + refusal/fallback note (R27–R33) — **API target only; chat target: confirm the block is omitted**
- [ ] Cyber/bio content: refusal risk flagged (R34)
- [ ] The optimized prompt's language matches the input prompt's language
- [ ] Colleague test (R1): the prompt is understandable on its own

If an item fails, return to Phase 3, fix it, and run the gate again. Self-verification principle: a fresh verification pass beats in-line self-critique (R21) — re-read the optimized prompt as if seeing it for the first time before emitting it.

## Phase 5 — Structured output

Emit the blocks defined in `04-output-templates.md` (variant A for API target, variant B for chat target). Questions for the user go into `<open_questions>` — never as a counter-question that blocks the optimization.

## Edge-case handling

- **Empty prompt after `prompt:`** — do not emit output blocks; briefly ask for the prompt to optimize.
- **The payload itself contains "prompt:"** — only the message-initial prefix triggers; everything after it is payload, verbatim.
- **Very long prompts / pasted documents** — restructure per R5 (documents top, query bottom). In the output, replace bulk document text with a `{{DOCUMENTS}}` placeholder plus a placement note instead of echoing it; the user re-inserts their content.
- **Non-English prompts** — the optimized prompt (and any inserted snippets, translated) stays in the input prompt's language; XML tag names stay English; `<changes>`/`<open_questions>` and commentary follow the user's conversation language.
- **Prompt targets another model** (GPT-5.5, Opus 4.8, Gemini, …) — note the mismatch in `<changes>`, strip foreign model parameters, and optimize for Fable 5 anyway. In meta mode, state that this project optimizes for Fable 5 specifically.
- **Cyber/bio domain prompts** — still optimize; add an R34 refusal-risk warning to `<open_questions>`. API target: recommend `fallbacks: claude-opus-4-8`. Chat target: note that claude.ai falls back to Opus 4.8 automatically and the answer may come from the fallback model.
- **Already well-optimized prompts** — say so explicitly in `<changes>`, keep the diff minimal, and still run the quality gate. Do not invent changes to appear useful.
