---
type: Index
title: The Claude Capability Library
description: A living, OKF-conformant wiki documenting everything the Claude product family can do — every surface, feature, toggle, command, and setting, example-rich and cross-linked.
domain: meta
tags: [home, index, claude, okf, living-wiki, reference]
related: [claude-ai, claude-code-overview, model-families, skills, mcp]
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing/
  - https://docs.claude.com
  - https://code.claude.com/docs
  - https://platform.claude.com/docs
---

# The Claude Capability Library

> A deep, example-rich reference to **everything Claude can do** — every product surface, model, feature, toggle, slash command, and setting — built as a **living wiki** that conforms to the **[Open Knowledge Format (OKF) v0.1](./SCHEMA.md)**.

This is not a single document. It is a small graph of plain-markdown pages with YAML frontmatter, cross-linked so an agent (or a human) can walk from any concept to its neighbors, refresh any page against current docs, and keep the whole thing honest as Claude ships new features.

**How to read it:** start at the domain you care about, or jump straight to a page below. Every page follows the same shape — orientation → at-a-glance → dense example-rich body → *Related pages* / *Open questions* / *Sources*. Unconfirmed specifics are flagged inline with **⚠️ verify** (rendered `WARN: verify` in some pages) so you always know what's solid and what needs a re-check.

---

## The four domains

| Domain | What lives here | Index |
|---|---|---|
| 🖥️ **Platform** | The surfaces you interact with Claude through | [platform/](./platform/index.md) |
| 🧠 **Models** | The engines: families, IDs, pricing, capabilities | [models/](./models/index.md) |
| ⌨️ **Claude Code** | Deep reference for the agentic coding tool | [claude-code/](./claude-code/index.md) |
| 🧩 **Capabilities** | Cross-surface features & extension systems | [capabilities/](./capabilities/index.md) |

---

## Every page at a glance

### 🖥️ Platform — *the surfaces*
| Page | What it covers |
|---|---|
| [Claude.ai & the Claude Apps](./platform/claude-ai.md) | Web/desktop/mobile/Chrome apps; chat workspace, sidebar nav, modes & toggles (web search, extended thinking, Research, Styles, Voice, Agent Mode), Memory, Artifacts entry, full Settings tree, plans (Free→Enterprise), usage limits, shortcuts. |
| [Claude Cowork](./platform/cowork.md) | The agentic *Tasks* mode in Claude Desktop that reads/edits/creates **real local files**; approval model, scheduled tasks, connectors, recovery, and enterprise controls (RBAC, OpenTelemetry). |
| [Claude Design](./platform/claude-design.md) | Anthropic Labs' **prompt-to-prototype** visual workspace (designs, prototypes, slides, one-pagers); canvas editing, toggles, version management, and design→code handoff to Claude Code. |
| [Claude Code — Overview](./platform/claude-code.md) | Navigational map of the terminal-native coding agent: install, surfaces, auth, the agent loop, tools, permission modes, `CLAUDE.md` memory — and links into every deep page. |

### 🧠 Models — *the engines*
| Page | What it covers |
|---|---|
| [Claude Model Families](./models/model-families.md) | Every current model — Fable 5, Mythos 5, Opus 4.8/4.7/4.6, Sonnet 4.6, Haiku 4.5 — with exact IDs/aliases, context windows, max output, cutoffs, pricing (cache/batch levers), Priority Tier, and availability across API/apps/Bedrock/Vertex/Foundry. |
| [Model Capabilities & Modes](./models/capabilities-and-modes.md) | What the models *do* and how each is toggled across API/Claude.ai/Claude Code: extended thinking, reasoning effort, Fast mode, 1M context, vision, tool use, structured outputs, caching, Batch API, rate limits, `service_tier`, `stop_reason`, beta headers. |

### ⌨️ Claude Code — *the coding agent, in depth*
| Page | What it covers |
|---|---|
| [Slash Commands](./claude-code/slash-commands.md) | Every built-in command grouped by purpose (with version gates) **and** custom commands/skills: frontmatter, `$ARGUMENTS`, `!` bash injection, `@` file refs, the Skill tool. |
| [Settings](./claude-code/settings.md) | `settings.json` full reference: five-layer precedence, every key, the deny→ask→allow permissions model, `Tool(specifier)` syntax, sandbox, status line, env vars, annotated example. |
| [Hooks](./claude-code/hooks.md) | The 30-event hook catalog, matchers, stdin/stdout contracts, exit-code semantics, structured JSON output, 10 worked examples, and security warnings. |
| [CLI Flags & Keyboard Shortcuts](./claude-code/cli-and-shortcuts.md) | The `claude` binary (interactive/print/headless), the full flag table, subcommands, the keyboard-shortcut chord table, vim mode, **git worktrees**, and `keybindings.json`. |
| [Subagents](./claude-code/subagents.md) | The Agent tool (formerly Task), built-in agent types, custom `.claude/agents`, fan-out, forks, `isolation: worktree`, per-agent tools/model/permissions. |
| [Dispatch, Remote Control & Routines](./claude-code/dispatch-remote-routines.md) | The full remote/cloud surface: **Dispatch**, **Remote Control**, **Routines** (cron/cloud), **Claude Code on the web**, **Channels**, and **GitHub Actions** — with a "where does it run" decision table. |

