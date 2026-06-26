---
type: Index
title: Claude Code — deep reference
description: The complete reference for Claude Code, Anthropic's terminal-native agentic coding tool — every slash command, setting, hook, CLI flag, subagent, and the remote/cloud execution surface.
domain: claude-code
tags: [claude-code, cli, slash-commands, settings, hooks, subagents, dispatch, index]
related: [claude-code-overview, slash-commands, settings, hooks, cli-shortcuts, subagents, dispatch-remote]
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
---

# Claude Code — deep reference

Claude Code is Anthropic's terminal-native agentic coding tool. The high-level map (install, surfaces, auth, the agent loop, tools, permission modes, `CLAUDE.md` memory) lives on the [overview page](../platform/claude-code.md); **this domain holds the exhaustive reference** for everything you configure and drive.

| Page | One-liner |
|---|---|
| [Slash Commands](./slash-commands.md) | Every built-in command (grouped by purpose, with version gates) plus custom commands/skills — frontmatter, `$ARGUMENTS`/`$N`, `!` bash injection, `@` file refs, and the model-invoked Skill tool. |
| [Settings](./settings.md) | `settings.json` full reference: the five-layer precedence stack, the `.claude/` directory, every key, the deny→ask→allow permissions model and `Tool(specifier)` syntax, sandbox, status line, and the env-var catalog. |
| [Hooks](./hooks.md) | The lifecycle hook system: the 30-event catalog, matchers, per-event stdin JSON, exit-code semantics (`0`/`2`/other), structured JSON output, 10 worked examples, and security. |
| [CLI Flags & Keyboard Shortcuts](./cli-and-shortcuts.md) | The `claude` binary — interactive vs `--print` headless, the full flag table, subcommands — plus the keyboard chord table, vim mode, **git worktrees**, and custom `keybindings.json`. |
| [Subagents](./subagents.md) | Context-isolated child agents: the Agent tool (formerly Task), built-in types, custom `.claude/agents`, the `/agents` manager, fan-out, forks, and `isolation: worktree`. |
| [Dispatch, Remote Control & Routines](./dispatch-remote-routines.md) | The remote/cloud execution surface — Dispatch, Remote Control, Routines (cron/cloud), Claude Code on the web, Channels, and GitHub Actions — with a "where does it run" decision table. |

## Reading order
1. New to Claude Code? Start with the [overview](../platform/claude-code.md).
2. Configuring a project? [Settings](./settings.md) → [Hooks](./hooks.md) → [Slash commands](./slash-commands.md).
3. Scaling work? [Subagents](./subagents.md) → [Dispatch/Remote/Routines](./dispatch-remote-routines.md).
4. Extending it? Cross to [Plugins](../capabilities/plugins.md), [Skills](../capabilities/skills.md), and [MCP](../capabilities/mcp.md).

## Related pages
- [Claude Code overview](../platform/claude-code.md) · [Plugins](../capabilities/plugins.md) · [MCP](../capabilities/mcp.md) · [Skills](../capabilities/skills.md)
- [Library home](../index.md) · [Glossary](../glossary.md)
