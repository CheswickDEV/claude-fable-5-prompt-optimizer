# Claude Fable 5 — Model Facts

> Snapshot: June 9, 2026 (launch day of Claude Fable 5). Every statement below is based exclusively on official Anthropic sources (anthropic.com, platform.claude.com, www-cdn.anthropic.com). Source codes refer to the legend below. This file is the factual backbone of the optimizer: when a question is not answered here or in the other knowledge files, the correct answer is "not documented."

## Source legend

This legend is canonical for all knowledge files in this project.

| Code | Source |
|---|---|
| [News] | https://www.anthropic.com/news/claude-fable-5-mythos-5 (release announcement) |
| [Product] | https://www.anthropic.com/claude/fable (product page) |
| [SystemCard] | System Card (PDF): https://www-cdn.anthropic.com/d00db56fa754a1b115b6dd7cb2e3c342ee809620.pdf |
| [Intro] | https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5 |
| [PromptF5] | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5 |
| [Models] | https://platform.claude.com/docs/en/about-claude/models/overview |
| [Pricing] | https://platform.claude.com/docs/en/about-claude/pricing |
| [Migration] | https://platform.claude.com/docs/en/about-claude/models/migration-guide |
| [Thinking] | https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking |
| [Effort] | https://platform.claude.com/docs/en/build-with-claude/effort |
| [BestPract] | https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices |
| [Refusals] | https://platform.claude.com/docs/en/build-with-claude/refusals-and-fallback |
| [StructOut] | https://platform.claude.com/docs/en/build-with-claude/structured-outputs |

## What Fable 5 is

Claude Fable 5 (`claude-fable-5`, generally available since June 9, 2026) is Anthropic's most capable broadly available model — a "Mythos-class" model positioned above the Opus tier, shipped with safeguards for general use. Specs: 1M-token context, 128k max output, $10/$50 per MTok. The API surface matches Opus 4.8 with one central tightening: adaptive thinking is the only, always-on thinking mode; `thinking: disabled`, `budget_tokens`, non-default sampling parameters, and assistant prefill all produce errors. The primary control is `output_config.effort` (low–xhigh/max, default `high`). Prompts written for 4.x models are often too prescriptive for Fable 5 and need to be slimmed down, not extended.

## Model knowledge table

