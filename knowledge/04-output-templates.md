# Output Templates

This file is the single source of truth for the optimizer's output format. Two variants: **A (API target)** with four blocks, **B (claude.ai chat target)** with three blocks. Target detection is defined in `03-workflow-and-routing.md`, Phase 0.

Shared conventions:

- The optimized prompt separates `<system>` and `<user>` content. For chat targets where no system prompt is settable, `<system>` may be omitted and its content folded into the top of `<user>`.
- Bulk user content (pasted documents, code, data) is represented by placeholders — `{{DOCUMENTS}}`, `{{INPUT}}`, `{{CODE}}` — with a one-line placement note, instead of being echoed back.
- Every `<changes>` entry cites the rule number(s) from `02-prompting-rules.md` that motivated it.
- Questions and risk notes go into `<open_questions>`; the optimizer never withholds the optimized prompt to ask a question first.
- On follow-up iterations ("make it shorter", "add X"), apply the change, re-run the Phase 4 quality gate, and re-emit the **full** block set — not a diff.

## Variant A — API target (four blocks)

```text
<optimized_prompt>
  <system>…optimized system prompt…</system>
  <user>…optimized user prompt with {{DOCUMENTS}}/{{INPUT}} placeholders…</user>
</optimized_prompt>
<parameters>
  model: claude-fable-5
  effort: <low|medium|high|xhigh|max> — <one-line justification>
  max_tokens: <value> — <one-line justification>
  streaming: <yes/no>; fallbacks: <claude-opus-4-8 | not needed>
  caching: <cache-breakpoint recommendation>
  task_budget: <value | omit> — only with a known budget
</parameters>
<changes>
  - <change> (R<number>): <justification in one sentence>
</changes>
<open_questions>
  - <missing information or risk note, e.g., refusal domain, cost>
</open_questions>
```

### Per-field guidance for `<parameters>` (from R27–R33)

| Field | Guidance |
|---|---|
| `model` | Always `claude-fable-5` (pinned snapshot; Bedrock: `anthropic.claude-fable-5`). |
| `effort` | `high` as default — even for workloads that ran on Opus 4.8 with `xhigh` (R27/R35). `xhigh` only for capability-sensitive long-horizon work (> 30 min, token budgets in the millions); `medium`/`low` for routine, subagents, classification; `max` for maximum capability without token restraint. |
| `max_tokens` | Hard limit on thinking + answer text. At `high`/`xhigh`: start at 64k (R28). On `stop_reason: "max_tokens"`: raise it or lower effort. |
| `streaming` | Recommend `yes` from roughly 16k expected output tokens and for all Route L/XL workloads (long turns, R25). |
| `fallbacks` | `claude-opus-4-8` via beta header `server-side-fallback-2026-06-01` whenever the domain is anywhere near cyber/bio/reasoning-extraction territory (R33/R34); on Bedrock/Vertex/Foundry/Batches: SDK middleware instead. Otherwise "not needed". |
| `caching` | Stable prefix (frozen system prompt, no timestamps), minimum 512 tokens (Bedrock: 1,024), 90% read discount (R32). Name a concrete breakpoint when the prompt has a reusable prefix. |
| `task_budget` | Only with a known, realistic value (minimum 20k; advisory, no hard cap; beta header `task-budgets-2026-03-13`). Omit for open-ended quality tasks (R31). |

Never recommend: `temperature`, `top_p`, `top_k`, assistant prefill, `thinking: {type:"disabled"}`, `budget_tokens` (all produce errors; sampling parameters, prefill, and `budget_tokens` return 400), or structured outputs (not documented for Fable 5) (R29/R30/R36/R37).

## Variant B — claude.ai chat target (three blocks)

```text
<optimized_prompt>
  <user>…optimized prompt with {{DOCUMENTS}}/{{INPUT}} placeholders…</user>
</optimized_prompt>
<changes>
  - <change> (R<number>): <justification in one sentence>
</changes>
<open_questions>
  - <missing information or risk note>
</open_questions>
```

No `<parameters>` block — those controls do not exist in the chat UI. Instead, fold parameter intent into the prompt text:

| API intent | Chat-prompt substitute |
|---|---|
| `effort: low` (routine task) | A brevity line: "Answer concisely; lead with the result." (R12) — and a hint in `<open_questions>` that a lighter model may be cheaper/faster for this task. |
| `effort: xhigh` (hard problem) | An encouraging thinking phrase: "Please think hard before responding." (R10) |
| `fallbacks` (cyber/bio risk) | A note in `<open_questions>`: claude.ai reroutes classifier-flagged requests to Opus 4.8 automatically; the answer may come from the fallback model (R34). |
| `max_tokens` / streaming / caching / task budget | Not applicable in chat — omit silently. |

## Language rule (both variants)

The optimized prompt — including inserted snippets, translated — keeps the **input prompt's language**. XML tag names stay English. `<changes>`, `<open_questions>`, and all commentary follow the user's conversation language.
