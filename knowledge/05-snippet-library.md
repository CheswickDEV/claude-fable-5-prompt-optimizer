# Snippet Library

Paste-ready text blocks for Phase 3, step 4 (see `03-workflow-and-routing.md`). Each snippet names its trigger (route + signal) and the rule it implements. Where the snippet quotes Anthropic's documented wording, the quote is carried verbatim; surrounding text follows the documented pattern. Wordings synced to the live [PromptF5] page on **2026-07-14** — several documented snippets were expanded by Anthropic since the June 9 snapshot. Translate snippets into the input prompt's language when the prompt is not in English; keep the meaning intact.

Insert snippets selectively — only what the route and the prompt's gaps call for. Stacking all of them would itself violate R3/R39 (over-prescription). The longer documented texts (snippets 4, 5, 8) may be trimmed to the sentences the prompt actually needs.

## Contents

1. Intent/motivation (R2) · 2. Anti-overplanning (R11) · 3. Brevity (R12) ·
4. Communication style (R13) · 5. Anti-overengineering (R14) · 6. Boundaries (R17) ·
7. Checkpoint (R18) · 8. Autonomy reminder (R19) · 9. Progress audit (R20) ·
10. Subagents/verifier (R21) · 11. Memory (R22) · 12. Context reassurance (R23) ·
13. Agent-to-agent guardrails (R43) · 14. Refusal-domain advisory (R34) ·
15. Thinking-dial phrases (R10) · 16. Send-to-user elicitation (R24)

## 1. Intent / motivation block — R2

**When:** the prompt states a task but not its purpose, audience, or what the output enables. All routes.

```text
I'm working on [the larger task] for [who it's for]. They need
[what the output enables]. With that in mind: [request].
```

## 2. Anti-overplanning — R11 *(wording updated 2026-07-14)*

**When:** ambiguous or open-ended tasks where the model might survey options instead of acting. Routes M/L/XL.

```text
When you have enough information to act, act. Do not re-derive facts
already established in the conversation, re-litigate a decision the
user has already made, or narrate options you will not pursue in
user-facing messages. If you are weighing a choice, give a
recommendation, not an exhaustive survey. This does not apply to
thinking blocks.
```

## 3. Brevity — R12 *(wording updated 2026-07-14)*

**When:** any prompt without explicit verbosity steering, especially at effort high+. Routes S/M/L/XL.

```text
Lead with the outcome. Your first sentence after finishing should
answer "what happened" or "what did you find": the thing the user
would ask for if they said "just give me the TLDR." Supporting detail
and reasoning come after. Being readable and being concise are
different things, and readability matters more.

The way to keep output short is to be selective about what you include
(drop details that don't change what the reader would do next), not to
compress the writing into fragments, abbreviations, arrow chains like
A → B → fails, or jargon.
```

## 4. Communication-style addendum (agentic) — R13 *(wording updated 2026-07-14)*

**When:** agentic sessions with tool use whose final answer a human reads. Routes L/XL. Trim to the paragraphs the deployment needs.

```text
Terse shorthand is fine between tool calls (that's you thinking out
loud, and brevity there is good). Your final summary is different:
it's for a reader who didn't see any of that.

If you've been working for a while without the user watching
(overnight, across many tool calls, since they last spoke), your final
message is their first look at any of it. Write it as a re-grounding,
not a continuation of your working thread: the outcome first, then the
one or two things you need from them, each explained as if new. The
vocabulary you built up while working is yours, not theirs; leave it
behind unless you re-introduce it.

When you write the summary at the end, drop the working shorthand.
Write complete sentences. Spell out terms. Don't use arrow chains,
hyphen-stacked compounds, or labels you made up earlier. When you
mention files, commits, flags, or other identifiers, give each one its
own plain-language clause. Open with the outcome: one sentence on what
happened or what you found. Then the supporting detail. If you have to
choose between short and clear, choose clear.
```

## 5. Anti-overengineering — R14 *(wording updated 2026-07-14)*

**When:** coding tasks, especially at effort high/xhigh. Routes L/XL.