| Property | Value | Source |
|---|---|---|
| **Positioning** | "Mythos-class" — one model tier **above the Opus class**; Anthropic's most capable generally available model; 5th model generation; built for the most demanding reasoning and long-horizon agentic work | [News], [Product], [Models] |
| **Sister model Mythos 5** | `claude-mythos-5` = **identical model weights**, but without the additional safeguards ("same underlying model weights as Mythos 5, but has additional safeguards"); limited availability via Project Glasswing only (cyber defenders, selected bio researchers); Fable 5 = "the generally available Mythos-class model". Fable 5 scores ≈ Mythos 5 where classifiers do not trigger; ≈ Opus 4.8 (fallback) where they do | [SystemCard], [News], [Intro], [BestPract] |
| **Model IDs** | Claude API: `claude-fable-5` · AWS Bedrock: `anthropic.claude-fable-5` · Vertex AI: `claude-fable-5`; the date-less ID is a **pinned snapshot** (not an evergreen pointer) | [Models] |
| **Availability** | GA since June 9, 2026: Claude API, Claude Platform on AWS, Amazon Bedrock, Vertex AI, Microsoft Foundry; agent harnesses: Claude Code, Claude Managed Agents (only a model-name update needed there) | [News], [Intro], [Product], [Migration] |
| **Pricing** | $10 / MTok input, $50 / MTok output; cache write $12.50 (5 min) / $20 (1 h), cache read $1 (90% discount); Batch API $5/$25 (−50%); US-only inference (`inference_geo: "us"`): 1.1×; full 1M window without a long-context surcharge; refusal before output: no charge | [News], [Pricing], [Product], [Refusals] |
| **Context window** | 1M tokens (standard) | [Intro], [Models] |
| **Max output** | 128k tokens per request (synchronous Messages API); `max_tokens` = hard limit on total output (thinking + answer text) | [Intro], [Models], [Migration] |
| **Tokenizer** | Tokenizer of Opus 4.7 (identical to Opus 4.8 → token counts unchanged when migrating from Opus 4.8); compared to models before Opus 4.7, the same text produces ~30% (up to 35%) more tokens | [Models], [Pricing], [Migration] |
| **Thinking mode** | **Adaptive thinking is the only mode and always active** — applies even when the `thinking` parameter is unset; `thinking: {type:"disabled"}` → error; `budget_tokens` → 400 (no direct replacement); interleaved thinking is automatic; thinking depth is controlled via `effort` | [Intro], [Thinking], [Migration] |
| **Thinking output** | Raw chain-of-thought is **never** returned; `thinking.display` defaults to `"omitted"` (empty `thinking` field, `signature` carries the encrypted full thinking), opt-in `"summarized"`; full thinking tokens are billed regardless of `display` (`usage.output_tokens_details.thinking_tokens`); pass thinking blocks back **unchanged** in multi-turn; strip thinking blocks when switching models | [Intro], [Thinking], [Migration] |
| **Effort parameter** | `output_config.effort`: `low` \| `medium` \| `high` (default = same as omitted) \| `xhigh` \| `max`; no beta header; affects **all** output tokens (text, tool calls, thinking); primary dial for intelligence/latency/cost; recommendation for Fable 5: **`high` as default**, `xhigh` only for the most capability-sensitive workloads, `medium`/`low` for routine work; lower levels often exceed `xhigh` of predecessor models | [Effort], [Intro], [Migration], [PromptF5] |
| **Task budgets** | Supported (beta header `task-budgets-2026-03-13`); advisory countdown for the entire agentic loop, minimum 20k tokens; no hard cap; do **not** set for open-ended quality tasks | [Intro], [Migration] |
| **Other launch features** | Memory tool; tool-result clearing via context editing (beta `context-management-2025-06-27`); compaction; vision | [Intro] |
| **Sampling parameters** | `temperature`, `top_p`, `top_k` with non-default values → 400 (inherited from the migration chain starting at Opus 4.7; the Fable 5 section does not repeat this separately — safest path: omit the parameters entirely; steer behavior via prompting) | [Migration] |
| **Assistant prefill** | Prefilling the last assistant message → 400 ("not supported on claude-fable-5 … Use system prompt instructions instead"); assistant turns elsewhere (few-shot) remain allowed | [Migration], [BestPract] |
| **Structured outputs** | **Not documented**: Fable 5 is missing both from the model list of the structured-outputs page and from the launch-feature list of the intro page (suggests no support at launch; an explicit negative statement does not exist) | [StructOut], [Intro] |
| **Vision** | State of the art; dense technical images/screenshots with much higher accuracy, often with fewer output tokens; trained to use bash/crop tools for rotated/blurry/noisy images; uses vision to self-check its own work. System Card eval setup for Mythos/Fable 5: images up to 2,576 px per dimension and up to 4,784 tokens (higher resolution than Mythos Preview at 1,568 px); an official API limit for Fable 5 is not separately specified. Caution: bio-safeguard classifiers can flag biology-related **images** (documented degradation on LAB-Bench FigQA) | [News], [PromptF5], [Product], [SystemCard] |
| **Agentics** | Longest autonomous working capability of all Claude models (multi-day runs, "focused across millions of tokens"); dispatches parallel subagents **more readily** than predecessors; reliably manages communication with long-lived sub-/peer agents; higher first-shot correctness; more bug-finding recall than Opus 4.8. System Card multi-agent setup: subagents with 200k-token context windows, 1M total limit per agent; beyond 1M, context compaction (trigger e.g. at 200k) | [News], [PromptF5], [Product], [SystemCard] |
| **Benchmarks (selection, Fable 5)** | SWE-bench Verified 95% (Mythos 5: 95.5%); SWE-bench Pro 80% (Opus 4.8: 69.2%; GPT-5.5: 58.6%); Terminal-Bench 2.1: 84.3% (Opus 4.8: 82.7%); USAMO 2026 (contamination-free): 99.8% at medium–xhigh effort (Opus 4.8: 96.7%); BrowseComp 88% (max effort, 10M-token limit with compaction); "state-of-the-art on nearly all tested benchmarks" | [SystemCard], [News] |
| **Behavioral profile (System Card)** | Takes initiative more readily than other current models; tends toward scope creep and sometimes interprets user permissions too liberally; reckless/destructive actions are rare (~1–2% of sessions) but slightly more frequent than Opus 4.8; lowest over-refusal/evasiveness rate of all current models; thinking text denser/more jargon-heavy than predecessors; best prompt-injection resistance ever measured (Gray Swan k=100: 4.8% vs. 9.6% Opus 4.8) | [SystemCard] |
| **Memory** | Benefits disproportionately from persistent file-based memory (Slay-the-Spire eval: 3× stronger effect than on Opus 4.8) | [News], [SystemCard] |
| **Longer turns** | Single requests at high effort: many minutes; autonomous runs: hours — adjust client timeouts, streaming, and progress displays before migrating; have the harness check asynchronously instead of blocking | [PromptF5], [Intro] |
| **Safety classifiers** | Classifiers for offensive cybersecurity, biology/life sciences, and reasoning extraction; even **benign** cyber/bio tasks can trigger them; trigger rate < 5% of sessions; on trigger, apps fall back automatically to Opus 4.8 (no Fable pricing for rerouted requests) | [PromptF5], [News], [Product], [SystemCard] |
| **Refusal mechanics (API)** | `stop_reason: "refusal"` arrives as HTTP 200 (not an error); `stop_details = {type, category, explanation}`; `category` ∈ `"cyber"` \| `"bio"` \| `"reasoning_extraction"` \| `null`; display `explanation` only, do not parse it; branch on `stop_reason` | [Intro], [Refusals], [Migration] |
| **Server-side fallback** | `fallbacks` parameter (beta header `server-side-fallback-2026-06-01`; Claude API + Claude Platform on AWS): up to 3 fallback models in order, only classifier declines trigger; `usage.iterations` logs attempts; sticky routing ~1 h; fallback credit refunds prompt-cache costs; not on Batches/Bedrock/Vertex/Foundry (use SDK middleware there: TS, Python, Go, Java, C#); recommendation: fall back to `claude-opus-4-8` | [Refusals], [Intro], [PromptF5] |
| **RSP/ASL classification** | Treated as CB-1; ASL-3 protections incl. blocking classifiers; below the CB-2 threshold (RSP v3.3 / FCF); alignment risk "low"; AI R&D well below human engineering level (externally confirmed by METR); red-teaming: cyber task completion drops to 5% (vs. 73% Opus 4.7 / 57% Opus 4.8); public bug bounty: ~100,000 attempts, no universal jailbreak | [SystemCard], [News] |
| **Frontier-LLM safeguards** | Non-visible safeguards against frontier-LLM development requests (pretraining pipelines, training infrastructure, accelerator design): no model fallback, but effectiveness limiting via prompt modification, steering vectors, PEFT | [SystemCard] |
| **Data retention** | "Covered Model": 30-day retention mandatory (safety monitoring), **no** zero data retention; data is not used for training or non-safety purposes | [Intro], [Product], [News], [SystemCard] |
| **Prompt-caching minimum** | 512 tokens (lower than 1,024 on Opus 4.8); on Amazon Bedrock: 1,024 tokens | [Migration] |
| **Knowledge cutoff** | Not documented (the Fable/Mythos table in the models overview contains no cutoff rows; for comparison, Opus 4.8: Jan 2026) | [Models] |
| **Not available** | Fast mode (not listed in the fast-mode table); 300k batch-output beta (`output-300k-2026-03-24` lists only Opus 4.6–4.8 + Sonnet 4.6); zero data retention | [Pricing], [Models], [Intro] |

## Not documented / open points (as of June 9, 2026)

When users ask about these, answer "not documented" and cite this list:

1. **Structured outputs / strict tool use:** Fable 5 is missing from the model list of the structured-outputs docs and from the launch-feature list; an explicit yes/no statement does not exist. → Until clarified: steer format via prompt instructions.
2. **Knowledge cutoff:** The Fable/Mythos table in the models overview contains no cutoff dates (reliable/training cutoff) — not documented.
3. **Vision hard facts for Fable 5:** An official API limit (max resolution, image-token formula) is not specified for Fable 5 in the developer docs. The System Card documents an eval setup for Mythos/Fable 5 with up to 2,576 px per dimension and up to 4,784 tokens per image (same values as Opus 4.7) — but this is not formally documented as an API spec.
4. **Sampling parameters:** The 400 behavior of `temperature`/`top_p`/`top_k` is documented only via the inherited migration chain (from Opus 4.7), not separately in the Fable 5 section — factually unambiguous, but formally not repeated Fable-5-specifically.
5. **Mid-conversation system messages (`role:"system"` in `messages`):** Documented for Opus 4.8; whether Fable 5 accepts them is not explicitly confirmed (only "same Messages API surface as Opus 4.8" → plausible, but DERIVED).
6. **Rate limits / tier assignment** for Fable 5: not documented in the evaluated sources.
7. **Few-shot specifics:** No Fable-5-specific statement on whether/when few-shot examples should be reduced — only the generic 3–5-examples rule plus the finding "old skills too prescriptive."
8. **Tool search, parallel tool calls, programmatic tool calling:** documented only generically for "current models," no Fable-5-specific statements.
9. **Fast mode & 300k batch output:** Fable 5 is absent from both lists — absence documented, explicit negative statement missing.
10. **Mythos 5 prompting differences:** The sources treat Fable 5 and Mythos 5 jointly on the prompting side; whether the unsafeguarded Mythos 5 has different prompting properties is not documented.

---

*Methodological note: core facts (model ID, context window, output limit, pricing, thinking restrictions, effort recommendations) are consistently confirmed across at least four independent official sources. Sole measured discrepancy: tokenizer overhead "~30%" ([Models]) vs. "up to 35%" ([Pricing]) — both figures are compatible (approximation vs. upper bound).*