### 🧩 Capabilities — *cross-surface features & extensions*
| Page | What it covers |
|---|---|
| [Artifacts](./capabilities/artifacts.md) | Standalone editable content (code, docs, HTML, SVG, Mermaid, React apps, dashboards); creation, editing, versions, publish/share/remix, Claude-powered apps, and enterprise live dashboards. |
| [Projects](./capabilities/projects.md) | Workspaces that bundle chats around shared context: knowledge base, project instructions, per-project memory, RAG, sharing, and artifacts-in-projects. |
| [Agent Skills](./capabilities/skills.md) | Folder-based `SKILL.md` packages with **progressive disclosure**; the three surfaces (apps/Code/API), install & management, and built-in document skills. |
| [Connectors](./capabilities/connectors.md) | The OAuth integration catalog (Drive, Gmail, Slack, Linear, …), custom remote-MCP connectors, Desktop Extensions (MCPB), and admin governance. |
| [Model Context Protocol (MCP)](./capabilities/mcp.md) | The open standard underneath connectors: architecture, transports, primitives, Claude Code config (`claude mcp add`, scopes, `.mcp.json`), OAuth, and security. |
| [Plugins](./capabilities/plugins.md) | Shareable bundles of commands/subagents/hooks/MCP/skills; the manifest, marketplaces, `/plugin`, install scopes, and enterprise controls. |

---

## How this library is organized (the OKF model)

This wiki follows the **[Open Knowledge Format](./SCHEMA.md)** — Google Cloud's open spec (v0.1, June 2026) for representing curated knowledge that both humans and AI agents can read and maintain:

- **Just markdown + YAML frontmatter.** No backend, no proprietary format. `git clone` it, `tar` it, or paste a page into a prompt.
- **Hierarchical domains** carry meaning; each directory has an `index.md` for progressive disclosure.
- **Cross-links are the knowledge graph.** Every page lists its edges in `related:` and in a *Related pages* section. The machine-readable graph is materialized in **[knowledge-graph.yaml](./knowledge-graph.yaml)**.
- **One required field (`type`)**, a small [type taxonomy](./SCHEMA.md#4-type-taxonomy), and an [honesty rule](./SCHEMA.md#6-page-anatomy-the-house-style): never invent precise values — flag them ⚠️.
- **Living, not static.** Pages carry a `confidence` and `timestamp`; **[log.md](./log.md)** is the changelog; the [refresh recipe](./SCHEMA.md#7-the-living-maintenance-model) tells any agent how to update a page safely.

### Meta files
- **[SCHEMA.md](./SCHEMA.md)** — the format contract (frontmatter spec, type taxonomy, linking & maintenance rules).
- **[glossary.md](./glossary.md)** — every term defined across the library, alphabetized and linked to its source page.
- **[knowledge-graph.yaml](./knowledge-graph.yaml)** — machine-readable nodes + edges (the OKF bonus artifact).
- **[log.md](./log.md)** — chronological change history, audit scores, and the tracked-gaps backlog.

---

## Keeping it alive

To refresh any page against the latest official docs, hand this to a Claude agent:

> "Read `claude-wiki/SCHEMA.md`, then update `<page>` against the latest official docs (docs.claude.com, code.claude.com/docs, platform.claude.com/docs). Correct stale facts, mark unconfirmable specifics ⚠️, bump `timestamp` and `confidence`, and append a one-line entry to `log.md`."

To rebuild the glossary and knowledge graph after editing pages, re-run the extractor (`node extract.js`) described in [log.md](./log.md).

---

## Library stats (2026-06-26)

- **18 content pages** across 4 domains + 4 meta files (this index, SCHEMA, glossary, knowledge-graph, log)
- **~9,000+ lines** of example-rich documentation
- **~218 glossary terms**
- Built by a multi-agent OKF workflow: **draft → adversarial verify → completeness audit → targeted enrichment**
- Verified against `docs.claude.com`, `code.claude.com/docs`, `platform.claude.com/docs`, `support.claude.com`, `anthropic.com`, `claude.com`

## Sources
- [Open Knowledge Format — Google Cloud](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing/)
- [Claude Docs](https://docs.claude.com) · [Claude Code Docs](https://code.claude.com/docs) · [Claude Platform Docs](https://platform.claude.com/docs)
