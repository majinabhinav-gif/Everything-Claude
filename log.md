---
type: Meta / Specification
title: Change Log
description: Chronological history of the Claude Capability Library — what was built/changed, audit scores, and the tracked-gaps backlog that keeps this a living wiki.
domain: meta
tags: [log, changelog, audit, backlog, living-wiki]
related: [index, schema]
timestamp: 2026-06-26T00:00:00Z
okf_version: "0.1"
---

# Change Log

Append a dated entry whenever a page is created or refreshed. This is the audit trail that makes the wiki *living* (see [SCHEMA.md §7](./SCHEMA.md#7-the-living-maintenance-model)).

---

## 2026-06-26 — Initial build

Created the library from scratch as an OKF v0.1 wiki via a multi-agent pipeline.

**Method:**
1. **Ground** — read the [Open Knowledge Format](https://cloud.google.com/blog/products/data-analytics/how-the-open-knowledge-format-can-improve-data-sharing/) spec; disambiguated the 2026 surfaces (Cowork, Claude Design, Dispatch/Remote/Routines, Mythos 5) via web research.
2. **Scaffold** — authored [SCHEMA.md](./SCHEMA.md) (the format contract) and the domain tree.
3. **Draft → Verify** — a background workflow ran 18 specialist writers (one per page), each researching official docs and writing a long, example-rich page; then 18 adversarial fact-checkers re-verified risky specifics against official docs and enriched gaps. *(One draft, `cli-and-shortcuts`, stalled mid-stream and was regenerated separately.)*
4. **Audit** — one completeness critic per domain scored coverage and produced a gap punch-list.
5. **Enrich** — 7 targeted agents closed the highest-value gaps (sidebar/Projects/Artifacts controls on Claude.ai; Cowork recovery & folder grants; Design canvas & versioning; Claude Code worktrees & keybindings; model rate-limits/`service_tier`/`stop_reason`/beta-headers; Artifacts panel controls; Mythos 5 verification).
6. **Assemble** — generated [glossary.md](./glossary.md) (~218 terms) and [knowledge-graph.yaml](./knowledge-graph.yaml) from page metadata; authored the root and per-domain indexes.

**Output:** 18 content pages (~9,000+ lines) + 5 meta files.

**Workflow stats:** 39 workflow agents + 8 follow-up agents · ~3.5M+ agent tokens · ~680 tool calls.

### Domain audit scores (post-build, pre-enrichment)
| Domain | Score | Notes |
|---|---|---|
| Models | 5 / 5 | Exceptional depth; pricing self-checking; honest WARN flags. |
| Capabilities | 5 / 5 | Strong disambiguation (Connectors/MCP/Plugins/Skills); version-gated. |
| Platform | 4.5 / 5 | Excellent sourcing; gaps in sidebar/Projects/Artifacts UI — **closed in enrichment**. |
| Claude Code | 4 / 5 | Critical gap was the missing CLI page — **regenerated**; worktrees/keybindings **added in enrichment**. |

### Notable verification outcomes (intellectual honesty)
- ⚠️ **`claude-opus-4-8[1m]` correction (2026-06-26):** the verify pass initially flagged `claude-opus-4-8[1m]` as a fabricated SKU and removed it. That was an over-correction — user confirmed (and the runtime confirms) it is a **real Claude Code / agent-runtime model handle** for the explicit 1M-context variant of Opus 4.8. Restored with the accurate framing: the *public API* SKU is bare `claude-opus-4-8` (full 1M at standard pricing, no `[1m]` API SKU), while `claude-opus-4-8[1m]` is the *runtime/Claude Code* variant handle. Fixed in `models/model-families.md` and `glossary.md`. Lesson: "not in the public API catalog" ≠ "fabricated" — Claude Code/runtime model handles are a separate, real namespace.
- ✅ **Mythos 5 confirmed real** (`claude-mythos-5`, launched 2026-06-09) — it postdates the early-2026 session knowledge cutoff, so its initial absence was staleness, not error.
- ✏️ **Corrected brief assumptions:** `service_tier` accepts only `auto`/`standard_only` as a *request* value (`priority` is a *response* value); there is **no `output-300k` beta header** (128K output is native on Claude 4+).
- Documented doc-level contradictions rather than papering over them (RAG-on-Free, Skills-Free eligibility, claude.ai Skills menu path).

---

## Tracked gaps / backlog (open work)

Honest list of what's still thin or unverified, for the next refresh pass. None block use; all are flagged inline on their pages with **⚠️ verify** / in *Open questions*.

**Models**
- A dedicated **Files API** treatment (upload/list/retrieve/delete, container vs persistent, retention).
- Token-efficient tool use, `mcp_servers` API param, and tool-call limits in the tool-use section.
- Prompt-cache 1-hour TTL header status (GA vs beta) and cache-aware rate-limiting detail.
- Multilingual section is thin (no benchmarked-language list).

**Claude Code**
- The full canonical set of `CLAUDE_CODE_DISABLE_*` env vars (only ~7 enumerated).
- A few `settings.json` defaults remain unverified (`cleanupPeriodDays`, `MCP_TIMEOUT`, the `theme` enum).

**Platform**
- Exact UI labels across Claude.ai (Starred vs Favorites; whether the desktop Chat/Cowork/Code switcher is tabs/dropdown/rail), per-plan numeric usage limits, voice option names.
- Cowork: the literal Add-folder picker UI and revoke control; whether grants are per-task or persistent.
- Claude Design: in-canvas viewport presets, formal version-history/restore UI vs the conversational save-and-reference flow, canvas keyboard shortcuts.

**Capabilities**
- A unified **Settings → Capabilities** map (the single toggle surface that gates artifacts/memory/code-execution/skills) — candidate for a future page.
- Per-chat connector on/off persistence; project-level model/effort selector; per-skill toggles on claude.ai.

**Possible future pages** (currently dangling links / future-work markers in [knowledge-graph.yaml](./knowledge-graph.yaml)):
- `memory` — cross-chat & project Memory currently documented inside [claude-ai.md](./platform/claude-ai.md#memory); may warrant its own cross-surface page (also covering Claude Code `CLAUDE.md`).

---

## Maintenance recipes

**Refresh a page** (paste to any Claude agent):
> "Read `claude-wiki/SCHEMA.md`, then update `<page>` against the latest official docs (docs.claude.com, code.claude.com/docs, platform.claude.com/docs). Correct stale facts, mark unconfirmable specifics ⚠️, bump `timestamp` and `confidence`, and append a one-line entry to `log.md`."

**Regenerate the glossary & knowledge graph** after editing pages — re-run the extractor that reads each page's frontmatter `entities`/`related` and rewrites [glossary.md](./glossary.md) and [knowledge-graph.yaml](./knowledge-graph.yaml). (Build script: `extract.js`; point it at the page set and the wiki root.)

**Decay confidence:** Claude ships fast — any page whose `timestamp` is more than a couple of months old should have its `confidence` treated as stale until re-verified.
