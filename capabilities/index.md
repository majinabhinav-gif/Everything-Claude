---
type: Index
title: Capabilities — cross-surface features & extensions
description: Features and extension systems that span multiple Claude surfaces — Artifacts, Projects, Skills, Connectors, MCP, and Plugins.
domain: capabilities
tags: [capabilities, artifacts, projects, skills, connectors, mcp, plugins, index]
related: [artifacts, projects, skills, connectors, mcp, plugins]
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
---

# Capabilities — cross-surface features & extensions

These are the features and **extension systems** that aren't tied to a single surface — they show up in [Claude.ai](../platform/claude-ai.md), [Cowork](../platform/cowork.md), and/or [Claude Code](../platform/claude-code.md), and they're how you teach Claude new tricks and connect it to your data.

| Page | One-liner |
|---|---|
| [Artifacts](./artifacts.md) | Standalone, editable content in a side panel — code, docs, HTML, SVG, Mermaid, React apps, dashboards — with versions, publish/share/remix, Claude-powered apps, and enterprise live dashboards. |
| [Projects](./projects.md) | Workspaces that bundle chats around a shared knowledge base, project instructions, and per-project memory (with RAG and Team/Enterprise sharing). |
| [Agent Skills](./skills.md) | Folder-based `SKILL.md` packages loaded via **progressive disclosure**; run on the apps, Claude Code, and the API; install/manage; built-in document skills. |
| [Connectors](./connectors.md) | The user-facing **OAuth catalog** (Drive, Gmail, Slack, Linear, GitHub, …), custom remote-MCP connectors, Desktop Extensions (MCPB), and admin governance. |
| [Model Context Protocol (MCP)](./mcp.md) | The open standard *underneath* connectors: host/client/server architecture, transports, primitives (tools/resources/prompts), Claude Code config, OAuth, and security. |
| [Plugins](./plugins.md) | Shareable bundles that add slash commands, subagents, hooks, MCP servers, and skills to Claude Code at once — with a manifest, marketplaces, and enterprise controls. |

## How the extension systems relate
A quick disambiguation (each page has a fuller cross-walk):
- **[Skills](./skills.md)** teach *procedural know-how* (a `SKILL.md` + scripts).
- **[Connectors](./connectors.md)** provide *live data & actions* in external apps — and are built on **[MCP](./mcp.md)**, the protocol.
- **[Plugins](./plugins.md)** *package* commands, [subagents](../claude-code/subagents.md), [hooks](../claude-code/hooks.md), MCP servers, and skills together for Claude Code.
- **[Projects](./projects.md)** bundle *context* (knowledge + instructions) around a set of chats; **[Artifacts](./artifacts.md)** are the *outputs* you build inside any of them.

## Related pages
- [Platform surfaces](../platform/index.md) · [Models](../models/index.md) · [Claude Code](../claude-code/index.md)
- [Library home](../index.md) · [Glossary](../glossary.md)
