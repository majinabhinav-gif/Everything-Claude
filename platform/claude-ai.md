---
type: Surface
title: Claude.ai & the Claude Apps
description: Exhaustive reference for the consumer/pro Claude apps — web, desktop, and mobile — covering every surface, chat workspace control, mode toggle, setting, plan, and shortcut.
domain: platform
tags: [claude-ai, web-app, desktop-app, mobile-app, chrome-extension, settings, plans, voice-mode, styles, memory, research, agent-mode]
related: [cowork, claude-code-overview, claude-design, model-families, capabilities-and-modes, artifacts, projects, skills, connectors, memory]
resource: https://support.claude.com/en/articles/8114491-get-started-with-claude
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://support.claude.com/en/articles/8114491-get-started-with-claude
  - https://support.claude.com/en/articles/12138966-release-notes
  - https://support.claude.com/en/articles/11049741-what-is-the-max-plan
  - https://support.claude.com/en/articles/9266767-what-is-the-team-plan
  - https://support.claude.com/en/articles/11095361-when-should-i-use-web-search-extended-thinking-and-research
  - https://support.claude.com/en/articles/11088861-use-research-on-claude
  - https://support.claude.com/en/articles/10684626-enable-and-use-web-search
  - https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context
  - https://support.claude.com/en/articles/8664678-change-the-model-effort-and-thinking-settings
  - https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work
  - https://support.claude.com/en/articles/10181068-styles-are-moving-to-skills
  - https://support.claude.com/en/articles/11101966-use-voice-mode
  - https://support.claude.com/en/articles/12012173-get-started-with-claude-in-chrome
  - https://support.claude.com/en/articles/12626668-use-quick-entry-with-claude-desktop-on-mac
  - https://privacy.claude.com/en/articles/12109829-how-do-i-change-my-model-improvement-privacy-settings
  - https://www.anthropic.com/news/higher-limits-spacex
  - https://www.claude.com/pricing
---

# Claude.ai & the Claude Apps

The "Claude apps" are the consumer- and Pro-facing surfaces where you talk to Claude through a chat UI: the **claude.ai web app**, the **macOS and Windows desktop apps**, the **iOS and Android mobile apps**, and **Claude in Chrome** (browser extension). They share one Anthropic account, one conversation history, and one settings store, so a chat started on your phone continues on your laptop. This page documents the chat workspace, every mode toggle, conversation management, the full Settings tree, plans, and keyboard shortcuts — the agentic surfaces (Cowork, Code) and the deep capabilities (Artifacts, Projects, Skills, Connectors) each have their own pages, cross-linked below.

