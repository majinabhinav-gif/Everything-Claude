---
type: Reference
title: Learning Resources — Claude Video Courses & Links
description: Curated, swap-automatable index of external tutorials, talks, and courses about Claude — pulled from the training catalog, with YouTube publish dates so stale videos can be replaced automatically.
domain: resources
tags: [resources, learning, tutorials, youtube, courses, external]
related: [claude-ai, cowork, claude-design, claude-code-overview, connectors, skills, plugins, mcp, subagents, model-families]
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
source_workbook: "D:/traingen website/New Page/Book1.xlsx (sheet: Course List)"
---

# Learning Resources — Claude Video Courses & Links

Curated external tutorials, talks, and courses about Claude, pulled from the training catalog (`Book1.xlsx` → *Course List*). Every entry links to its source, names the creator, and — for YouTube videos — records the **publish date** and **video ID** so outdated videos can be swapped automatically when better ones appear. Each resource is mapped to the wiki page it supports.

> **At a glance** — **What it is:** a human-readable index + a machine-readable companion ([`claude-video-resources.yaml`](./claude-video-resources.yaml)). **Who it's for:** anyone who wants to *watch/learn* rather than read the reference. **Status:** living — see *Automated swapping* below.

## How automated swapping works

YouTube content ages: a "best tutorial" today may be superseded next quarter. To keep this list fresh without manual auditing, every video entry carries machine-readable metadata in [`claude-video-resources.yaml`](./claude-video-resources.yaml):

- `youtube_id`, `url`, `channel`, **`published`** (ISO date), `topic`, `maps_to` (the wiki page it supports), `added`, `last_verified`, `status`, and `swap_policy`.
- A future agent/script can, per entry: search YouTube for a newer, higher-quality video on the same `topic`, compare its upload date against `published`, and — if a better/newer one exists — replace `url` + `youtube_id` + `published`, bump `last_verified`, and append a note to [`../log.md`](../log.md).

**Refresh recipe** (paste to any Claude agent):

> "Read `claude-wiki/resources/claude-video-resources.yaml`. For each entry with `status: active`, check whether a newer or clearly better YouTube video now covers the same `topic`. If so, update `url`/`youtube_id`/`published`/`last_verified` in the YAML and the matching row in `resources/index.md`, and log the swap in `claude-wiki/log.md`. Leave entries unchanged when the current pick is still best; always bump `last_verified`."

## Getting started & core features

