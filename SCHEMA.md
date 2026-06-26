---
type: Meta / Specification
title: Wiki Schema & OKF Conformance
description: The format contract for the Claude Capability Library — frontmatter spec, type taxonomy, linking rules, and the "living wiki" maintenance model.
domain: meta
tags: [okf, schema, meta, contract, living-wiki]
related: [index]
timestamp: 2026-06-25T00:00:00Z
okf_version: "0.1"
sources:
  - https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing/
  - https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf
---

# Wiki Schema & OKF Conformance

This library is a **living wiki** that documents *everything the Claude product family can do* — every surface, feature, toggle, command, and setting — organized per the **Open Knowledge Format (OKF) v0.1** (Google Cloud, published 2026-06-13).

OKF formalizes the "LLM-as-librarian" pattern into a portable, vendor-neutral standard: **markdown files with YAML frontmatter in a hierarchical directory structure, cross-linked as a knowledge graph.** The same file is human-readable *and* machine-parseable; the bundle ships as plain files in git; agents maintain it without boredom while humans curate.

This page is the **contract** every other page conforms to.

---

## 1. The three OKF design principles (and how this wiki honors them)

| OKF principle | What it means | How this wiki applies it |
|---|---|---|
| **Minimally opinionated** | Only `type` is strictly required; producers define the rest | Every page has rich-but-optional frontmatter; only `type` + `title` are mandatory |
| **Producer/consumer independence** | The format is the contract, not a platform | Pure markdown + YAML; readable in any editor, GitHub, or LLM context window |
| **Format, not platform** | No proprietary accounts/SDKs/lock-in | Just files. `git clone`, `tar`, or paste into a prompt — no backend required |

---

## 2. Directory layout (hierarchy carries domain meaning)

```
claude-wiki/
├── index.md                     ← root: progressive-disclosure home + full nav
├── SCHEMA.md                    ← this contract (meta)
├── glossary.md                  ← every defined term, alphabetized
├── knowledge-graph.yaml         ← machine-readable nodes + edges (OKF bonus)
├── log.md                       ← chronological change history (living)
├── platform/                    ← the surfaces you interact with Claude through
│   ├── index.md
│   ├── claude-ai.md
│   ├── cowork.md
│   ├── claude-design.md
│   └── claude-code.md
├── models/                      ← the engines
│   ├── index.md
│   ├── model-families.md
│   └── capabilities-and-modes.md
├── claude-code/                 ← deep reference for the coding agent
│   ├── index.md
│   ├── slash-commands.md
│   ├── settings.md
│   ├── hooks.md
│   ├── cli-and-shortcuts.md
│   ├── subagents.md
│   └── dispatch-remote-routines.md
└── capabilities/                ← cross-surface features
    ├── index.md
    ├── artifacts.md
    ├── projects.md
    ├── skills.md
    ├── connectors.md
    ├── mcp.md
    └── plugins.md
```

**Path = identity.** Once a page's path is set, it stays stable (OKF tenet *"immutable file paths as identity"*). Rename the display via the `title` field, not the file. Each directory has an `index.md` for progressive disclosure.

---

## 3. Frontmatter spec

Every content page begins with YAML frontmatter. `type` and `title` are required; the rest are recommended.

```yaml
---
type: <Type Taxonomy value>          # REQUIRED — see §4
title: <Human-readable name>         # REQUIRED
description: <one-sentence summary>   # used by the glossary & index
domain: platform | models | claude-code | capabilities | meta
tags: [searchable, keywords]
related: [page-id, page-id]          # ids of linked pages (the graph edges)
resource: <canonical official URL>   # ground-truth link (docs page)
timestamp: 2026-06-25T00:00:00Z      # last meaningful update
confidence: high | medium | low      # how well-verified the page is
okf_version: "0.1"
sources: [url, url]                  # everything cited
---
```

