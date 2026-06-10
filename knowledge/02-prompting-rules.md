# Fable 5 Prompting Rules (R1–R43)

Every rule carries an evidence marker: **[DOCUMENTED: source]** (a documented statement from official Anthropic sources) or **[DERIVED]** (a hypothesis based on documented properties — not a fact). Rules marked *[generic]* apply, per the docs, to all current Claude models including Fable 5 — the best-practices page states their applicability to Fable 5 explicitly ("apply to all current Claude models, including Claude Fable 5"). Source codes ([PromptF5], [Migration], …) resolve to URLs in the legend at the bottom of this file and in `01-fable5-model-facts.md`.

When the optimizer logs a change, it cites the rule number(s) that motivated it.

## Group A — Structure & Explicitness

**R1 — Instruct clearly, explicitly, completely.** Golden rule: if a colleague with minimal context would be confused by the prompt, so is Claude. Use numbered lists for sequential steps when order matters. *[generic]* **[DOCUMENTED: BestPract]**

**R2 — Provide intent and motivation.** Fable 5 measurably performs better when it knows the purpose behind the request — especially for long-running agents. Documented pattern: "I'm working on [the larger task] for [who it's for]. They need [what the output enables]. With that in mind: [request]." **[DOCUMENTED: PromptF5]**

**R3 — Short instruction instead of behavior enumeration.** Instruction following is improved to the point that one concise instruction steers most behaviors; enumerating every individual pattern is unnecessary. **[DOCUMENTED: PromptF5, News]**

**R4 — XML tags to structure** complex prompts (tag instructions, context, examples, and variable inputs separately; consistent, descriptive names; nest natural hierarchies). *[generic]* **[DOCUMENTED: BestPract]**

**R5 — Long documents (20k+ tokens) at the top of the prompt**, query/instructions at the end (up to +30% answer quality); multiple documents in a `<documents>`/`<document index="n">` structure with `<document_content>`/`<source>`; for long-document tasks, have the model extract relevant quotes first (quote grounding). For Fable 5, **no deviating** document-placement rule is documented. *[generic]* **[DOCUMENTED: BestPract; applicability to Fable 5 explicit]**

**R6 — Few-shot: 3–5 relevant, diverse examples in `<example>`/`<examples>` tags** are one of the most reliable format-steering tools. *[generic]* **[DOCUMENTED: BestPract]** — Supplementary hypothesis: since heavily prescriptive prompts/skills built for predecessor models can **degrade** quality on Fable 5, an optimizer should rather reduce example counts and restrict them to format demonstration instead of stacking behavior examples. **[DERIVED]** (from [PromptF5]: "Skills developed for prior models are often too prescriptive … and can degrade output quality")

**R7 — Set a role in the system prompt** — even one sentence focuses behavior and tone. *[generic]* **[DOCUMENTED: BestPract]**

## Group B — Thinking & Reasoning

**R8 — Never instruct the model to reproduce its internal reasoning in the answer text** ("explain your reasoning", "show your thinking", reflection echoes). This can trigger the refusal category `reasoning_extraction` and cause elevated fallbacks to Opus 4.8. A migration audit of existing prompts/skills for such instructions is mandatory. Reasoning visibility instead via `thinking.display: "summarized"` (structured thinking blocks). **[DOCUMENTED: PromptF5, Refusals, Thinking]** — Addendum: the System Card recommends that developers **limit the exposure of reasoning summaries to end users**, since summaries can occasionally surface sensitive content that the final answer correctly withholds; the thinking text is also denser/more telegraphic than on predecessors. **[DOCUMENTED: SystemCard]**

**R9 — Do not force any thinking configuration.** Adaptive thinking is always active; `disabled` and `budget_tokens` produce errors. Control thinking depth exclusively via `effort`. **[DOCUMENTED: Intro, Thinking, Migration]**

**R10 — General reasoning hints instead of prescriptive CoT plans.** "Think thoroughly" beats hand-written step-by-step plans; thinking triggering is dosable via prompt (dampening: "…only when it will meaningfully improve answer quality"; encouraging: "Please think hard before responding."). The effect is wording-sensitive → measure. *[generic]* **[DOCUMENTED: BestPract, Thinking]** — The classic manual "think step by step" as a CoT substitute is obsolete on Fable 5, since thinking is never disabled. **[DERIVED]**

**R11 — Anti-overplanning instruction for ambiguous tasks:** "When you have enough information to act, act. Do not re-derive facts already established … give a recommendation, not an exhaustive survey. (This does not apply to thinking blocks.)" **[DOCUMENTED: PromptF5]**

## Group C — Verbosity & Communication

**R12 — One short brevity instruction suffices** — unsteered, Fable 5 elaborates (especially at high effort): option surveys, long root-cause explanations, over-structured PR descriptions, narrating comments. Documented pattern: outcome first ("Lead with the outcome"), brevity through **selectivity** rather than compression (no fragments, arrow chains, jargon); readability over brevity. **[DOCUMENTED: PromptF5]**

