---
name: agy-routing
description: Use when following superpowers subagent-driven-development (or any plan execution that dispatches implementer subagents) and the superpowers-agy plugin is enabled — routes implementer tasks to Google's Gemini via the agy-implementer agent while reviews stay on Claude.
---

# Routing implementers to Antigravity (Gemini)

The superpowers process stays exactly as written. Only the implementer
dispatch changes: code is written by Gemini through the official Antigravity
CLI (`agy`), driven by the `superpowers-agy:agy-implementer` bridge agent.

| Role | Dispatch |
|---|---|
| Mechanical implementation (complete spec, 1-2 files) | `subagent_type: superpowers-agy:agy-implementer`, `AGY_MODEL: gemini-3.8-flash-high` |
| Standard / integration implementation | `subagent_type: superpowers-agy:agy-implementer`, `AGY_MODEL: gemini-3.8-flash-high` |
| Fix rounds 1-3 | fresh `agy-implementer` with the findings, the prior `AGY_CONVERSATION`, `AGY_MODEL: gemini-3.8-flash-high` |
| Architecture / design-judgment implementation, fix rounds 4-5 | Claude implementer per the skill's Model Selection |
| All reviews (per-task, re-review, final) | Claude, per the skill |

## Dispatching

- Use the superpowers implementer prompt template as the dispatch prompt and
  add one line: `AGY_MODEL: <model>`. For fix rounds also add
  `AGY_CONVERSATION: <id from the previous report>` so agy keeps its context.
- Do not set `model` on the dispatch; the agent pins its own cheap bridge
  model. Gemini does the writing.
- Name a temp prompt directory inside the plan workspace (for example
  `<workspace>/agy-prompts/`) so prompts stay out of the repo.
- agy's work is reviewed like any implementer's. Never skip the task review.

## One agy run at a time

Never dispatch agy-implementer agents in parallel. superpowers already
forbids parallel implementers; for agy it also matters for the account:
Google's guidance for subscription sign-in is single-user, sequential or
low-concurrency use of the unmodified binary.

## Fallback

If the agent reports `AGY_UNAVAILABLE: <reason>` (quota, sign-in, timeout,
unknown model), re-dispatch that task to a Claude implementer chosen per
Model Selection, continue the plan, and tell your human partner about the
fallback.

## Models

`agy models` lists what the account offers. Only Google models belong here
(Gemini). If `gemini-3.8-flash-high` is not listed, pick the highest Gemini
tier that is, and say so in the ledger.