### Field notes
- **`type`** — the single required OKF field. Pick from the taxonomy in §4.
- **`resource`** — the official doc the page mirrors (e.g. `https://code.claude.com/docs/...`). This is OKF's "resource URI for ground truth."
- **`related`** — page **ids** (filename without `.md`). These are the directed edges of the knowledge graph; `knowledge-graph.yaml` is generated from them.
- **`confidence`** — set by the verification pass. `low` flags a page that needs human curation.

---

## 4. Type taxonomy

A consistent, queryable set of `type` values:

| `type` | Used for |
|---|---|
| `Surface` | A product/app you interact with (Claude.ai, Cowork, Claude Code, Claude Design) |
| `Model Reference` | Model families, IDs, pricing, capabilities |
| `Feature` | A capability that spans surfaces (Artifacts, Projects, Skills) |
| `Command Reference` | Exhaustive command/flag listings (slash commands, CLI) |
| `Configuration Reference` | Settings, hooks, env vars |
| `Protocol` | A standard/spec (MCP) |
| `Extension System` | Plugins, Connectors, Skills as packaging |
| `Index` | A directory landing page |
| `Meta / Specification` | This schema, the glossary |

---

## 5. Cross-linking rules (the knowledge graph)

OKF: *"don't rely solely on file hierarchy — markdown links create the semantic graph."*

- Link with **relative paths** from the linking file's location.
  - Same folder → `[Skills](./skills.md)`
  - Sibling folder → `[Model families](../models/model-families.md)`
  - Up to root → `[Glossary](../glossary.md)`
- Every page ends with a **`## Related pages`** section listing its edges.
- Mirror those edges in the `related:` frontmatter list (as page ids) so the graph stays machine-readable.
- Link **liberally**. A link to a page that doesn't exist yet is a *marker for future work*, not an error (same philosophy as `[[wikilink]]` backlinks).

---

## 6. Page anatomy (the house style)

Each content page follows this shape for scannability:

1. **Orientation** — 2–3 sentences: what this is and why you'd use it.
2. **At-a-glance block** — *What it is / Where you find it / Who can use it (plans) / Status*.
3. **Body** — H2/H3 sections, dense and example-rich. Every feature gets a concrete example: a command, a config snippet, a sample interaction, or a table row.
4. **`## Related pages`** — cross-links.
5. **`## Open questions / to verify`** — anything unconfirmed (so the wiki stays honest).
6. **`## Sources`** — official docs first, then reputable write-ups.

**Honesty rule:** never invent precise values (prices, limits, IDs, exact menu labels) that can't be supported. Mark unconfirmed specifics inline with **⚠️ verify**. Accuracy beats completeness when they conflict.

---

## 7. The "living" maintenance model

What makes this a *living* wiki rather than a static doc dump:

- **Agent-maintained, human-curated.** An LLM agent can be pointed at any page to refresh it against current docs; humans decide what matters. Predictable formatting (this schema) is what makes safe automated updates possible.
- **Version-controlled.** The whole bundle lives in git. `timestamp` + git history = audit trail.
- **`log.md` is the changelog.** Every refresh appends a dated entry.
- **`confidence` decays.** Claude ships fast; a page last verified months ago should have its `confidence` downgraded until re-checked.
- **Refresh recipe (paste to any Claude agent):**
  > "Read `claude-wiki/SCHEMA.md`, then update `<page>` against the latest official docs (docs.claude.com, code.claude.com/docs, platform.claude.com/docs). Correct stale facts, mark unconfirmable specifics with ⚠️ verify, bump `timestamp` and `confidence`, and append a one-line entry to `log.md`."

---

## Related pages
- [Wiki home / index](./index.md)
- [Glossary](./glossary.md)
- [Knowledge graph (machine-readable)](./knowledge-graph.yaml)
- [Change log](./log.md)

## Sources
- [How the Open Knowledge Format can improve data sharing — Google Cloud](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing/)
- [OKF reference implementation — GoogleCloudPlatform/knowledge-catalog](https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf)
