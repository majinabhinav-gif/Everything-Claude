---
type: Index
title: Models — the Claude engines
description: The model families that power every Claude surface, and the capabilities/modes you can turn on across the API, Claude.ai, and Claude Code.
domain: models
tags: [models, opus, sonnet, haiku, fable, mythos, capabilities, index]
related: [model-families, capabilities-and-modes]
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
---

# Models — the Claude engines

Every surface in this library runs on a Claude model. This domain is the authoritative reference for **which models exist** (IDs, context, pricing, availability) and **what they can do** (the capabilities and modes you toggle per request or per surface).

| Page | One-liner |
|---|---|
| [Claude Model Families](./model-families.md) | The full lineup — Fable 5, Mythos 5, Opus 4.8/4.7/4.6, Sonnet 4.6, Haiku 4.5 — with exact model IDs/aliases, context windows, max output, cutoffs, pricing (prompt-cache & batch levers, Priority Tier), and availability across Claude API, the apps, Bedrock, Vertex, and Foundry. |
| [Model Capabilities & Modes](./capabilities-and-modes.md) | The capability layer: extended thinking, reasoning effort, Fast mode, the 1M-token context window, vision, tool use, structured outputs, prompt caching, the Batch API, citations, computer use, streaming, **rate limits & usage tiers**, `service_tier`, the `stop_reason` table, and the beta-header reference. |

## Where capabilities get toggled
The same capability is often turned on differently per surface — the [capabilities page](./capabilities-and-modes.md) gives the toggle path for each:
- **API** — request parameters and beta headers (e.g. `thinking`, `service_tier`, `tools`).
- **Claude.ai** — UI toggles in the [composer](../platform/claude-ai.md#modes--toggles) (web search, extended thinking, Research).
- **Claude Code** — flags and slash commands (see [CLI & shortcuts](../claude-code/cli-and-shortcuts.md) and [slash commands](../claude-code/slash-commands.md)).

## Related pages
- [Platform surfaces](../platform/index.md) · [Claude Code](../claude-code/index.md) · [Capabilities](../capabilities/index.md)
- [Library home](../index.md) · [Glossary](../glossary.md)