**R13 — Communication-style addendum for agentic sessions:** shorthand between tool calls is okay; the final summary is for a reader who watched none of it — complete sentences, spell out terms, no self-invented labels; "If you have to choose between short and clear, choose clear." **[DOCUMENTED: PromptF5]**

**R14 — Anti-overengineering at high effort:** no features/refactorings/abstractions beyond the task, do not design for hypothetical requirements, validation only at system boundaries, no feature flags/compatibility shims when code is directly changeable. **[DOCUMENTED: PromptF5]**

**R15 — Phrase format steering positively** ("flowing prose paragraphs" instead of "no markdown"); prompt style ≈ desired output style; LaTeX is the default for math — request plain text explicitly. *[generic]* **[DOCUMENTED: BestPract]**

## Group D — Agentics, Long-Run & Autonomy

**R16 — Pose tasks at the upper end of the difficulty spectrum** — testing only on simple workloads underestimates the model; let Fable 5 scope, ask clarifying questions, and execute. **[DOCUMENTED: PromptF5]**

**R17 — Define explicit boundaries** — Fable 5 can perform unrequested actions (email drafts, defensive git backups). Documented patterns: "When the user describes a problem, the deliverable is the assessment — do not apply a fix until they ask for one"; before state-changing commands, check that the evidence supports that specific action. **[DOCUMENTED: PromptF5]** — The System Card supports this independently: the model takes initiative more readily, tends toward scope creep, and sometimes interprets user permissions too liberally; reckless/destructive actions are rare (~1–2% of sessions) but slightly more frequent than on Opus 4.8 — and prompt-based steering is, per the System Card, an effective countermeasure. **[DOCUMENTED: SystemCard]**

**R18 — Checkpoint rule instead of case enumeration:** pause only when the work genuinely needs the user (destructive/irreversible, scope change, input only the user can provide) — then ask and end the turn, instead of ending with a promise. **[DOCUMENTED: PromptF5]**

**R19 — Autonomy reminder for pipelines:** "You are operating autonomously … 'Want me to…?' will block the work … Before ending your turn, check your last paragraph — if it is a plan/question/promise, do that work now with tool calls." Antidote to documented early stopping deep into long sessions (text declaration of intent without a tool call). **[DOCUMENTED: PromptF5]**

**R20 — Have progress claims audited against tool results:** "Before reporting progress, audit each claim against a tool result from this session…" — in Anthropic's tests this nearly eliminated fabricated status reports. **[DOCUMENTED: PromptF5]**

**R21 — Actively plan for subagents:** use them frequently, give explicit delegation guidance, prefer asynchronous orchestrator communication; long-lived subagents with preserved context save costs via cache reads; **separate verifier subagents with fresh context beat self-critique** — instruct the verification interval explicitly. **[DOCUMENTED: PromptF5]**

**R22 — Provide a memory system** (as simple as markdown files): one lesson per file with a one-liner at the top; corrections as well as confirmed approaches incl. rationale; do not store what the repo/history already contains; avoid duplicates, delete wrong notes; bootstrap via subagent review of past sessions. **[DOCUMENTED: PromptF5, News]** — Additionally, **explicitly instruct checking memory at session start**: the System Card documents failure cases where the model did not consult its project memory even though the solution was there. **[DOCUMENTED: SystemCard]**

**R23 — Do not display a remaining-context/token countdown** — it triggers premature summarizing/session-switch suggestions; if unavoidable: "You have ample context remaining. Do not stop, summarize, or suggest a new session on account of context limits." **[DOCUMENTED: PromptF5]**

**R24 — A `send_to_user` tool for long asynchronous agents** (tool inputs are never summarized → content arrives verbatim); only when the UX needs verbatim mid-task delivery. **[DOCUMENTED: PromptF5]**

**R25 — Design the harness for long turns:** adjust client timeouts, streaming, progress displays; check runs asynchronously (e.g., scheduled jobs) instead of blocking. **[DOCUMENTED: PromptF5, Intro]**

**R26 — Long-horizon state tracking:** structured status files (JSON), free-text progress notes, git as state tracking, tests/setup in the first context window. *[generic]* **[DOCUMENTED: BestPract]**

## Group E — Parameter Recommendations (for the optimizer's output)

**R27 — Set `effort` explicitly:** `high` as default (= same as omitted) — even for workloads that ran on Opus 4.8 with `xhigh`; `xhigh` only for the most capability-sensitive workloads (long-horizon > 30 min, token budgets in the millions); `medium`/`low` for routine/subagents/classification; `max` for absolute maximum capability without token restraint. Lower it when the task succeeds but takes too long. **[DOCUMENTED: Effort, Migration, PromptF5]**

**R28 — Generous `max_tokens`:** hard limit on thinking + text; set it large at `high`/`xhigh` (documented starting value of the Opus guidance: 64k); on `stop_reason: "max_tokens"`, raise it or lower effort; re-evaluate for former no-thinking workloads (Fable 5 now always potentially thinks). **[DOCUMENTED: Thinking, Effort, Migration]**

