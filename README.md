# pi-delegate

A [pi](https://pi.dev) extension that delegates bounded coding tasks to an isolated, non-interactive `pi` subprocess — with **live TUI progress**.

```
⏳ Delegate · implement
3 turns · ↑12k ↓1.2k $0.0043
→ bash rg -n auth src/
→ read src/auth.ts:1-40
→ edit src/auth.ts
```

The parent agent keeps its context window clean while a cheaper model does the work in a separate process. Progress (status, turns taken, token/cost consumption, recent tool calls) streams into the parent's TUI as it happens.

## Roles

| Role | Tools | Thinking | Purpose |
|------|-------|----------|---------|
| `scout` | read-only + web | low | Recon, returns compressed findings with paths/lines |
| `implement` | read/write + web | medium | Smallest correct change, focused checks |
| `review` | read-only + web | medium | Actionable findings on the working diff |
| `chore` | read/write + web | low | Bounded repository chore |

Overrides: `provider`, `model`, `thinking`, `timeoutSeconds`, `maxTurns`, `maxCostUsd`, `cwd`. Limit events (timeout/turns/cost) preserve completed filesystem work and return partial progress for reassignment.

## Requirements

This extension spawns a child `pi` process with two other extensions loaded for web access and lazy-coding guidance. Install both first:

```bash
pi install npm:pi-web-access
pi install npm:@dietrichgebert/ponytail
```

If either is missing, a delegate call fails with an error naming the package to install.

## Install

```bash
pi install git:github.com/theMladyPan/pi-delegate
```

Or pin a ref:

```bash
pi install git:github.com/theMladyPan/pi-delegate@v0.1.0
```

To try without installing:

```bash
pi -e git:github.com/theMladyPan/pi-delegate
```

## How it works

- Spawns `pi --mode json -p --no-session` with a role-scoped system prompt and tool set.
- Parses the child's JSON event stream (`message_end`, `turn_end`, `tool_execution_start/end`).
- Streams `onUpdate` to the parent TUI (60ms coalesced) so you see live turns, usage, and tool calls.
- `renderCall` shows the role + limits + task preview; `renderResult` shows a compact live view (partial) and a full summary (expanded) with files changed, all tools, and the output as Markdown.

The child's model-visible output is returned to the parent; on failure the parent gets diagnostics.

## License

MIT
