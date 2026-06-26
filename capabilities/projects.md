---
type: Feature
title: Projects
description: Dedicated, reusable Claude.ai workspaces that bundle chats around a shared knowledge base, custom instructions, and per-project memory.
domain: capabilities
tags: [projects, knowledge-base, custom-instructions, rag, memory, collaboration, claude-ai, context-window, artifacts, connectors]
related: [claude-ai, artifacts, connectors, skills, capabilities-and-modes, mcp]
resource: https://support.claude.com/en/articles/9517075-what-are-projects
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://support.claude.com/en/articles/9517075-what-are-projects
  - https://support.claude.com/en/articles/9519177-how-can-i-create-and-manage-projects
  - https://support.claude.com/en/articles/11473015-retrieval-augmented-generation-rag-for-projects
  - https://support.claude.com/en/articles/8606394-how-large-is-the-context-window-on-paid-claude-plans
  - https://support.claude.com/en/articles/9797557-usage-limit-best-practices
  - https://support.claude.com/en/articles/10185728-understanding-claude-s-personalization-features
  - https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
  - https://support.claude.com/en/articles/8664678-change-the-model-effort-and-thinking-settings
  - https://www.anthropic.com/news/projects
  - https://claude.com/blog/skills-explained
  - https://support.claude.com/en/articles/10166901-use-google-workspace-connectors
---

# Projects

Projects are self-contained workspaces inside Claude.ai that bundle a set of chats around a **shared, persistent context** — a knowledge base of uploaded files, a set of custom instructions, and (when memory is on) a dedicated per-project memory. Instead of re-pasting your style guide, codebase, or research corpus into every new conversation, you load it once into the project and every chat in that project inherits it. Projects are the canonical "this is what you need to know" container in Claude.ai, complementing Skills ("this is how to do things") and Connectors ("here's where the live data lives").

**At a glance**

| | |
|---|---|
| **What it is** | A reusable Claude.ai workspace = knowledge base + project instructions + grouped chats + per-project memory. |
| **Where you find it** | `claude.ai/projects`, or the **Projects** entry in the left sidebar of the web / desktop / mobile apps. |
| **Who can use it (by plan)** | **All plans**, including **Free** — *"Projects are available to all users, including those with free Claude accounts."* **Free is capped at 5 projects** (*"Free users can create a maximum of five projects"*). **Pro, Max, Team, Enterprise** get the full experience. **Sharing/collaboration** is **Team & Enterprise only**. |
| **Which model runs project chats** | **The model you pick in the model selector** (next to the send button), not a fixed "Projects model." Claude remembers your last selection; Team/Enterprise admins can set an org-wide default (**Organization settings → Models**). Projects originally launched in 2024 on Claude 3.5 Sonnet — that is history, not the current default. |
| **Status** | GA and actively evolving. Now supports RAG-backed knowledge, per-project memory, connector-sourced knowledge, and project archiving. |

---

## What a Project is (and is not)

A project gives you three things that a one-off chat does not:

1. **A knowledge base** — documents, PDFs, text, and code you upload once and reuse across *every* chat in the project.
2. **Project (custom) instructions** — a system-prompt-like block that shapes tone, role, and rules for all chats in the project.
3. **A grouped chat history with its own memory** — all conversations you start inside the project live together and (optionally) share a project-scoped memory summary that is **siloed from your other chats**.

What a project is **not**:

- It is **not** a shared live document — each chat is still a separate conversation. Per the docs: *"Context is not shared across chats within a project unless the information is added into the project knowledge base."* If you want chat A's findings available to chat B, put them in the knowledge base (or rely on project memory).
- It is **not** a Skill. Projects hold **background knowledge** ("here's what you need to know"); Skills hold **procedural know-how and executable code** that loads on demand ("here's how to do things"). See [./skills.md](./skills.md).
- It is **not** a Connector. Connectors stream **live external data** at query time; a project knowledge base is a **curated snapshot** you control. The two combine well — see [Connectors as a knowledge source](#connectors-as-a-knowledge-source).

> Mental model from Anthropic's own guidance: *"Projects say 'here's what you need to know.' Skills say 'here's how to do things.' MCP connects Claude to data; Skills teach Claude what to do with that data."*

---

## Where Projects live in the UI

- **List view:** `claude.ai/projects` or sidebar → **Projects**. Shows your active projects, an **Archived** tab, and on Team/Enterprise a **Shared with me** tab (plus, on Team, a shared-activity surface).
- **Project main page:** left = chat composer + project chat history; **right = the knowledge base panel** ("**Project knowledge**") with the **+** add button; top = project name, description, **Set project instructions**, the **"..."** menu (star / archive / delete), and (Team/Enterprise) **Share project**.
- **Star** a project (card **"..."** menu → **Star**, or the star icon inside the project) to pin it for quick sidebar access.
- **Move a chat** into a project: dropdown next to the chat's name → **Add to project**, or bulk-select chats on your history page and click the project icon. Useful for retroactively organizing, and for controlling what feeds project memory.
- **Archive** a project (via the **"..."** menu) to retire it without deleting; it moves to the **Archived** tab and can be unarchived later.

---

## The project knowledge base

The knowledge base is the heart of a project. Anything you add to the right-hand **Project knowledge** panel is treated as context for **all** chats in that project.

### What you can add

- **Files**: documents, PDFs, plain text, spreadsheets, and source code via the **+** button.
- **Pasted text**: paste a long passage and save it as a knowledge "doc" without a file.
- **Connector-sourced files**: e.g. pull a Google Drive doc into the knowledge base (see below).

### Limits and capacity

| Limit | Value | Notes |
|---|---|---|
| Per-file size | **30 MB** | Applies to each file you add to Project knowledge (the same 30 MB-per-file limit that applies to file uploads across Claude.ai). PDFs over 30 MB can sometimes be handled via Claude's code environment instead of the context window. |
| Number of files | Effectively **unlimited** | Total content must fit the window, or RAG engages. |
| Base context window | **200K tokens** (≈ "500 pages of text or more") | The in-context budget before RAG, on most models. |
| Larger window (current top models) | **500K tokens on all paid plans** | Per the official context-window article, **Claude Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6** support a **500K-token** window when chatting on any paid plan — so the in-context budget (and the point at which RAG kicks in) is larger when you're on one of these models. (Claude Code separately offers 1M for these models + Fable 5.) See [../models/capabilities-and-modes.md](../models/capabilities-and-modes.md). |
| Effective capacity with RAG | **Up to 10×** the window | RAG auto-expands beyond the in-context window — *"up to 10x more content in your projects… while maintaining response quality."* |
| Free-plan projects | **5 projects max** | Hard cap on Free. Paid plans have no documented project-count cap. |

> Caching note (confirmed): Anthropic's usage-limit guidance states *"Content in projects is cached and doesn't count against your limits when reused… When you upload documents to a project, they're cached for future use. Every time you reference that content, only new/uncached portions count against your limits."* Practical upshot: upload your core working documents into Project knowledge once and reference them across many chats without burning messages re-sending them.

### How retrieval works — in-context vs. RAG

There are two modes, and Claude switches between them automatically:

1. **In-context mode** (small knowledge bases): everything fits in the window (200K, or 500K on the current top models) and is loaded directly. Full fidelity, nothing is "searched."
2. **RAG mode** (large knowledge bases): when the knowledge **approaches or exceeds the context window limits**, Claude switches to **Retrieval Augmented Generation**. Instead of loading everything, it uses a **project knowledge search tool** to pull only the **most relevant** chunks per query.

Key RAG facts:

- **Automatic**: no setup, no toggle — *"RAG activates automatically when needed. No setup or configuration is required."* It engages when the project approaches/exceeds the window and **reverts** to in-context mode if you delete enough knowledge to drop below the threshold.
- **Visible**: you'll see *"a visual indicator showing that your project is RAG-enabled,"* and you'll watch Claude **invoke the knowledge search tool** in its response.
- **Capacity**: stores *"up to 10x more content in your projects… by up to 10x while maintaining response quality."* (Treat 10× as Anthropic's stated guidance, not a contractual hard cap.)
- **Quality**: *"Response accuracy remains consistent with in-context processing."*
- **Trade-off to know**: RAG retrieves only the *relevant* slice per query — **full retrieval of the entire corpus is not guaranteed in a single response**. For exhaustive "read every line" tasks, keep the knowledge small enough to stay in-context, or ask narrowly.

> **WARN — documented plan conflict.** Anthropic's two official articles disagree on whether RAG is gated to paid plans:
> - **"What are projects?"** says: *"Enhanced project knowledge with RAG is only available to users with paid Claude plans (Pro, Max, Team, or Enterprise)."*
> - **"Retrieval augmented generation (RAG) for projects"** says: *"RAG for projects is available for all Claude plans (free, Pro, Max, Team, and Enterprise)."*
>
> Both were live as of 2026-06-26. The safest reading: **RAG is fully supported on paid plans; its availability on Free is ambiguous** in the docs. If accuracy matters for a Free account, verify in-product (look for the RAG indicator) rather than relying on either article alone.

---

## Project (custom) instructions

Project instructions are a per-project instruction block applied to every chat in the project — think of it as a project-scoped system prompt that layers on top of your account-wide **Profile instructions**.

- **Set it:** project page → **Set project instructions** → write the guidance → **Save instructions**.
- **Scope:** applies to *all* chats in this project only; does not affect chats elsewhere.
- **Stacks with Profile instructions:** account-wide preferences (Settings) + project instructions + any active Skill/style all compose.

Example project instructions for a brand-voice writing project:

```text
You are the in-house copy editor for Northwind Coffee.
- Voice: warm, plainspoken, never corporate. Short sentences.
- Always use American spelling and the Oxford comma.
- Reference the brand style guide and approved-claims list in Project knowledge
  before making any health or sourcing claim. If a claim isn't supported there,
  flag it instead of writing it.
- Output drafts as an Artifact titled "Draft — <topic>" so I can iterate inline.
```

---

## Per-project memory

Memory is available to **all Claude users (Free, Pro, Max, Team, and Enterprise)** across web, Desktop, and Mobile. When memory is on, **each project gets its own memory space and a dedicated project summary**, kept separate from your other projects and from your non-project chats.

| Aspect | Behavior |
|---|---|
| **Plan availability** | Memory is available on **all plans** (Free, Pro, Max, Team, Enterprise). |
| **What it remembers** | Per Anthropic: your role/projects/professional context, communication preferences and working style, technical preferences and coding style, and ongoing project details. |
| **Isolation** | *"Each project has its own separate memory space and dedicated project summary, so the context within each of your projects is focused, relevant, and separate from other projects or non-project chats."* |
| **Standalone synthesis** | Claude synthesizes key insights across your standalone (non-project) chat history; this synthesis *"is updated every 24 hours and provides context for every new standalone conversation."* **Project chats are excluded** from that standalone synthesis (they feed only their own project memory). |
| **Toggling** | **Settings → Capabilities.** Options: **Pause** (*"Claude keeps its existing memory but won't use memory or make new memories"*) and **Reset** (*"Permanently deletes all memories including project memories"*). |
| **Incognito** | Incognito chats aren't saved to history or memory — *"Claude won't remember your chats, so they won't be saved to Claude's memory or your chat history."* |
| **Team** | Members manage memory individually — *"Individual Team plan members manage their own memory settings directly"*; no org-level control. |
| **Enterprise** | Owners can disable memory org-wide via **Organization settings → Capabilities**; members' access depends on that org setting. |
| **Moving chats** | Moving a chat **into/out of** a project changes which memory summary it feeds — a lever for curating project context vs. standalone synthesis. |

Why this matters: memory isolation means your "Q3 board deck" project won't leak phrasing or facts into your "personal travel planning" chats, and vice versa.

---

## Conversations within a Project

Every chat started from the project page is a **project chat**:

- It automatically inherits the **knowledge base** + **project instructions** + **project memory**.
- It appears in the project's grouped history, not your global chat list.
- Sibling chats are **independent** — they don't see each other's transcripts unless that content is promoted into Project knowledge or captured by project memory.
- You can **move** an existing standalone chat into the project (dropdown next to the chat name) to bring it under the project's context and memory.

Typical pattern: keep the project knowledge base as the durable "facts," spin up a fresh chat per task (e.g., "draft the FAQ", "summarize the new contract", "write the migration plan"), and let each chat stay focused while sharing the same grounding.

---

## Sharing & collaboration (Team & Enterprise)

Sharing is **only available on Team and Enterprise** plans.

**Visibility at creation** (Team/Enterprise): choose **Keep it private** or **Share with your broader organization**.

**Share an existing project:** open it → **Share project** → add people by **name, email, or bulk email list**, or share org-wide.

**Permission levels:**

| Level | Rights |
|---|---|
| **Can use** | View the project and chat in it; **cannot edit** knowledge/instructions/members. |
| **Can edit** | Modify the project — knowledge base, instructions, **and member settings**. |

**Managing access:** change permissions or remove people from the sharing menu (**only the creator can remove access**). Recipients find shared projects under the **Shared with me** tab, and **email notifications** are sent when a project is shared with them.

**Archiving a shared project:** archiving **resets sharing permissions back to private** — re-share after unarchiving if you still want collaborators.

**Shared activity (Team):** Team users can share **conversation snapshots** into a **shared project activity feed**, so teammates can discover useful prompting approaches and reuse them.

**Privacy:** Anthropic states shared project data **won't be used to train generative models without explicit consent**. (WARN: verify against your org's current data-use settings / DPA.)

---

## Artifacts within Projects

Artifacts work inside projects exactly as elsewhere — generated code, documents, diagrams, and designs render in a dedicated **side panel** with larger code windows and **live frontend previews** for review. Two project-specific synergies:

- **Style-grounded artifacts:** because the knowledge base + project instructions are always present, an artifact (e.g., a doc, a React component, a dashboard) is generated *already conforming* to your project's conventions.
- **Iterative review:** keep refining the same artifact across turns within the project; the knowledge base keeps each revision on-brand.

See [./artifacts.md](./artifacts.md) for the full artifact reference (types, editing, publishing, live dashboards).

---

## Connectors as a knowledge source

Connectors let a project's knowledge stay close to live source-of-truth systems. The most common pattern is **Google Drive → Project knowledge**:

- You can add **Google Drive** files to a project's knowledge base. Per the docs, *"Google Docs added to chats and projects sync directly from Google Drive, so you're always working with the latest version"* — so the knowledge stays current rather than being a frozen snapshot.
- **Confirmed caveat:** *"The Google Drive connector is only available when adding to Files in private projects. This option will be disabled for shared projects."* So Drive-into-knowledge is **private-projects-only**.
- Multiple Drive files can be added for broader context, subject to the same window/RAG behavior as uploaded files.

For the catalog, OAuth, admin controls, and custom MCP connectors, see [./connectors.md](./connectors.md) and [./mcp.md](./mcp.md).

---

## How to create & manage a Project (step-by-step)

```text
CREATE
1. Go to claude.ai/projects (or hover the left sidebar → Projects).
2. Click "+ New Project" (upper right corner).
3. Give it a name and description.
   NOTE: "Claude will not have access to these details" — name/description are
   organizational metadata for you, not context Claude reads.
4. (Team/Enterprise) Choose private or "Share with your broader organization".

ADD KNOWLEDGE
5. In the right-hand "Project knowledge" panel, click "+".
6. Upload documents/text/code, paste text, or add a Google Drive file
   (Drive option is private-projects-only; disabled for shared projects).
   Limits: 30MB per file; unlimited files; 200K window (500K on Opus 4.8/4.7/4.6
   and Sonnet 4.6) then automatic RAG (up to 10x) on paid plans.

SET INSTRUCTIONS
7. Click "Set project instructions" → write guidance → "Save instructions".

WORK
8. Start chats from the project page; each inherits knowledge + instructions + project memory.
9. Star the project to pin it:
     - Projects page: card "..." menu → "Star".
     - Inside a project: star icon in the upper-right corner.
10. Move existing chats in:
     - Single chat: dropdown arrow next to the chat name → "Add to project".
     - Bulk: from your chat history page, select multiple chats → click the project icon.

SHARE (Team/Enterprise)
11. "Share project" → add by name, email, or bulk-paste email list, or share org-wide
    → assign "Can use" or "Can edit". Recipients see it under "Shared with me".

ARCHIVE / UNARCHIVE (instead of delete, when you want to keep it recoverable)
12. Project "..." menu (upper right) → archive → confirm. Archived projects live under
    the "Archived" tab on the Projects page. Archiving a SHARED project resets its
    sharing permissions back to private.
13. To unarchive: Archived tab → "..." → "Unarchive" (or open it and confirm).

DELETE
14. Projects page or open project → "..." menu → "Delete" → confirm "Yes, delete".
    NOTE: you cannot delete an archived project — unarchive it first.
```

---

## Worked examples

### 1. Codebase companion (engineering)

```text
Name:        Payments Service Companion
Description: Context for our Go payments microservice.
Knowledge:   service README, ARCHITECTURE.md, the OpenAPI spec, key handler files,
             the runbook, and the last 3 incident postmortems.
Instructions:
  "You are a senior engineer on the payments team. Prefer idiomatic Go.
   Cite the file in Project knowledge you're basing changes on. When proposing
   schema changes, include a migration plan and a rollback. Never invent
   endpoint names — check the OpenAPI spec in knowledge first."
Use it for: 'Write a retry wrapper for the Stripe client matching our error
            conventions', 'Draft the postmortem for INC-4412', 'Explain the
            settlement flow to a new hire.'
```

Because the knowledge base is reused, every chat already "knows" the architecture — no re-pasting. With a large repo, RAG retrieves the relevant files per question.

### 2. Research project (analysis)

```text
Name:        Battery Supply-Chain Review
Knowledge:   ~40 PDFs (analyst reports, filings, interview transcripts),
             a glossary doc, and a "key questions" doc.
Instructions:
  "Answer only from Project knowledge; if a claim isn't supported there, say so.
   Always cite the source document name. Use a neutral, analytical tone."
Use it for: literature synthesis, contradiction-spotting across sources,
            and generating a cited brief as an Artifact.
```

This corpus exceeds the in-context window (200K, or 500K on Opus 4.8/4.7/4.6 and Sonnet 4.6), so **RAG engages automatically** — you'll see the knowledge-search tool fire and a RAG indicator. Keep questions specific to maximize retrieval relevance, and remember the trade-off: a single response retrieves only the relevant slice, not the entire corpus.

### 3. Writing project with a style guide (content)

```text
Name:        Northwind Blog
Knowledge:   brand style guide, tone-of-voice doc, approved-claims list,
             5 exemplar posts.
Instructions: (the brand-voice block shown earlier in this page)
Use it for: 'Draft a 600-word post on our new cold brew', 'Rewrite this in
            our voice', 'Generate 10 headline options' — each output already
            on-brand, delivered as an editable Artifact.
```

---

## Projects vs. Skills vs. Connectors vs. plain chat — when to use which

| You want… | Use | Why |
|---|---|---|
| A reusable bundle of **background knowledge** for many related chats | **Projects** | Knowledge base + instructions persist across the project's chats. |
| **Procedural know-how / executable code** that loads only when relevant | **Skills** | On-demand, context-window-efficient. See [./skills.md](./skills.md). |
| **Live, up-to-the-minute external data** at query time | **Connectors / MCP** | Streams from the source system. See [./connectors.md](./connectors.md). |
| A quick one-off with no reusable context | **Plain chat** | No setup overhead. |

These compose: a **Project** (knowledge) + a **Skill** (procedure) + a **Connector** (live data) in the same conversation is the most powerful configuration.

---

## Related pages

- [../platform/claude-ai.md](../platform/claude-ai.md) — the Claude.ai app where Projects live; plans, settings, sidebar.
- [./artifacts.md](./artifacts.md) — Artifacts generated inside projects (code, docs, dashboards).
- [./connectors.md](./connectors.md) — pulling live/Drive knowledge into a project.
- [./skills.md](./skills.md) — Skills vs. Projects, and combining them.
- [./mcp.md](./mcp.md) — Model Context Protocol behind custom connectors.
- [../models/capabilities-and-modes.md](../models/capabilities-and-modes.md) — context windows, 1M context, extended thinking.
- [../models/model-families.md](../models/model-families.md) — model IDs (Opus 4.8/4.7/4.6, Sonnet 4.6) and their context windows.

## Open questions / to verify

- **RAG on the Free plan — documented contradiction (unresolved):** "What are projects?" says RAG is *paid-only*; "RAG for projects" says it's available on *all plans including free*. Both were live 2026-06-26. Treat Free-plan RAG as ambiguous until Anthropic reconciles the two articles.
- Exact **RAG activation threshold** in tokens. Docs say "approaches or exceeds the context window limits" but don't publish a precise trigger number; the window itself is 200K (or 500K on Opus 4.8/4.7/4.6 and Sonnet 4.6), so the trigger is model-dependent.
- Whether the full set of **connectors that can populate a knowledge base** extends beyond Google Drive (Drive confirmed private-projects-only; others not enumerated in the projects docs).
- Whether RAG quality genuinely matches in-context for **exhaustive "read every document" tasks** (docs claim parity, but RAG by design retrieves a relevant slice per query).

**Resolved since the prior draft** (no longer open): per-project memory is on all plans; project-knowledge caching does not count reused content against limits; Drive-into-knowledge is private-projects-only; project chats use the model you select (no fixed "Projects model"); the 200K window now has a 500K variant on current top models; archiving/unarchiving exists.

## Sources

- [What are projects? — Claude Help Center](https://support.claude.com/en/articles/9517075-what-are-projects)
- [How can I create and manage projects? — Claude Help Center](https://support.claude.com/en/articles/9519177-how-can-i-create-and-manage-projects)
- [Retrieval augmented generation (RAG) for projects — Claude Help Center](https://support.claude.com/en/articles/11473015-retrieval-augmented-generation-rag-for-projects)
- [How large is the context window on paid Claude plans? — Claude Help Center](https://support.claude.com/en/articles/8606394-how-large-is-the-context-window-on-paid-claude-plans)
- [Usage limit best practices (project caching) — Claude Help Center](https://support.claude.com/en/articles/9797557-usage-limit-best-practices)
- [Understanding Claude's personalization features — Claude Help Center](https://support.claude.com/en/articles/10185728-understanding-claude-s-personalization-features)
- [Use Claude's chat search and memory — Claude Help Center](https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context)
- [Change the model, effort, and thinking settings — Claude Help Center](https://support.claude.com/en/articles/8664678-change-the-model-effort-and-thinking-settings)
- [Collaborate with Claude on Projects — Anthropic](https://www.anthropic.com/news/projects)
- [Skills explained: Skills vs Prompts, Projects, MCP, subagents — Claude](https://claude.com/blog/skills-explained)
- [Use Google Workspace connectors — Claude Help Center](https://support.claude.com/en/articles/10166901-use-google-workspace-connectors)