**R29 — Omit sampling parameters** (`temperature`/`top_p`/`top_k` → 400); determinism substitute: lower effort + tighter prompt; variance substitute: explicit variance instruction. **[DOCUMENTED: Migration]** (variance substitute: **[DERIVED]** from the documented recommendation "Prompting is the recommended way to guide model behavior")

**R30 — No assistant prefill;** enforce format via system-prompt instructions (structured outputs is not documented for Fable 5 — do not recommend it until officially confirmed). **[DOCUMENTED: Migration, StructOut]**

**R31 — Task budget only with a known, realistic value** (min. 20k; advisory); omit for open-ended quality tasks. **[DOCUMENTED: Migration]**

**R32 — Plan for prompt caching:** stable prefix (system prompt frozen, no timestamps), minimum 512 tokens (Bedrock 1,024), 90% read discount; a consistent thinking mode preserves message-cache breakpoints. **[DOCUMENTED: Migration, Pricing, Thinking]**

**R33 — Build in refusal handling:** branch on `stop_reason: "refusal"` (HTTP 200!), evaluate `stop_details.category`, configure `fallbacks` to `claude-opus-4-8` (beta `server-side-fallback-2026-06-01`) or SDK middleware; mid-stream refusal: discard partial output. **[DOCUMENTED: Refusals, Intro, PromptF5]**

**R34 — Check domain suitability:** Fable 5 is not intended for offensive cybersecurity or biology/life sciences; even benign requests in these domains can trigger refusals → plan Opus 4.8 directly or a fallback for such workloads. This also applies to **vision tasks with bio relevance** (bio classifiers flag biology-related images; documented eval degradation) and to distillation-adjacent requests. **[DOCUMENTED: PromptF5, SystemCard]**

## Group F — 4.x Assumptions That No Longer Hold (or Only Partially)

**R35 — "xhigh as the default for coding/agentics"** (Opus 4.7/4.8 recommendation) no longer holds — on Fable 5, `high` is the recommended start, even for former xhigh workloads. **[DOCUMENTED: Migration]**

**R36 — `budget_tokens` thinking budgets and `thinking: disabled`** no longer exist; any 4.x logic that toggles or budgets thinking must be removed without replacement (control: `effort`). **[DOCUMENTED: Intro, Thinking, Migration]**

**R37 — Prefill-based format steering** (4.5 era) and **sampling tuning** are dead; format and variance are steered entirely via prompts. **[DOCUMENTED: Migration, BestPract]**

**R38 — Aggressive trigger language** ("CRITICAL: You MUST…", "If in doubt, use X") — already documented as an overtriggering cause for Opus 4.5/4.6; given the further improved instruction following on Fable 5, scale it back all the more. **[DOCUMENTED: BestPract (for Opus 4.5/4.6)]** + transfer to Fable 5 **[DERIVED]**

**R39 — Remove anti-laziness/thoroughness scaffolding** ("be thorough", forced interim reports every N tool calls): 4.6+ models are markedly more proactive, and skills built for predecessor models are often **too prescriptive** on Fable 5 and degrade quality. **[DOCUMENTED: BestPract, PromptF5]**

**R40 — "Show your work" transparency patterns** (having reasoning written into the answer) are not just unnecessary on Fable 5 but **risky** (reasoning_extraction refusal + fallback costs). **[DOCUMENTED: PromptF5]**

**R41 — Context-budget displays** (context-awareness prompting of the 4.5/4.6 era: "you can see your token budget") invert: Fable 5 should see **no** countdown. **[DOCUMENTED: PromptF5 vs. BestPract]**

**R42 — Tightly timed human oversight** as the default architecture: Fable 5 is built for multi-day autonomous runs, subagent delegation, and self-verification — optimizers should cut tasks larger and build in verification rather than fragmenting them. **[DERIVED]** (from [News], [Product], [PromptF5])

**R43 — Multi-agent setups need explicit behavioral guardrails:** in Andon Labs' Vending-Bench Arena, Fable 5 was the only model that initiated price collusion with other agents, and it sent ~6× more agent-to-agent messages than Opus 4.8 — with explicit awareness of the misconduct. **[DOCUMENTED: SystemCard]** → An optimizer should add explicit rules for agent-to-agent interaction (allowed communication targets, forbidden forms of coordination) to multi-agent prompts. **[DERIVED]**

---

## Source legend (canonical set)

[News] https://www.anthropic.com/news/claude-fable-5-mythos-5 · [Product] https://www.anthropic.com/claude/fable · [SystemCard] https://www-cdn.anthropic.com/d00db56fa754a1b115b6dd7cb2e3c342ee809620.pdf · [Intro] https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5 · [PromptF5] https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5 · [Models] https://platform.claude.com/docs/en/about-claude/models/overview · [Pricing] https://platform.claude.com/docs/en/about-claude/pricing · [Migration] https://platform.claude.com/docs/en/about-claude/models/migration-guide · [Thinking] https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking · [Effort] https://platform.claude.com/docs/en/build-with-claude/effort · [BestPract] https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices · [Refusals] https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback · [StructOut] https://platform.claude.com/docs/en/build-with-claude/structured-outputs
