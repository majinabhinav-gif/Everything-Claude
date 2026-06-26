# Everything Claude — a living capability wiki

A deep, example-rich reference to **everything the Claude product family can do** — every surface, model, feature, toggle, slash command, and setting — built as a **living wiki** that conforms to the **[Open Knowledge Format (OKF) v0.1](SCHEMA.md)** (plain markdown + YAML frontmatter, hierarchical domains, cross-linked as a knowledge graph).

> 📖 **Start here → [index.md](index.md)** (the full home page and navigation).

---

## What's inside

| Domain | What lives here |
|---|---|
| 🖥️ **[Platform](platform/index.md)** | The surfaces: [Claude.ai & apps](platform/claude-ai.md) · [Cowork](platform/cowork.md) · [Claude Design](platform/claude-design.md) · [Claude Code overview](platform/claude-code.md) |
| 🧠 **[Models](models/index.md)** | [Model families](models/model-families.md) (Fable/Mythos/Opus/Sonnet/Haiku, IDs, pricing) · [Capabilities & modes](models/capabilities-and-modes.md) |
| ⌨️ **[Claude Code](claude-code/index.md)** | [Slash commands](claude-code/slash-commands.md) · [Settings](claude-code/settings.md) · [Hooks](claude-code/hooks.md) · [CLI & shortcuts](claude-code/cli-and-shortcuts.md) · [Subagents](claude-code/subagents.md) · [Dispatch/Remote/Routines](claude-code/dispatch-remote-routines.md) |
| 🧩 **[Capabilities](capabilities/index.md)** | [Artifacts](capabilities/artifacts.md) · [Projects](capabilities/projects.md) · [Skills](capabilities/skills.md) · [Connectors](capabilities/connectors.md) · [MCP](capabilities/mcp.md) · [Plugins](capabilities/plugins.md) |
| 📺 **[Learning resources](resources/index.md)** | Curated external videos & courses about Claude — with creators and **YouTube publish dates**, built to be swap-automatable |

**Meta:** [SCHEMA.md](SCHEMA.md) (the format contract) · [glossary.md](glossary.md) (~218 terms) · [knowledge-graph.yaml](knowledge-graph.yaml) (machine-readable nodes + edges) · [log.md](log.md) (changelog + backlog).

---

## How it's organized (the OKF model)

- **Just markdown + YAML.** No backend, no lock-in. `git clone` it, `tar` it, or paste a page into a prompt.
- **Cross-links are the knowledge graph.** Every page lists its edges in `related:` and a *Related pages* section; the machine-readable graph lives in [knowledge-graph.yaml](knowledge-graph.yaml).
- **One required field (`type`)**, a small type taxonomy, and an **honesty rule**: never invent precise values — unverified specifics are flagged inline with ⚠️ / `WARN: verify`.
- **Living, not static.** Pages carry a `confidence` and `timestamp`; [log.md](log.md) is the changelog; [SCHEMA.md §7](SCHEMA.md) has the refresh recipe any agent can follow.

This wiki **is** a graph — open the folder in [Obsidian](https://obsidian.md) to explore it visually via the graph view (the included `.obsidian/` config enables it).

---

## Keeping it alive

Paste this to any Claude agent to refresh a page:

> "Read `SCHEMA.md`, then update `<page>` against the latest official docs (docs.claude.com, code.claude.com/docs, platform.claude.com/docs). Correct stale facts, mark unconfirmable specifics ⚠️, bump `timestamp` and `confidence`, and append a one-line entry to `log.md`."

The [Learning Resources](resources/index.md) list is backed by [`resources/claude-video-resources.yaml`](resources/claude-video-resources.yaml) so outdated YouTube tutorials can be detected (by publish date) and swapped automatically.

---

## Stats

- 18 reference pages across 4 domains + a curated learning-resources index (15 videos/courses) + 5 meta files
- ~10,600 lines · ~218 glossary terms · cross-linked and link-checked
- Built and verified against `docs.claude.com`, `code.claude.com/docs`, `platform.claude.com/docs`, `support.claude.com`, `anthropic.com`, `claude.com`

## License & contributions

This is a community knowledge base compiled from public Anthropic/Claude documentation. Corrections and additions welcome — keep entries conformant to [SCHEMA.md](SCHEMA.md).