| Topic | Video / resource | Creator | Published | Difficulty | Supports |
|---|---|---|---|---|---|
| Claude Tutorial | [Tutorials \| Claude](https://claude.com/resources/tutorials) | Anthropic | — | Easy | [index](../index.md) |
| Claude For Begineers | [Claude AI Tutorial for Beginners (Step-by-Step)](https://www.youtube.com/watch?v=r2vYObllqJU) | Kevin Stratvert | 2026-04-30 | Easy | [claude-ai](../platform/claude-ai.md) |
| Claude Cowork | [Claude Cowork Fundamentals In 22 Minutes](https://www.youtube.com/watch?v=uGwDuvSqgYI) | Tina Huang | 2026-05-10 | Easy | [cowork](../platform/cowork.md) |
| Claude Design | [Claude Design Just Changed Everything (New Update)](https://www.youtube.com/watch?v=VtD-KTeyxaU) | Griffin Wooldridge | 2026-06-19 | Easy | [claude-design](../platform/claude-design.md) |
| Claude Code for Beginners | [Claude Code - Full Tutorial for Beginners](https://www.youtube.com/watch?v=ntDIxaeo3Wg) | Tech With Tim | 2026-02-27 | Intermediate | [claude-code](../platform/claude-code.md) |
| Claude Connectors | [Claude Connectors Tutorial for Beginners](https://www.youtube.com/watch?v=FpRNId-jeTQ) | Kevin Stratvert | 2026-06-18 | Easy | [connectors](../capabilities/connectors.md) |
| Claude Skills | [Claude Skills Explained Simply (Master in 7 Minutes)](https://www.youtube.com/watch?v=O6tQ6V_P8a0) | Tristen O'Brien | 2026-05-23 | Easy | [skills](../capabilities/skills.md) |
| Claude PlugIns | [Claude Code Plugins Explained In 7 Minutes](https://www.youtube.com/watch?v=_lBG5h3AY50) | Software Engineer Meets AI | 2026-03-29 | Intermediate | [plugins](../capabilities/plugins.md) |

## Technical & building

| Topic | Video / resource | Creator | Published | Difficulty | Supports |
|---|---|---|---|---|---|
| Claude Tricks & tips | [32 Tricks to Level Up Claude Code in 16 Mins](https://www.youtube.com/watch?v=jqoFP9QapXI) | Nate Herk \| AI Automation | 2026-04-27 | Easy | [cli-and-shortcuts](../claude-code/cli-and-shortcuts.md) |
| Building Agents using Claude Code | [How to Build Effective Claude Code Agents in 2026](https://www.youtube.com/watch?v=RzLV8sfFdMM) | Nate Herk \| AI Automation | 2026-06-18 | Intermediate | [subagents](../claude-code/subagents.md) |
| AI for developers | [Why we built—and donated—the Model Context Protocol (MCP)](https://www.youtube.com/watch?v=PLyCki2K0Lg&list=PLf2m23nhTg1PBzCb-nOGFH6NFYSkkVuZ5) | Anthropic | 2025-12-11 | Advanced | [mcp](../capabilities/mcp.md) |
| Create Brain | [Build A Claude Knowledge Base That Self-Improves!](https://www.youtube.com/watch?v=ib74sLgjIBM) | Systems Made Better | 2026-05-23 | Intermediate | [SCHEMA](../SCHEMA.md) |

## Models & ecosystem

| Topic | Video / resource | Creator | Published | Difficulty | Supports |
|---|---|---|---|---|---|
| Claude mythos | [Claude Fable 5 Is INSANE – Hands-On With the BEST Model Yet!](https://www.youtube.com/watch?v=9GLYsrMpprs&t=675s) | Bijan Bowen | 2026-06-09 | Easy | [model-families](../models/model-families.md) |

## Use cases

| Topic | Video / resource | Creator | Published | Difficulty | Supports |
|---|---|---|---|---|---|
| Claude for Finance | [Accelerating private equity deal flows with Claude](https://www.youtube.com/playlist?list=PLf2m23nhTg1PTXjULcWHgzJn1oBvC6NuV) | Anthropic | playlist | Intermediate | [claude-ai](../platform/claude-ai.md) |
| Generating Videos using AI | [Seedance 2.0 + Claude Changed How I Make AI Videos](https://www.youtube.com/watch?v=Nqifd6VHU1k) | Roboverse | 2026-05-30 | Intermediate | [artifacts](../capabilities/artifacts.md) |

## Notes

- **Source of truth:** rows tagged Claude/Anthropic in `Book1.xlsx` → *Course List*. Column C carried the real hyperlink targets (extracted via the cell hyperlink, not the display text).
- **Dates** are the YouTube **upload dates** scraped from each watch page (`uploadDate`). A `—` means a non-video resource; `playlist` means a multi-video playlist (no single date).
- Creator/title shown are the **canonical YouTube** values where available, otherwise the catalog's text.

## Related pages

- [Library home](../index.md) · [Glossary](../glossary.md) · [Change log](../log.md)
- Supported pages: [Claude.ai](../platform/claude-ai.md) · [Cowork](../platform/cowork.md) · [Claude Design](../platform/claude-design.md) · [Claude Code](../platform/claude-code.md) · [Connectors](../capabilities/connectors.md) · [Skills](../capabilities/skills.md) · [Plugins](../capabilities/plugins.md) · [MCP](../capabilities/mcp.md) · [Subagents](../claude-code/subagents.md) · [Model families](../models/model-families.md)

## Open questions / to verify

- Publish dates are scraped from YouTube's `uploadDate` field; re-verify if YouTube changes its page markup. A `needs_date: true` flag in the YAML marks any entry where the scrape failed.
- `maps_to` assignments are best-effort topical matches; adjust as the wiki grows.

## Sources

- Training catalog: `Book1.xlsx` (sheet *Course List*), column C hyperlinks.
- Per-video metadata: each YouTube watch page's `uploadDate` / `og:title` / `ownerChannelName`.