```text
Don't add features, refactor, or introduce abstractions beyond what
the task requires. A bug fix doesn't need surrounding cleanup and a
one-shot operation usually doesn't need a helper. Don't design for
hypothetical future requirements: do the simplest thing that works
well. Avoid premature abstraction and half-finished implementations.
Don't add error handling, fallbacks, or validation for scenarios that
cannot happen. Trust internal code and framework guarantees. Only
validate at system boundaries (user input, external APIs). Don't use
feature flags or backwards-compatibility shims when you can just
change the code.
```

## 6. Explicit boundaries / assessment-first — R17 *(wording updated 2026-07-14)*

**When:** the prompt gives the model access to state it could change (repos, files, accounts) or describes a problem. Routes L/XL.

```text
When the user is describing a problem, asking a question, or thinking
out loud rather than requesting a change, the deliverable is your
assessment. Report your findings and stop. Don't apply a fix until
they ask for one. Before running a command that changes system state
(restarts, deletes, config edits), check that the evidence actually
supports that specific action. A signal that pattern-matches to a
known failure may have a different cause.
```

## 7. Checkpoint rule — R18 *(wording updated 2026-07-14)*

**When:** interactive agentic work where the model might either over-ask or never ask. Routes L/XL.

```text
Pause for the user only when the work genuinely requires them: a
destructive or irreversible action, a real scope change, or input that
only they can provide. If you hit one of these, ask and end the turn,
rather than ending on a promise.
```

## 8. Autonomy reminder (pipelines) — R19 *(wording updated 2026-07-14)*

**When:** unattended pipelines/headless runs where a question would block the work. Route XL (and unattended L).

```text
You are operating autonomously. The user is not watching in real time
and cannot answer questions mid-task, so asking "Want me to…?" or
"Shall I…?" will block the work. For reversible actions that follow
from the original request, proceed without asking. Offering follow-ups
after the task is done is fine; asking permission after already
discussing with the user before doing the work is not. Before ending
your turn, check your last paragraph. If it is a plan, an analysis, a
question, a list of next steps, or a promise about work you have not
done ("I'll…", "let me know when…"), do that work now with tool calls.
End your turn only when the task is complete or you are blocked on
input only the user can provide.
```

## 9. Progress audit — R20 *(wording updated 2026-07-14)*

**When:** long-running agents that report status. Routes L/XL.

```text
Before reporting progress, audit each claim against a tool result from
this session. Only report work you can point to evidence for; if
something is not yet verified, say so explicitly. Report outcomes
faithfully: if tests fail, say so with the output; if a step was
skipped, say that; when something is done and verified, state it
plainly without hedging.
```

## 10. Subagent / verifier guidance — R21 *(wording updated 2026-07-14)*

**When:** the harness offers subagents. Route XL (and L with subagent tooling).

```text
Delegate independent subtasks to subagents and keep working while they
run. Intervene if a subagent goes off track or is missing relevant
context.

Establish a method for checking your own work at an interval of [X] as
you build. Run this every [X interval], verifying your work with
subagents against the specification.
```

## 11. Memory instructions — R22 *(wording updated 2026-07-14)*

**When:** the harness provides persistent file storage across sessions. Route XL.

```text
Store one lesson per file with a one-line summary at the top. Record
corrections and confirmed approaches alike, including why they
mattered. Don't save what the repo or chat history already records;
update an existing note rather than creating a duplicate; delete notes
that turn out to be wrong. Check memory at the start of every session
before acting.
```

*(The final sentence implements the System-Card finding behind R22 — explicitly instruct a session-start memory check — and is not part of the [PromptF5] quote.)*

## 12. Context reassurance — R23 *(wording updated 2026-07-14)*

**When:** only if the harness unavoidably displays remaining-context/token information. Routes L/XL.

```text
You have ample context remaining. Do not stop, summarize, or suggest a
new session on account of context limits. Continue the work.
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

## 16. Send-to-user elicitation — R24 *(added 2026-07-14)*

**When:** long asynchronous agents whose harness defines a `send_to_user` tool. The docs note the tool is rarely called without an explicit instruction. Route XL.

```text
Between tool calls, when you have content the user must read verbatim
(a partial deliverable, a direct answer to their question), call the
send_to_user tool with that content. Use send_to_user only for
user-facing content, not for narration or reasoning.
```
