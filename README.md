# superpowers-agy

A Claude Code plugin that hands the implementation tasks of
[superpowers](https://github.com/obra/superpowers) `subagent-driven-development`
to Google's Gemini, through the official
[Antigravity CLI](https://github.com/google-antigravity/antigravity-cli) (`agy`).
Claude keeps the planning, the reviews and the hard tasks; Gemini writes the code
on your Google AI plan's quota.

It does not modify superpowers. It adds a bridge agent, a small CLI wrapper and a
routing skill, so superpowers updates keep working.

## How it works

```
superpowers controller (Claude)
  └─ dispatches superpowers-agy:agy-implementer   (Claude Haiku, bridge only)
        ├─ agy-implement → agy -p … --mode accept-edits   (Gemini writes tests/code)
        ├─ runs the tests itself → RED / GREEN evidence, up to 3 feedback rounds
        ├─ commits
        └─ writes the superpowers implementer report
  └─ task review, re-reviews, final review stay on Claude
```

- **Edit-only Gemini.** agy runs in `--mode accept-edits`. It can create and
  edit files but never runs shell commands. The bridge runs the tests, sends
  failures back, and commits. Nothing runs with `--dangerously-skip-permissions`.
- **Routing.** Mechanical and standard implementer tasks and fix rounds 1-3 go
  to `gemini-3.8-flash-high`. Architecture tasks, fix rounds 4-5 and every
  review stay on Claude. See `skills/agy-routing/SKILL.md`.
- **Fallback.** If agy fails (quota, sign-in, timeout, bad model), the bridge
  reports `AGY_UNAVAILABLE` and the controller re-dispatches the task to a
  Claude implementer.

## Requirements

- Claude Code with the `superpowers` plugin installed.
- `agy` on `PATH`, signed in (`agy` once interactively), with Gemini models
  listed by `agy models`.
- `git`, `jq`, `bash`.

## Install

```bash
claude plugin marketplace add ultimopl/superpowers-agy   # or a local path to a clone
claude plugin install superpowers-agy@superpowers-agy
```

Restart Claude Code. A session-start hook reminds the controller to load the
`superpowers-agy:agy-routing` skill whenever it runs subagent-driven-development.

## Terms of use

The Antigravity Additional Terms forbid using third-party software to access
the service, with "OpenClaw with Antigravity OAuth" as the example. Asked about
this exact setup, a Google moderator on the AI Developers Forum said that
running the official, unmodified `agy` binary locally in headless mode from a
local coordinator is permitted, on these conditions:
[thread](https://discuss.ai.google.dev/t/can-a-local-ai-coordinator-launch-the-official-antigravity-cli-with-subscription-sign-in/186246/2).

- personal, single-user use;
- sequential or low-concurrency runs;
- no reuse of the OAuth tokens.

This plugin keeps to that: it only calls the official binary, never touches
credentials, and superpowers never runs two implementers at once. Read the
[terms](https://antigravity.google/terms) yourself before using it. Also note
that the Antigravity CLI may collect your interactions to improve the product
unless you opt out in its settings, so the code agy sees goes to Google.

## Files

| Path | Purpose |
|---|---|
| `agents/agy-implementer.md` | Bridge agent: drives agy, runs tests, commits, reports |
| `bin/agy-implement` | One edit-only agy turn; exits non-zero when agy reports an error |
| `skills/agy-routing/SKILL.md` | Which superpowers roles go to agy and how to dispatch them |
| `hooks/hooks.json` | Session-start reminder to load the routing skill |

## License

MIT