> **At a glance**
> - **What it is:** The chat-first product family for using Claude — web, desktop (Mac/Windows), mobile (iOS/Android), and a Chrome extension, all synced to one account.
> - **Where you find it:** [claude.ai](https://claude.ai) in a browser; native desktop apps from claude.ai/download; mobile apps from the Apple App Store / Google Play; "Claude in Chrome" from the Chrome Web Store.
> - **Who can use it by plan:** Free, Pro, Max (5×/20×), Team, Enterprise. Chat is on every plan; agentic surfaces (Cowork, Code, Research) and higher limits are gated to paid plans — see [Plans](#plans-free--pro--max--team--enterprise).
> - **Status:** Active and evolving fast. Many 2026 features (Memory for all users, interactive apps on mobile, Claude in Chrome for all paid plans, the Styles→Skills migration) are recent; specifics marked **WARN: verify** are subject to change.

---

## Surfaces (where the apps run)

| Surface | How to get it | Distinctive capabilities | Notes |
|---|---|---|---|
| **Web app** | Visit [claude.ai](https://claude.ai), sign in | Full chat, Artifacts, Projects, Research, Connectors, all Settings | The canonical surface; everything ships here first or simultaneously. |
| **Desktop — macOS** | Download from claude.ai/download | Adds **Cowork** and **Claude Code**; local file access; **Quick Entry** global launcher (double-tap Option); optional Caps Lock dictation; menu-bar presence | Cowork desktop runs an agent in an isolated local VM. Cowork reached general availability **April 9, 2026**. See [./cowork.md](./cowork.md). |
| **Desktop — Windows** | Download from claude.ai/download | Same Cowork + Code surfaces as macOS (Cowork GA April 9, 2026) | **Quick Entry is macOS-only** today; on Windows you can pin Claude to the taskbar or assign a shortcut to the desktop icon. Desktop extensions / MCP connectors configurable locally. |
| **iOS** | App Store | Voice mode, camera capture, share-sheet import, Siri Shortcuts, interactive apps, Apple Health analysis (US, Pro/Max) | Interactive apps reached mobile **March 25, 2026**. Health/fitness analysis reported for US Pro/Max (Jan 2026). **WARN: verify** regional availability. |
| **Android** | Google Play | Voice mode, camera/photo upload, share-target import, interactive apps | Interactive apps reached mobile **March 25, 2026**. |
| **Claude in Chrome** | Chrome Web Store extension | Claude reads/clicks/navigates in the active browser tab; sidebar chat; multi-tab, multi-step workflows; downloads files; acts in Slack/Gmail/Calendar | **In beta for all paid plans** (Pro, Max, Team, Enterprise) — no longer waitlisted. Pro is limited to **Haiku 4.5**; Max/Team/Enterprise can pick the model (Opus 4.7, Sonnet 4.6, Haiku 4.5). |

All surfaces authenticate the same way (email + password, Google sign-in, or SSO for Team/Enterprise) and stay in sync. You can hand off mid-conversation by opening the same chat from history on another device ("continue on another device" is implicit — there is no separate handoff button; history is cloud-synced).

---

## The chat workspace

The core of every Claude app is a single conversation surface: a scrollable message thread with the **composer** (message box) pinned at the bottom and a left **sidebar** for navigation.

### Left sidebar / primary navigation

The left sidebar is the app's primary navigation. It can be collapsed/expanded with the toggle at its top-right (`Cmd/Ctrl+Shift+S`); it **cannot be fully disabled**. Contents, top to bottom, are roughly:

| Sidebar item | What it does |
|---|---|
| **New chat** | Starts a fresh conversation (`Cmd/Ctrl+O`). The **ghost / incognito icon** here opens a temporary chat that isn't saved to history (see [Memory](#memory)). |
| **Search chats** | Opens chat search (`Cmd/Ctrl+K`) to find past conversations by title/content; backed by chat search (toggleable in Settings → Capabilities). |
| **Recents / chat history** | The scrollable list of recent conversations, cloud-synced across devices. Each row has a `⋯` menu (rename, star, delete, share, add to project — see [Conversation management](#conversation-management)). |
| **Starred** | Quick-access section for chats — and **starred Projects** — you've pinned. **WARN: verify** exact label ("Starred" vs "Favorites") and whether chats and projects share one section. |
| **Projects** | Entry point to your Projects workspace (`claude.ai/projects`); see [Projects](#projects-on-claudeai) below. **WARN: verify** whether Projects is a fixed top-level item or revealed on hover for all plans. |
| **Artifacts** | Entry point to the Artifacts gallery — your saved/published artifacts that persist between sessions; see [Artifacts panel](#artifacts-panel). **WARN: verify** sidebar placement/label. |
| **Chat / Cowork / Code mode switcher** *(desktop)* | On the **desktop apps**, a mode selector exposes three tabs — **Chat** (conversations), **Cowork** (longer agentic/Dispatch work), and **Code** (software development). In Cowork/Code, the sidebar instead lists sessions/tasks you can filter by status (Active/Archived) and environment (Local/Cloud) and run in parallel. See [./cowork.md](./cowork.md) and [./claude-code.md](./claude-code.md). |
| **Account / profile** | Bottom of the sidebar — your initials/name open the account menu (Settings, plan, sign-out). |

> The Chat/Cowork/Code switcher is a **desktop** affordance; the web app is Chat-first (Cowork/Code surface differently on web). **WARN: verify** the exact on-screen labels and whether the switcher appears as tabs, a dropdown, or rail icons in your build.

### Projects on claude.ai

**Projects** are self-contained workspaces with their own chat history, knowledge base, and instructions — surfaced in the apps as a first-class destination. This page covers only the *surface*; for the knowledge base, custom instructions, and project memory, see [../capabilities/projects.md](../capabilities/projects.md).

- **Open the Projects list** — hover the left side of the sidebar and click **Projects**, or go directly to **`claude.ai/projects`**. Existing projects appear as a list/grid you click to open.
- **Create a project** — click **`+ New Project`** in the upper-right of the Projects page, then give it a name and (optional) description.
- **Open from a chat** — a chat's `⋯` menu has **Add to Project** to file an existing conversation into one (see [Conversation management](#conversation-management)).
- **Plan limits** — Projects are available on **every plan including Free**; **Free is capped at 5 projects**, while paid plans get **unlimited Projects**. **WARN: verify** the current Free cap.

```
Sidebar → Projects → [ + New Project ]
  Name: "Northstar pricing"
  → upload deck + notes to the knowledge base, set instructions, then chat inside it
```

### The composer (message box)

The composer is where you type. Key behaviors:

- **Multi-line input** — `Enter` sends; `Shift+Enter` inserts a newline (this is the default). Paste of multi-line text is preserved.
- **`/` command menu** — typing `/` at the start of the composer opens a menu for in-app actions (e.g. inserting a Style/Skill, starting common flows). Custom slash commands are primarily a **Claude Code** concept; in the apps `/` surfaces a smaller built-in set. **WARN: verify** the exact app-side command list.
- **`+` / attach menu** — opens attachment and tool options: upload files, add from connectors (e.g. Google Drive), apply a **Style**, enable a **Skill**, and start **Research**. Research is launched from this `+` menu (see [Research](#research-deep-research)).
- **Tool toggles** — the **slider/controls icon** in the input opens a dropdown with **Web search** and **Extended thinking** toggles; **Research** is started from the `+` menu (paid plans). See [Modes & toggles](#modes--toggles).
- **Model picker** — a dropdown (near the composer or chat header) to choose the model and, for some models, an **effort**/thinking setting. As of **May 28, 2026** the latest flagship is **Claude Opus 4.8**. See [../models/model-families.md](../models/model-families.md).
- **Send / Stop** — the send button becomes a **Stop** control while Claude is generating, letting you interrupt mid-response.

### Attachments, uploads, and pasting

Claude apps accept a wide range of inputs from the `+` menu, drag-and-drop, or paste:

| Input | How | Notes |
|---|---|---|
| **Files** (PDF, DOCX, CSV, TXT, code, etc.) | `+` → Upload, or drag-drop | Per-file size and per-chat count limits apply. **WARN: verify** exact caps (commonly cited around 30 MB/file, but not confirmed in current official docs). |
| **Images** (PNG/JPG/etc.) | Upload or paste | Vision analysis: read diagrams, charts, photos, UI mockups. |
| **PDFs** | Upload | Text + layout + embedded images parsed; long PDFs count heavily against context. |
| **Screenshots** | Paste from clipboard (`Ctrl/Cmd+V`) | Great for "what does this error mean?" flows. |
| **Camera capture** (mobile) | Camera icon | Snap a photo and ask about it in-line. |
| **From connectors** | `+` → connector (e.g. Google Drive) | Pull a Drive/Workspace file directly. See [../capabilities/connectors.md](../capabilities/connectors.md). |

Example interaction:

```
You: [pastes a screenshot of a stack trace]
     Why is this throwing? It only happens on Windows.
Claude: This is a path-separator bug — line 14 hard-codes "/"…
```

### Message actions

Hovering (desktop/web) or long-pressing/tapping (mobile) a message reveals actions:

| Action | Applies to | Effect |
|---|---|---|
| **Copy** | Claude messages | Copy the message (or a code block) to clipboard. |
| **Edit** | Your messages | Rewrite a prior prompt; resending **forks** the thread from that point. |
| **Retry / Regenerate** | Claude messages | Re-run the last turn; often offers "try again" with the same or a different model. |
| **Branch / fork** | Your messages | Editing-and-resending creates a divergent branch you can switch between (the original is preserved). |
| **Feedback** (👍 / 👎) | Claude messages | Thumbs up/down; thumbs-down opens an optional report dialog. |
| **Open in Artifacts** | Claude messages with rendered output | Pops the code/doc/diagram into the [Artifacts](../capabilities/artifacts.md) side panel. |

> Editing an earlier prompt and re-sending is the canonical way to "go back" — the prior branch is retained and you can navigate between versions of the exchange.

### Interrupting & regenerating

You stay in control of a response while it streams and after it lands:

| Control | Where | Effect |
|---|---|---|
| **Send → Stop** | The send button flips to a **Stop** control while Claude generates | Click to **halt mid-response**; the partial answer is kept and you can continue from there. |
| **Esc to stop** | Keyboard, while generating | Stops the in-progress generation. **WARN: verify** that `Esc` is bound to stop in the apps (it is the stop key in Claude Code's interactive mode; the app binding is not confirmed in the help center). |
| **Retry / Regenerate** | Hover a Claude message → **Retry** | Re-runs the last turn to get a different answer. |
| **Switch model and retry** | Retry / model picker on the message | Some surfaces let you **re-run the same turn with a different model** (e.g. retry on a stronger model after a weak answer). **WARN: verify** exact affordance and label per surface. |

> Stopping a response does not delete it — the partial text remains in the thread, so you can stop a long answer that's already gone where you need, then type a follow-up. To replace an answer entirely, use **Retry**; to change the *prompt*, **Edit** the message (which forks the thread — see [Message actions](#message-actions)).

---

## Modes & toggles

The apps layer several optional behaviors on top of standard chat. Most are per-message toggles in/near the composer; some are account-level.

### Standard chat vs Agent Mode

- **Standard chat** — Claude answers in the thread, optionally calling tools (web search, connectors) it's allowed to use.
- **Agent Mode** — Claude works more autonomously across multiple steps to *do* a task rather than just answer. On desktop this manifests most fully as **Cowork** (an agent running multi-step work in a local VM; GA April 9, 2026) and via **persistent agent threads** you can monitor from desktop or mobile. See [./cowork.md](./cowork.md). **WARN: verify** whether a standalone composer-level "Agent Mode" switch exists on all surfaces vs. being surfaced as Cowork; naming has shifted.

### Web search

Web search is an **opt-in toggle**, off by default. Open the **slider/controls icon** in the chat input, find **Web search** in the dropdown, and switch it on; turn it off for chats that don't need live data. It lets Claude run live searches and ground answers in current web content (with citations).

- **Availability:** supported across current models — **Fable 5, Opus 4.8, Opus 4.7, Sonnet 4.6, Opus 4.6, and Haiku 4.5** — on Free and up.
- **Team/Enterprise:** an Owner must first enable web search at the workspace level under **Admin settings → Capabilities** before members can toggle it per chat.

```
[Web search: ON]
You: What changed in the latest Claude release notes this week?
Claude: According to support.claude.com (June 2026)… [cites pages]
```

### Extended thinking

The **Extended thinking** toggle (same slider/controls dropdown as Web search) lets Claude spend more time reasoning before answering, showing its work in an expandable block. Best for hard math, multi-constraint planning, and tricky debugging. Pairs with the model/effort picker. See [../models/capabilities-and-modes.md](../models/capabilities-and-modes.md).

### Research (deep research)

**Research** is an agentic, multi-search investigation: Claude runs many searches that build on each other, decides what to dig into next, and returns a structured, cited report.

- **Plans:** **paid only** — Pro, Max, Team, or Enterprise. Not on Free.
- **How to start it:** click the **`+` button** at the bottom-left of the chat and select **Research**. A **blue indicator** appears at the bottom of the chat window; click it again to turn Research off.
- **Requirement:** **web search must be enabled** for Research to work.

Use it for "go find and synthesize everything about X" tasks rather than quick lookups. See [../models/capabilities-and-modes.md](../models/capabilities-and-modes.md).

### Styles (response tone/format)

**Styles** shape how Claude writes. Built-in presets historically include **Normal** (default), **Concise**, **Explanatory**, **Formal**, and **Learning**.

- **Concise** — short, direct answers.
- **Explanatory** — in-depth, educational.
- **Formal** — polished, professional.
- **Learning** — guides you with steps/questions rather than just answering.

**Custom styles:** historically built from a writing sample or a prompt description via the `+` / "Use style" flow (e.g. paste three of your own emails to make Claude write like you).

> **Migration note (Styles → Skills):** Per the official help article, **the Styles menu is being removed from Claude entirely**. Existing **custom styles are converted to Skills automatically** (they appear as disabled Skills you can enable). The default styles **Concise, Explanatory, and Formal will no longer be available after the migration**; the **Learning** style **is preserved as a Skill**. Anthropic's suggested alternative to a retired style is to **describe the same response behavior in your instructions** (e.g. project or profile instructions). During the transition your existing styles remain available alongside Skills. See [../capabilities/skills.md](../capabilities/skills.md). **WARN: verify** exactly which presets remain on your account at any given time.

### Voice mode (mobile)

**Voice mode** is a full two-way spoken conversation (you speak, Claude speaks back) — distinct from **dictation** (speech-to-text that just fills the composer).

- **Availability:** a **beta** feature available on **all plans** (Free included) on **Claude.ai (web)** and **Claude Mobile (iOS/Android)**. It launched Pro-only (May 2025) and became free for everyone in early 2026.
- **Usage:** voice conversations **count toward your regular plan usage limits**. (Third-party reports cite roughly 20–30 voice conversations/day on Free — **WARN: verify**, not stated in official docs.)
- **Voices:** Claude offers **several voice options**; the official help article does not name them. Third-party reporting lists **Buttery, Airy, Mellow, Glassy, Rounded** — **WARN: verify** current names/count against the in-app picker.
- You can seamlessly switch between text and voice within the same conversation.
- **Desktop voice mode** is not explicitly documented in the help center as of mid-2026. **WARN: verify** desktop/web availability.

### Learning mode

Surfaced as the **Learning** style/skill — Claude scaffolds understanding (asks guiding questions, builds up concepts) instead of handing over the final answer. Useful for studying or onboarding to a new topic. Note: Learning is being **preserved as a Skill** in the Styles→Skills migration.

---

## Memory

Claude can carry context **across chats** so a new conversation already knows your background and ongoing work.

- **Memory generation** — Claude builds memory from your chat history so new chats start informed instead of blank. **Available to all users** (Free, Pro, Max, Team, Enterprise) on web, Claude Desktop, and Claude Mobile.
- **Search and reference chats** — Claude can search and pull from your prior conversations (RAG-based; appears as tool calls during a chat). Scope is your chats outside projects, plus each Project's conversations separately.
- **Project memory** — within a Project, instructions and knowledge persist for every chat in that Project. See [../capabilities/projects.md](../capabilities/projects.md) and the [Memory section](#memory) above.
- **Controls** — managed under **Settings → Capabilities**:
  - **"Search and reference chats"** toggle — whether Claude can search your history.
  - **Memory** toggle with three states: **Enabled** (default), **Pause memory** (keep existing memory, stop creating new), and **Reset memory** (permanently delete all memories).
- **Incognito mode** — click the **ghost icon** in the upper-right when starting a new chat (outside projects) to open a temporary conversation that **isn't saved to your chat history** (and is not used for training).

```
[New chat, days later]
You: pick up where we left off on the pricing deck
Claude: Last week we drafted a 3-tier pricing slide for "Northstar"…
```

---

## Artifacts panel

When Claude generates something self-contained — code, a document, a diagram/chart, an interactive app — it can render it in the **Artifacts** side panel beside the chat, where you can view, edit, iterate, and (on supported plans) publish or share it. Claude can also create **interactive charts, diagrams, and visualizations in-line** in the message thread (shipped **March 12, 2026**), and these **interactive apps reached mobile on March 25, 2026**. Full details: [../capabilities/artifacts.md](../capabilities/artifacts.md).

### Artifacts panel — concrete controls

The buttons users actually click on an open artifact (web/desktop):

| Control | Where | What it does |
|---|---|---|
| **Open / expand** | Click the artifact tile, or the expand control | Opens the side panel; expand to a wider/full-screen view. **WARN: verify** the exact full-screen icon/label. |
| **Copy** | Lower-right of the artifact window | Copies the artifact's content to the clipboard. |
| **Download** | Lower-right of the artifact window | Saves the artifact as a file to use outside the conversation. |
| **View underlying code** | Artifact window | Shows the source/code behind a rendered doc, app, or visualization. |
| **Edit with Claude** | Highlight text (Markdown docs) → **Edit with Claude** | Make a targeted change by selecting text and describing the edit. |
| **Try fixing with Claude** | Appears near an error in an interactive artifact | One-click hand-off to Claude to repair a runtime error. |
| **Version selector** | Artifact header | Switch between versions of the artifact; each **Publish** also becomes a selectable version. |
| **Publish** | Artifact → **Publish** | Generates a **public link**; anyone with the link can view/interact until you unpublish. Adds it to your **Artifacts** gallery. Logged-out viewers can use it but not edit (AI-powered artifacts require sign-in). |
| **Unpublish** | After publishing → **Unpublish** | Revokes the public link. Note: once unpublished, **that same artifact cannot be re-published**. |
| **Get embed code** | After publishing | Opens a modal with auto-generated embed code to paste into another website. |
| **Share & copy link** *(Team/Enterprise)* | **Share** → **Share & copy link** | Org accounts share **within the organization only** — artifacts on Team/Enterprise **cannot be published publicly**. |
| **Remix / make a copy** | On a published artifact (viewer side) | A signed-in Claude user can create their **own editable copy** to iterate on; your original stays unchanged. |

> Team/Enterprise sharing is **org-scoped**: members can **Share** an artifact internally but **cannot Publish** it to a public link. Personal plans (Free/Pro/Max) use **Publish → public link / embed**. See [../capabilities/artifacts.md](../capabilities/artifacts.md).

---

## Conversation management

The left sidebar and chat header give you the tools to organize history.

| Capability | Where | Notes |
|---|---|---|
| **History** | Sidebar list of recent chats | Cloud-synced across devices. |
| **Search** | `Cmd/Ctrl+K` or the search box | Find chats by content/title; powered by chat search (can be disabled in Settings → Capabilities). |
| **Star / favorite** | Chat menu (⋯) | Pin important chats. **WARN: verify** exact label ("Star" vs "Favorite"). |
| **Rename** | Chat menu (⋯) → Rename | Auto-titles can be overwritten. |
| **Delete** | Chat menu (⋯) → Delete | Removes the conversation. |
| **Archive** | Chat menu (⋯) | Hide from the main list without deleting. **WARN: verify** presence/label on your plan. |
| **Share link** | Chat menu (⋯) → Share | Creates a read-only public link to a snapshot of the conversation. |
| **Add to Project** | Chat menu (⋯) | Move/attach a chat into a Project. |
| **Continue on another device** | Implicit | Open the same chat from history on any signed-in device. |

---

## Settings & account

Open Settings from the **profile icon (top-right) → Settings** on web/desktop, or the account/profile area on mobile. The tree below reflects mid-2026; **labels marked WARN: verify may differ slightly by plan/region/app.**

### Profile / Account
- Name, email, and (for org plans) workspace membership.
- **Plan & billing** — current plan, upgrade/downgrade, billing portal.
- **Export data** — request an export of your account data/conversations.
- **Delete account** — permanent account deletion.

### Appearance
- **Theme** — Light / Dark / System.
- **Language** — interface language. **WARN: verify** which languages are exposed in the UI vs. just response language.
- Font/density and "what's new" options may appear here depending on app.

### Capabilities
- **Search and reference chats** — toggle whether Claude can search your history.
- **Memory** — Enabled / Pause memory / Reset memory (cross-chat memory).
- **Web search** — for personal plans this is a per-chat toggle; for Team/Enterprise the workspace-level enablement lives under **Admin settings → Capabilities**.

### Models / behavior
- **Default model** picker and **effort/thinking** defaults (also reachable per-chat). See [../models/model-families.md](../models/model-families.md).
- Default **Style** selection (being migrated to **Skills** — see [Styles](#styles-response-toneformat)).

### Connectors
- **Manage connectors** — connect/disconnect Google Workspace, Slack, Microsoft 365/Outlook, remote MCP connectors, and desktop extensions; manage OAuth and per-connector permissions. See [../capabilities/connectors.md](../capabilities/connectors.md) and [../capabilities/mcp.md](../capabilities/mcp.md).

### Privacy / Data controls
- **Help Improve Claude** — the model-improvement (training) toggle, found under **Settings → Privacy** (`claude.ai/settings/data-privacy-controls`; on mobile: Settings → Privacy). Toggle it off to exclude your data from training. Note: even when off, conversations flagged by safety classifiers **may still be used** to improve trust-and-safety models and for safety research.
- **Location metadata** — let Claude estimate rough (city/region) location for local results; can be switched off. **WARN: verify** exact label.
- **Data retention** — retention preferences (richer controls on Enterprise: custom retention, audit logs).
- **Incognito** — start chats that aren't saved or used for training (ghost icon on a new chat).

### Beta / Labs features
- Opt into experimental features (e.g. **Claude Design** shipped under Anthropic Labs on **April 17, 2026**; computer-use research previews). **WARN: verify** which toggles are present on your account.

### Notifications
- Email/product notifications, and (desktop/mobile) push notifications for long-running agent threads/tasks. **WARN: verify** granularity.

### Security
- **Multi-factor authentication (MFA)** — enable 2FA on your account.
- Active sessions / sign-out-everywhere. **WARN: verify** session-management UI.
- For org plans: SSO/SAML, SCIM, JIT provisioning, and admin-managed controls live in the **admin console**, not personal Settings.

---

## Plans: Free / Pro / Max / Team / Enterprise

Pricing and limits below are as published in 2026; **verify current numbers at [claude.com/pricing](https://www.claude.com/pricing)** before quoting.

| Plan | Price (2026) | Headline limits | Unlocks |
|---|---|---|---|
| **Free** | $0 | Lower usage; standard access | Chat (web/desktop/mobile), web search, **Memory**, file creation, code execution, desktop extensions, Slack/Google Workspace + remote MCP connectors, extended thinking, voice mode (beta). *No Cowork, no Claude Code, no Research.* |
| **Pro** | **$20/mo** (or **$17/mo billed annually — $200 up front**) | Higher usage than Free; 5-hour caps doubled May 6, 2026 | Everything in Free **plus** Claude Code, **Cowork**, **Claude Design**, **unlimited Projects**, **Research**, more models (incl. Opus), Microsoft 365/Outlook integration. |
| **Max 5×** | **$100/mo** | ~5× Pro usage per session; higher output limits; priority access during peaks | Everything in Pro **plus** 5× usage, priority access, early features. |
| **Max 20×** | **$200/mo** | ~20× Pro usage per session; highest individual limits; priority access | Everything in Pro **plus** 20× usage, highest individual limits, earliest features. |
| **Team** | **Standard:** $25/seat/mo ($20 annual). **Premium:** $125/seat/mo ($100 annual; ~5× Standard usage) | Min **5 seats**, up to **150 seats** | Pro features for teams **plus** SSO + Domain Capture, JIT provisioning, role-based permissioning, central billing/admin tools, spend controls (org + per-user), enterprise search connectors (Drive/Gmail/Calendar/GitHub/M365/Slack), mixed standard/premium seats. |
| **Enterprise** | Self-serve **$20/seat** + usage that scales with model/task; or custom (contact sales) | Custom; granular spend controls | Team **plus** custom roles, SCIM, audit logs, custom data retention, expanded security/compliance controls. (Contact sales for HIPAA, IP allowlisting, etc. — **WARN: verify** specific features.) |

> **Pricing-page caveat:** As of this writing the public pricing page has a likely **typo listing Max 20× as "From $100/mo."** The authoritative **What is the Max plan?** help article states **Max 5× = $100/mo and Max 20× = $200/mo** — use those figures. Mobile (in-app purchase) pricing may differ from web pricing.

**Usage & reset windows:**
- Most usage runs on a rolling **5-hour window** plus, for Pro/Max/Team, **weekly caps** (an overall cap plus a separate model-specific cap). Weekly limits reset on a fixed day assigned to your account.
- The **5-hour limits were doubled on May 6, 2026** for Pro, Max, Team, and seat-based Enterprise plans, and prior **peak-hour reductions were removed**. This was tied to Anthropic's compute expansion (incl. a SpaceX/Colossus deal). **Weekly caps were not changed** by that announcement.
- Hitting a limit pauses new messages until the window resets; Claude shows a countdown. **WARN: verify** exact per-plan message counts — official docs intentionally avoid fixed numbers because they fluctuate with model and demand.

See [../models/model-families.md](../models/model-families.md) for which models each plan can select.

---

## Keyboard shortcuts (web & desktop)

| Action | macOS | Windows/Linux |
|---|---|---|
| New chat | `Cmd+O` | `Ctrl+O` |
| Search / command menu | `Cmd+K` | `Ctrl+K` |
| Toggle sidebar | `Cmd+Shift+S` | `Ctrl+Shift+S` |
| `/` command menu | `/` at start of composer | `/` |
| New line in composer | `Shift+Enter` | `Shift+Enter` |
| Show shortcut help | `Cmd+?` | `Ctrl+?` |
| **Desktop — Quick Entry (global launcher)** | **double-tap `Option`** (customizable to `Option+Space` or a custom shortcut) | **Not available** — Quick Entry is macOS-only |
| **Desktop — voice dictation** | `Caps Lock` toggle — **macOS only, disabled by default** (it takes over Caps Lock; requires macOS 14+) | — |

> **Quick Entry** (macOS desktop) opens a text box from any app to start a new chat, capture a screenshot, share a window, or dictate — without switching windows. Configure it under **Settings → General** (Desktop app), where you can change the launcher shortcut and enable/disable the Caps Lock dictation shortcut. **Quick Entry and the Caps Lock dictation shortcut are macOS-only**; on Windows, pin Claude to the taskbar or assign a shortcut to the desktop icon. Some chords vary by build — use the in-app `Cmd/Ctrl+?` help to confirm.

---

## Mobile-specific features

- **Voice mode** — beta, free for all plans, hands-free two-way conversations (iOS/Android) with selectable voices; counts toward plan usage.
- **Camera capture** — snap a photo and ask about it inline.
- **Share-sheet / share-target import** — send text, images, or files from other apps into Claude.
- **Siri Shortcuts** (iOS) — launch Claude actions by voice.
- **Reminders integration** (iOS) — **WARN: verify** current scope.
- **Interactive apps** — Artifacts-style interactive outputs (live charts/diagrams) render on mobile (since **March 25, 2026**).
- **Apple Health analysis** (iOS) — analyze Health/fitness data with permission; reported US-based **Pro/Max** (Jan 2026). **WARN: verify** regional rollout.
- **Persistent agent threads** — monitor/steer Cowork tasks from mobile (Pro/Max).

---

## Related pages

- [./cowork.md](./cowork.md) — the desktop agent (Cowork) that does multi-step work in a local VM.
- [./claude-design.md](./claude-design.md) — prompt-to-prototype visual workspace (Anthropic Labs).
- [./claude-code.md](./claude-code.md) — Claude Code overview (CLI/IDE/desktop/web).
- [../models/model-families.md](../models/model-families.md) — every model, ID, price, and context window.
- [../models/capabilities-and-modes.md](../models/capabilities-and-modes.md) — extended thinking, 1M context, vision, tools.
- [../capabilities/artifacts.md](../capabilities/artifacts.md) — the Artifacts panel in depth.
- [../capabilities/projects.md](../capabilities/projects.md) — Projects: knowledge base, instructions, memory.
- [Memory (on this page)](#memory) — cross-chat and project memory controls.
- [../capabilities/skills.md](../capabilities/skills.md) — Skills (where Styles are migrating).
- [../capabilities/connectors.md](../capabilities/connectors.md) — connectors catalog, OAuth, admin.
- [../capabilities/mcp.md](../capabilities/mcp.md) — Model Context Protocol for custom connectors.

## Open questions / to verify

- Exact label of any standalone in-app **Agent Mode** toggle vs. the Cowork surface, and whether such a switch exists in the chat composer on all surfaces.
- Current **voice option names/count** (Buttery/Airy/Mellow/Glassy/Rounded reported by third parties but not named in the official help article) and whether **desktop/web voice mode** is officially supported.
- Precise **file upload caps** (size per file, count per chat) and supported file types as of mid-2026 (the ~30 MB figure is unconfirmed).
- Exact **5-hour and weekly message counts** per plan post-May-2026 (official docs avoid fixed numbers).
- Whether **Star/Favorite** and **Archive** appear identically across plans, and the exact menu labels.
- Whether **Team content is excluded from model training by default** (not confirmed in the current Team help article).
- Scope of iOS **Reminders** integration and current **Apple Health** regional availability.
- The exact in-app label/path for **location metadata** and any session-management UI under Security.
- Exact **sidebar labels and ordering** — "Starred" vs "Favorites", whether **Projects** and **Artifacts** are fixed top-level items or hover-revealed, and the precise form of the desktop **Chat/Cowork/Code** switcher (tabs vs dropdown vs rail icons).
- Whether **`Esc` stops generation** in the apps (confirmed in Claude Code's interactive mode; app binding unconfirmed) and the exact **"switch model and retry"** affordance/label per surface.
- The artifact **full-screen/expand** control's exact icon/label, and confirmation of **download/copy** placement (currently "lower-right of the artifact window" per help docs).
- Current **Free Projects cap** (cited as 5) and whether the **Projects** sidebar entry behaves identically across plans.

## Sources

- [Get started with Claude — Help Center](https://support.claude.com/en/articles/8114491-get-started-with-claude)
- [Release notes — Help Center](https://support.claude.com/en/articles/12138966-release-notes)
- [What is the Max plan? — Help Center](https://support.claude.com/en/articles/11049741-what-is-the-max-plan)
- [What is the Team plan? — Help Center](https://support.claude.com/en/articles/9266767-what-is-the-team-plan)
- [When should I use web search, extended thinking, and research? — Help Center](https://support.claude.com/en/articles/11095361-when-should-i-use-web-search-extended-thinking-and-research)
- [Use research on Claude — Help Center](https://support.claude.com/en/articles/11088861-use-research-on-claude)
- [Enable and use web search — Help Center](https://support.claude.com/en/articles/10684626-enable-and-use-web-search)
- [Use Claude's chat search and memory — Help Center](https://support.claude.com/en/articles/11817273-use-claude-s-chat-search-and-memory-to-build-on-previous-context)
- [Change the model, effort, and thinking settings — Help Center](https://support.claude.com/en/articles/8664678-change-the-model-effort-and-thinking-settings)
- [How do usage and length limits work? — Help Center](https://support.claude.com/en/articles/11647753-how-do-usage-and-length-limits-work)
- [Styles are moving to skills — Help Center](https://support.claude.com/en/articles/10181068-styles-are-moving-to-skills)
- [Use voice mode — Help Center](https://support.claude.com/en/articles/11101966-use-voice-mode)
- [Get started with Claude in Chrome — Help Center](https://support.claude.com/en/articles/12012173-get-started-with-claude-in-chrome)
- [Use quick entry with Claude Desktop on Mac — Help Center](https://support.claude.com/en/articles/12626668-use-quick-entry-with-claude-desktop-on-mac)
- [How do I change my model improvement privacy settings? — Privacy Center](https://privacy.claude.com/en/articles/12109829-how-do-i-change-my-model-improvement-privacy-settings)
- [Higher usage limits for Claude (and a SpaceX compute deal) — Anthropic](https://www.anthropic.com/news/higher-limits-spacex)
- [Claude pricing](https://www.claude.com/pricing)
- [Publish and share artifacts — Help Center](https://support.claude.com/en/articles/9547008-publish-and-share-artifacts)
- [What are artifacts and how do I use them? — Help Center](https://support.claude.com/en/articles/9487310-what-are-artifacts-and-how-do-i-use-them)
- [How can I create and manage projects? — Help Center](https://support.claude.com/en/articles/9519177-how-can-i-create-and-manage-projects)
- [Navigating the Claude desktop app: Chat, Cowork, Code — Claude](https://claude.com/resources/tutorials/navigating-the-claude-desktop-app)
</content>
</invoke>
