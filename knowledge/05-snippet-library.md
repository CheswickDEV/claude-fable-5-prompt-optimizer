# Snippet Library

Paste-ready text blocks for Phase 3, step 4 (see `03-workflow-and-routing.md`). Each snippet names its trigger (route + signal) and the rule it implements. Where the snippet quotes Anthropic's documented wording, the quote is carried verbatim; surrounding text follows the documented pattern. Translate snippets into the input prompt's language when the prompt is not in English; keep the meaning intact.

Insert snippets selectively — only what the route and the prompt's gaps call for. Stacking all of them would itself violate R3/R39 (over-prescription).

## 1. Intent / motivation block — R2

**When:** the prompt states a task but not its purpose, audience, or what the output enables. All routes.

```text
I'm working on [the larger task] for [who it's for]. They need
[what the output enables]. With that in mind: [request].
```

## 2. Anti-overplanning — R11

**When:** ambiguous or open-ended tasks where the model might survey options instead of acting. Routes M/L/XL.

```text
When you have enough information to act, act. Do not re-derive facts
already established, re-litigate decisions already made, or narrate
options you will not pursue. If you are weighing a choice, give a
recommendation, not an exhaustive survey. (This does not apply to
thinking blocks.)
```

## 3. Brevity — R12

**When:** any prompt without explicit verbosity steering, especially at effort high+. Routes S/M/L/XL.

```text
Lead with the outcome. Keep the response short by being selective
about what you include, not by compressing the writing — complete
sentences, no fragments or jargon. Readability over brevity.
```

## 4. Communication-style addendum (agentic) — R13

**When:** agentic sessions with tool use whose final answer a human reads. Routes L/XL.

```text
Shorthand between tool calls is fine. The final summary is for a
reader who watched none of the work: complete sentences, spell out
terms, no labels invented mid-session. If you have to choose between
short and clear, choose clear.
```

## 5. Anti-overengineering — R14

**When:** coding tasks, especially at effort high/xhigh. Routes L/XL.

```text
Build only what the task requires. No features, refactorings, or
abstractions beyond it; do not design for hypothetical requirements.
Validate at system boundaries only. Change code directly instead of
adding feature flags or compatibility shims.
```

## 6. Explicit boundaries / assessment-first — R17

**When:** the prompt gives the model access to state it could change (repos, files, accounts) or describes a problem. Routes L/XL.

```text
When the user describes a problem, the deliverable is the assessment —
do not apply a fix until they ask for one. Before any state-changing
action, check that the evidence supports that specific action. Do not
take actions beyond what was requested.
```

## 7. Checkpoint rule — R18

**When:** interactive agentic work where the model might either over-ask or never ask. Routes L/XL.

```text
Pause only when the work genuinely needs the user: destructive or
irreversible actions, a scope change, or input only the user can
provide. Then ask and end the turn — do not end a turn with a promise
of work not yet done.
```

## 8. Autonomy reminder (pipelines) — R19

**When:** unattended pipelines/headless runs where a question would block the work. Route XL (and unattended L).

```text
You are operating autonomously; no one is available to answer
questions. "Want me to…?" will block the work. For reversible actions
that follow from the request, proceed. Before ending your turn, check
your last paragraph — if it is a plan, a question, or a promise, do
that work now with tool calls.
```

## 9. Progress audit — R20

**When:** long-running agents that report status. Routes L/XL.

```text
Before reporting progress, audit each claim against a tool result from
this session. Report only what a tool result confirms; if a step was
skipped or failed, say so plainly.
```

## 10. Subagent / verifier guidance — R21

**When:** the harness offers subagents. Route XL (and L with subagent tooling).

```text
Use subagents freely for parallelizable work and keep long-lived
subagents alive rather than respawning them. Verification: dispatch a
separate verifier subagent with fresh context (it beats self-review);
verify after every [N] completed units of work.
```

## 11. Memory instructions — R22

**When:** the harness provides persistent file storage across sessions. Route XL.

```text
Maintain a memory directory of markdown files: one lesson per file,
one-line summary at the top. Record corrections and confirmed
approaches with the why. Do not store what the repo or history already
records; delete notes that turn out wrong. Check memory at the start
of every session before acting.
```

## 12. Context reassurance — R23

**When:** only if the harness unavoidably displays remaining-context/token information. Routes L/XL.

```text
You have ample context remaining. Do not stop, summarize, or suggest a
new session on account of context limits.
```

## 13. Agent-to-agent guardrails — R43

**When:** multi-agent setups where agents can message other agents (including third-party ones). Route XL.

```text
You may communicate with other agents only to [allowed purposes, e.g.,
coordinate task handoffs and exchange task-relevant data]. Do not
propose, accept, or coordinate pricing, market division, or any
agreement with competing agents. When unsure whether a coordination is
permitted, do not engage and report it instead.
```

## 14. Refusal-domain advisory — R34

**When:** the prompt touches cybersecurity or biology/life sciences (text or images). Goes into `<open_questions>`, not the prompt itself.

```text
This task touches the [cyber|bio] domain. Even benign requests here
can trigger Fable 5's safety classifiers (refusal category "[cyber|
bio]"). API: configure fallbacks to claude-opus-4-8 (beta header
server-side-fallback-2026-06-01). claude.ai: requests may be answered
by Opus 4.8 automatically.
```

## 15. Thinking-dial phrases — R10

**When:** the task needs noticeably more or less deliberation than the default; replaces any "think step by step" found in the input. Wording is sensitivity-tested — measure if quality matters.

```text
Encouraging: "Please think hard before responding."
Dampening:   "Think only when it will meaningfully improve answer
              quality."
```
