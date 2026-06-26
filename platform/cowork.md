---
type: Surface
title: Claude Cowork
description: Anthropic's agentic mode in the Claude Desktop app that reads, edits, and creates real local files and runs multi-step knowledge-work tasks end to end.
domain: platform
tags: [cowork, desktop, agent, knowledge-work, scheduled-tasks, mcp, enterprise, rbac, opentelemetry, dispatch]
related: [claude-ai, claude-code-overview, dispatch-remote, connectors, mcp, skills]
resource: https://www.anthropic.com/product/claude-cowork
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://www.anthropic.com/product/claude-cowork
  - https://claude.com/product/cowork
  - https://claude.com/blog/cowork-for-enterprise
  - https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork
  - https://support.claude.com/en/articles/13364135-use-claude-cowork-safely
  - https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork
  - https://support.claude.com/en/articles/14479288-claude-cowork-desktop-architecture-overview
  - https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans
  - https://support.claude.com/en/articles/14477985-monitor-claude-cowork-activity-with-opentelemetry
  - https://support.claude.com/en/articles/12138966-release-notes
  - https://thenewstack.io/anthropic-claude-cowork-promotion/
---

# Claude Cowork

Claude Cowork is the **agentic, file-aware mode inside the Claude Desktop app** that does knowledge work for you: instead of telling you how to do a task, Claude reads, edits, and creates real files in folders you choose, moves between your local apps and connected tools, and runs a multi-step job all the way to a finished deliverable. It is the non-coding sibling of Claude Code — same "let the agent actually do the work" philosophy, aimed at researchers, analysts, ops, legal, and finance teams rather than engineers. You give it a goal (and optionally a cadence), it does the assembly, and you keep the judgment calls.

> At-a-glance
> - **What it is:** An agentic "Tasks" mode in the Claude Desktop app (sits next to **Chat**, and **Code** where Claude Code desktop is available). Anthropic's framing: *"Give it a goal and Claude works on your computer, local files, and applications to return a finished deliverable."*
> - **Where you find it:** Claude Desktop app for **macOS** and **Windows** → click the **Cowork** tab in the mode selector to switch into **Tasks** mode. (Not on web; phone/browser act only as a remote control — see Dispatch below.)
> - **Who can use it (by plan):** **All paid plans** — Pro, Max, Team, Enterprise — via the desktop app. Free has no access. It is **on by default** on Team/Enterprise, but org owners can disable it org-wide. Granular per-user/per-role enablement is **Enterprise-only** (see Enterprise section); the **Zoom** connector is available across paid tiers.
> - **Status:** **Generally available** since **April 9, 2026** (after a research preview that began **Jan 12, 2026** for Max on macOS). Remote dispatch from phone/browser shipped as a research preview in **March 2026** for Pro/Max.

---

## What Cowork is (and how it differs from Chat and Code)

The Claude Desktop app exposes (up to) three modes from one mode selector:

| Mode | What it does | Typical user |
|------|--------------|--------------|
| **Chat** | Standard conversational Claude — answers, drafts, reasons, calls connectors, but does not autonomously drive your local files/apps to completion. | Everyone |
| **Cowork** ("Tasks") | Agentic knowledge work: reads/edits/creates **real local files** in folders you grant, runs multi-step jobs end to end, can be scheduled on a cadence. | Researchers, analysts, ops, legal, finance |
| **Code** | Claude Code's agentic coding loop (CLI/IDE/desktop). Appears in the selector where Claude Code desktop is available. | Developers — see [./claude-code.md](./claude-code.md) |

Anthropic's framing on claude.com: *"Unlike Chat, Cowork lets Claude complete work on its own. Describe the outcome and cadence, and it takes action and keeps you informed."* The marketing headline is **"Delegate to Claude, delight in the result"** — *"Hand off a task, get a polished deliverable."* Cowork is positioned as **Claude Code power for knowledge work** — the same agent harness pointed at documents and spreadsheets instead of repositories.

Key design principle — **human oversight stays on**: *"It completes tasks, but consequential decisions remain with the user."* Cowork does the assembly; you approve the consequential moves.

### Headline capabilities (from claude.com)
- **Schedule tasks** — define a goal + cadence once, let it run (e.g., *"check your email every morning, pull metrics, or run your weekly Slack digest"*).
- **Organize files** — rename, sort, deduplicate, surface what's relevant in a folder.
- **Build spreadsheets** — e.g., extract data from screenshots/PDFs into a formatted sheet.
- **Prepare reports** — assemble scattered source docs into a structured draft.
- **Analyze notes** — synthesize across meeting notes, transcripts, and documents.

---

## Timeline

| Date (2026) | Milestone |
|-------------|-----------|
| **Jan 12** | Research preview launches — initially **Max** subscribers on **macOS** only. *(Release notes: "Cowork research preview on Claude Desktop (macOS only) for Max plans.")* |
| **Jan 16** | Preview access expanded to **Pro** (still macOS only). |
| **Feb 25** | **Scheduled tasks** ship — "ability to create and schedule both recurring and on-demand tasks in Cowork." |
| **Mar 17** | **Persistent task thread** / remote management research preview for Pro/Max — manage Cowork tasks from Claude Desktop or the iOS/Android apps. |
| **Mar 23** | **Computer-use Dispatch** — Claude can pick up and manage tasks while you're away from the desktop. |
| **Apr 9** | **General availability** across **all paid plans** on **both macOS and Windows**, alongside the enterprise management features and the Zoom connector. Windows arrived at GA — it was *not* a separate earlier release. |
| **Jun 5 – Jul 5** | Limited-time promotion: **5-hour usage limit in Cowork doubled (2×)** for Pro/Max/Team (and legacy seat-based Enterprise) at no charge. |
| **Jun 25** | Team/Enterprise admins can require members to **verify their device** before viewing or steering local sessions remotely. |

---

## Platforms and requirements

- **macOS** Claude Desktop app and **Windows** Claude Desktop app (latest version required for full feature set). Windows reached parity at **GA on Apr 9, 2026** — the macOS-only window covered the Jan–Apr research preview.
- **Not** available on the web app. The phone and browser are **remote controls only** — the actual work runs on your computer.
- **The Claude Desktop app must remain open while Claude is working.** Scheduled tasks *"only run while your computer is awake and the Claude Desktop app is open."* (Closing the *mobile* app is fine — that's just the remote; the desktop is what executes.)
- **Conversation history is stored locally** on the desktop machine. On Team/Enterprise it *"cannot be centrally managed or exported by admins"* — an important data-governance caveat (see Enterprise section).

---

## How it works: pointing Cowork at folders

Cowork can only touch what you explicitly give it. Two layers of context/control:

1. **Global instructions** — persistent guidance that applies to every Cowork task.
   - Path: **Settings → Cowork**, then click **Edit** next to *Global instructions*, type your instructions in the text box, and click **Save**.
   - Example: house style, "always save outputs to `~/Deliverables`, never overwrite originals."
2. **Folder instructions** — per-folder project context attached when you add a local folder to a task. Claude can **update these notes during a session** so it carries learnings forward.

To start a task:
1. Open Claude Desktop → click the **Cowork** tab in the mode selector to switch to **Tasks** mode.
2. Describe the task in natural language; optionally add the folder(s) it should work in.
3. **Review Claude's approach** (its plan) before it proceeds.
4. Approve actions per your permission mode (below), then let it run to completion.

### Example: kickoff prompt

```text
Work in ~/Clients/Acme/Q2-contracts.
For each PDF, extract: counterparty, effective date, term length,
auto-renewal (yes/no), and termination notice period.
Build Acme-Q2-summary.xlsx with one row per contract.
Flag any contract missing a termination clause in a "Review" column.
Do not modify the source PDFs.
```

### Folder grant mechanics — scope, persistence, revocation

This is load-bearing because Cowork writes to the **real local filesystem in place** — there is no copy/sandbox layer for file edits (see *Recovery & undo* below). The official model:

- **What it can touch.** *"Claude can only read and write files in folders you've connected, and network access follows the egress settings you've configured."* The boundary is enforced in software: *"Access is gated by an application-layer permission system that enforces the member's connected-folder rules and your organization's network egress settings."* (architecture overview). Folder reads/writes run **natively on your device** — they are *not* sandboxed; only **shell commands and code Claude writes** run in an isolated Linux VM. So a connected folder = real files on your disk.
- **Scope guidance.** Anthropic recommends scoping narrowly: *"Consider creating a dedicated working folder for Claude rather than granting broad access, and keep backups of important files."* *"You control which local files Claude can access."* `WARN: verify` whether a granted folder automatically includes its **subfolders** and whether multiple folders can be connected to one task — the Help Center recommends a "dedicated working folder" but does not enumerate subfolder recursion or a multi-folder limit.
- **The add-folder UI.** Folders are "connected" / "selected" per task, and **Folder instructions** attach project context at that moment: *"Folder instructions add project-specific context to Cowork when you select a local folder. Claude can also update these on its own during a session."* `WARN: verify` the exact picker flow — the Help Center references "select a local folder" and a folder-connection model but does **not** document the literal "Add folder" button/picker steps or where it lives in the task composer.
- **Per-task vs. persistent.** `WARN: verify` — official docs describe folders as "connected" (implying a persistent connection you reuse), but **do not state** whether a grant is scoped to a single task or persists across sessions until removed. Safest assumption: treat a connected folder as **persistent until you remove it**, and re-check the connection before sensitive runs.
- **Revoking a folder.** `WARN: verify` — there is **no documented revoke/disconnect flow** in the current Help Center. The safe pattern is to **remove/disconnect the folder from the task (or in Cowork settings) and rely on OS-level folder permissions** as a backstop; do not assume a specific in-app "remove folder" control exists until confirmed.

---

## Permission / approval model

Cowork's whole value is that it acts on real files, so the approval model is central. In **Settings → Cowork** you choose how Claude handles approvals:

| Mode | Behavior | When to use |
|------|----------|-------------|
| **Ask before acting** (recommended default) | Claude **pauses for your approval on each action** (each edit, create, move, tool call). | First runs, unfamiliar folders, anything irreversible. |
| **Act without asking** | Claude proceeds **without pausing** between steps. | Trusted, well-scoped, repeatable jobs. |

Guardrails that apply in **both** modes:
- **Claude always asks before permanently deleting files** — deletion is never silent, regardless of mode. *"You will see a permission prompt and will need to select 'Allow'."*
- **Web search, web fetch, and MCP connectors** are treated as network egress and follow their own permission/connector authorization (granted via **Settings → Connectors**), separate from local-file approvals.
- Whether each AI-initiated action was **approved manually, rejected, or initiated automatically** is recorded in the OpenTelemetry event stream on Team/Enterprise (see Enterprise section).

> Practical pattern: start a new folder in **Ask before acting**, watch a few approvals to confirm Claude's behavior, then flip to **Act without asking** for the bulk run — but keep delete-confirmation as your backstop.

### Approve / reject / stop affordances in the task UI

The documented controls for steering a running task:

| Control | What it does | Source wording |
|---------|--------------|----------------|
| **Approve each action** | In *Ask before acting*, Claude halts at each step for your go-ahead. | *"Ask before acting: Claude pauses so you can approve each action."* |
| **"Allow" on delete prompt** | The one prompt shown in **both** modes; you must click **Allow** before any permanent deletion. | *"You'll see a permission prompt and must select 'Allow' before Claude can perform deletion tasks."* |
| **App-access prompt + blocklist** (computer use) | For computer/app control, Claude prompts per application; blocked apps are auto-denied. | *"Claude asks for your permission before accessing each application… Any requests from Claude to use blocked applications will be automatically denied."* |
| **Steer mid-task** | Type into the thread to course-correct without stopping. | *"Steering: You can jump in to course-correct or provide additional direction mid-task."* |
| **Stop the task** | Halt a run outright. | *"If something feels off, stop the task immediately."* / *"You can stop Claude at any point."* |
| **Pause / delete a task** | Pause an idle or scheduled task; delete via the **⋮** menu or trash icon. | *"Pause tasks you're not actively using."* / *"…delete a task at any time using the 'Delete' option… click '⋮' next to the task, or select tasks from your Tasks list and click the trash icon."* |

`WARN: verify` two specifics the docs do **not** name: (a) the exact label of the **deny/reject** button on a per-action approval (only **"Allow"** is named verbatim, on the delete prompt — there is no documented "Reject"/"Deny" wording), and (b) **keyboard shortcuts** — the Help Center documents **no keyboard shortcuts** for approve/deny/pause/stop. Do not assume any exist until confirmed.

---

## Recovery, undo & checkpoints

Because Cowork **edits and creates real files in place on your disk** — *"Delivers finished outputs directly to your file system"* — recovery deserves its own treatment. The honest summary: **there is no documented undo, rewind, checkpoint, snapshot, or version-history feature in Cowork, and no "ran in a copy/workspace" model for file edits.** Plan recovery on the assumption that file changes are immediate and durable.

- **No copy/sandbox for files.** File reads and writes run **natively on the host filesystem**, not in an isolated workspace: *"The agent loop runs natively on the device. This includes Claude's conversation handling, file reads and writes in connected folders…"* Only **shell commands and code Claude writes** run in *"an isolated virtual machine (VM)… isolated from the host operating system."* So edits to your documents/spreadsheets are real and not staged in a copy.
- **The only built-in safety net is the delete prompt.** *"Cowork requires your explicit permission before permanently deleting any files."* That gates *deletion*, not *overwrites* — an in-place edit or overwrite in **Act without asking** is not separately confirmed.
- **Official recovery guidance is "keep your own backups."** *"Consider creating a dedicated working folder for Claude rather than granting broad access, and keep backups of important files."* Anthropic positions user-side backup — not an in-app revert — as the recovery path.
- **Don't conflate with Claude Code.** Claude Code (the "Code" mode) has its own checkpoint/rewind tooling; **that does not apply to Cowork.** `WARN: verify` — if a Cowork-native checkpoint/version-history feature ships later, update this section; as of the current Help Center it is **not** present.

**Practical recovery playbook (since the app provides none):**

| Situation | Mitigation (user-provided, not Cowork features) |
|-----------|-------------------------------------------------|
| Risk of bad overwrites | Run in **Ask before acting**; instruct Claude *"never overwrite originals — write outputs to a new file"* (and set this in **Global instructions**). |
| Need point-in-time rollback | Keep the working folder under **version control (git)** or OS snapshotting — **Time Machine** (macOS) / **File History** or a **VSS restore point** (Windows). `WARN: verify` (OS-level, outside Cowork). |
| Accidental edits to sources | Keep source files **read-only / in a separate folder you don't connect**; tell Claude *"do not modify the source files."* |
| Auditing what changed (Team/Enterprise) | **File access** events (paths read/modified) are in the **OpenTelemetry** stream — useful for *forensics after the fact*, not for rollback (see Enterprise section). |

> `WARN: verify` — none of the official Cowork articles describe an undo, "restore previous version," or checkpoint control. This section states the **safest known behavior** (in-place, durable edits; back up yourself); treat any future in-app revert as unconfirmed until documented.

## Scheduled tasks (cadence)

Cowork can run a task **on-demand or on a recurring cadence you define once** — this is the "set it and forget it" half of the product (shipped **Feb 25, 2026**).

- **Create a schedule** by typing **`/schedule`** inside any Cowork task, or via **Scheduled** in the **left sidebar**.
- Tasks can run **on-demand** or **automatically on a cadence** (e.g., every weekday morning, every Friday at 5pm). `WARN: verify` the exact cadence options/granularity (daily/weekly presets vs. cron-style) in the current UI — the Help Center confirms "recurring and on-demand" scheduling but does not enumerate the picker.
- **Hard requirement:** *"Scheduled tasks only run while your computer is awake and the Claude Desktop app is open."* If your machine is asleep at the scheduled time, the run is skipped/deferred.

### Example scheduled workflows

| Cadence | Task |
|---------|------|
| Weekday 8:00am | "Check my email for anything from a client or my manager, summarize what needs a reply, and draft replies in my Drafts for me to review." |
| Daily 9:00am | "Pull yesterday's key metrics from the dashboard export in `~/Reports/raw`, append to `metrics-master.xlsx`, and flag any KPI down >10% WoW." |
| Friday 4:30pm | "Read this week's meeting notes in `~/Notes/2026-wXX`, compile a Slack-ready weekly digest of decisions, owners, and open items." |

For scheduling/automation that runs **in the cloud without your machine on**, see Routines in [../claude-code/dispatch-remote-routines.md](../claude-code/dispatch-remote-routines.md) — Cowork's scheduler is desktop-bound by contrast.

---

## Connectors and MCP integration

Cowork is not limited to local files — it reaches your other tools through **connectors** and **Model Context Protocol (MCP)** servers, authorized in **Settings → Connectors**.

- Standard Claude connectors (e.g., Google Drive, Gmail, Google Calendar, Slack-style integrations) flow into Cowork tasks so it can read source material and post/draft outputs.
- **Custom/remote MCP servers** extend Cowork with arbitrary tools and data — see [../capabilities/mcp.md](../capabilities/mcp.md) and [../capabilities/connectors.md](../capabilities/connectors.md).
- **Zoom connector** (added at GA, available on paid plans): a Zoom-built connector that brings **meeting intelligence** into Cowork — *"AI Companion meeting summaries and action items alongside transcripts and smart recordings"* — so teams can turn conversations into agentic workflows. Admins can restrict it org-wide. Example: *"Take the action items from my 10am Zoom and add each as a row in `~/Projects/launch/tasks.xlsx` with owner and due date."*

```text
# Example combining a connector + local files in one Cowork task
Read the transcript of today's "Acme Renewal" Zoom call (Zoom connector),
extract every commitment we made with a date,
then update ~/Clients/Acme/commitments.md and create calendar holds for each.
```

---

## Dispatch: start and steer from phone or browser

Cowork's heavy lifting always runs **on your desktop**, but you can **dispatch and monitor** it remotely. Anthropic's wording: *"Message Claude from anywhere, come back when it's done"* and *"Send a task to Claude from your phone. It picks up where you left off."*

- The phone/browser is a **thin control surface**; the desktop is the compute host. *"Work continues in the background, even when you close the [mobile] app."*
- Rollout history: a **persistent task thread** for managing Cowork tasks from Claude Desktop or the iOS/Android apps shipped as a **research preview for Pro/Max on Mar 17, 2026**; **computer-use Dispatch** (Claude managing tasks while you're away) followed **Mar 23, 2026**. So remote steer is live for Pro/Max — `WARN: verify` whether it has graduated from research preview to GA for all plans.
- **Security:** since **Jun 25, 2026**, Team/Enterprise admins can **require device verification** before a member can view or steer local sessions remotely.

So the model is: **desktop = engine, phone/browser = remote control.** Launch a task on the train, nudge it, and review the result later — all while the desktop machine does the work.

For the broader remote-control / Dispatch / Agent View story (shared with Claude Code), see [../claude-code/dispatch-remote-routines.md](../claude-code/dispatch-remote-routines.md).

---

## Usage limits

Cowork is compute-intensive — it *"consumes more of your usage allocation than chatting."* Limits operate on the same rolling-window model as the rest of Claude:

- **5-hour rolling window** governs short-term usage; **weekly limits** sit on top.
- **Promotion (Jun 5 – Jul 5, 2026):** the **5-hour usage limit in Cowork is doubled (2×)** at **no extra cost** and **no action required** — applied automatically.
  - **Eligible:** **Pro, Max, Team**, and **legacy seat-based Enterprise**. **Not** eligible: Free plans and **consumption-based Enterprise seats**.
  - **Scope:** the 2× applies **only to Cowork**. Limits for Chat on web/desktop/mobile and for Claude Code are unchanged.
  - **Weekly limits are unchanged** by the promo.
  - After **July 5, 2026**, Cowork 5-hour limits **return to standard** automatically; no billing/plan change.
- **Manage/monitor usage:** **Settings → Usage**.

Tips to stay within limits: **batch related work into one session**, use plain **Chat** for simple asks, and reserve Cowork for true multi-step jobs. `WARN: verify` the exact numeric 5-hour/weekly caps per plan — Anthropic does not publish fixed token counts and they vary by plan and demand. See [./claude-ai.md](./claude-ai.md) for plan-level limit details.

---

## Enterprise & admin features (GA, April 9 2026)

At GA, Cowork shipped a set of **organization controls** so teams can deploy it company-wide. Administer these from the org **admin console** (and Analytics/Admin APIs where noted).

| Feature | What it does | Plans |
|---------|--------------|-------|
| **Org-wide enable/disable** | Cowork (and plugins) is **on by default**; owners/primary owners toggle it under **Organization settings → Capabilities**. On **Team** this is **all-or-nothing** — "granular controls by user or role are not currently available." | Team, Enterprise |
| **RBAC (groups + custom roles)** | Admins organize users into **groups** (manually or via **SCIM** from your IdP) and assign **custom roles** that define which capabilities members can use — so Cowork can be enabled for specific users/teams and rolled out in phases. | **Enterprise only** |
| **Group spend limits** | Per-team budgets / cost ceilings set from the admin console. | Team, Enterprise |
| **Usage analytics** | Cowork activity appears in the **admin dashboard** and the **Analytics API**: per-user sessions, **skill and connector invocations**, active users, and **DAU/WAU/MAU** across date ranges. | Team, Enterprise |
| **OpenTelemetry observability** | Cowork emits events to standard SIEM/observability pipelines (see below). | Team, Enterprise |
| **Per-connector action control** | Admins can **restrict which actions are available within each MCP connector** across the organization. | Team, Enterprise |
| **Zoom connector** | Meeting summaries, transcripts, action items, smart recordings into Cowork; admins can restrict org-wide. | Paid plans |

### OpenTelemetry observability (Team + Enterprise)

Cowork streams a rich event set to your collector. Per the Help Center, events include:

- **User prompts** — the full text of prompts submitted to Cowork.
- **Tool and MCP invocations** — every tool call, including MCP server name, tool name, parameters, success/failure, and execution time.
- **File access** — paths Claude reads, modifies, or otherwise touches.
- **Skills and plugins** — which skills/plugins Claude invokes.
- **Human approval decisions** — whether each action was approved, rejected, or initiated automatically.
- **API requests and errors** — per-request model, token counts, estimated cost, duration, and errors.

Compatible destinations called out officially: **SIEM** — Splunk, Cribl; **observability** — Honeycomb, Datadog; **log aggregation** — Elasticsearch, Loki; **columnar store** — ClickHouse.

```text
Cowork (Team/Enterprise) → OpenTelemetry exporter →
  your collector → Splunk / Cribl (SIEM) · Honeycomb / Datadog (observability)
  · Elasticsearch / Loki (logs) · ClickHouse (analytics)
```

### Audit & compliance caveat

There is **no separate Cowork audit-log surface**, and **Cowork activity is *not* captured in the Compliance API at this time.** Each Cowork OTel event carries a **shared user-account identifier** you can use to **correlate** it with Compliance API records from other surfaces. Conversation history lives **locally** and cannot be centrally managed/exported by admins — so OTel export is the primary path for centralized monitoring/compliance.

---

## Concrete example workflows

**1. Contract data extraction → spreadsheet**
```text
Folder: ~/Legal/NDAs-2026
For every PDF: pull counterparty, signature date, governing law,
and confidentiality term (years). Output NDA-register.xlsx,
one row per file. Don't touch the PDFs. Ask me before overwriting the register.
```

**2. Weekly Slack digest (scheduled)**
```text
/schedule  →  Fridays 16:30
Read ~/Notes/weekly and the #project-x Slack channel (connector).
Produce a digest: decisions, owners, blockers, next-week priorities.
Save digest-wXX.md and post it to #project-x.
```

**3. Morning email triage (scheduled, ask-before-acting)**
```text
/schedule  →  Weekdays 08:00
Scan Gmail (connector) for messages from clients/manager since 5pm yesterday.
Summarize what needs a reply and draft responses in Drafts.
Do not send anything — leave drafts for my review.
```

**4. Folder cleanup**
```text
Folder: ~/Downloads
Find duplicate and near-duplicate files, group screenshots by month into
dated subfolders, and produce a deletion-candidates.csv. Confirm before any delete.
```

**5. Report assembly from scattered sources**
```text
Folder: ~/Projects/Q2-review
Synthesize the 14 source docs + the metrics export into a 3-page
exec summary (Q2-exec-summary.docx): wins, misses, asks, Q3 plan.
Cite which source each claim came from.
```

**6. Zoom meeting → action tracker (connector + local + calendar)**
```text
From today's "Acme Renewal" Zoom (Zoom connector), pull the AI Companion
summary and action items. Append each to ~/Clients/Acme/actions.xlsx with
owner + due date, and create a Google Calendar hold for anything due this week.
```

---

## Related pages
- [./claude-ai.md](./claude-ai.md) — Claude.ai apps, settings, plans, and usage limits Cowork inherits.
- [./claude-code.md](./claude-code.md) — Claude Code (the "Code" mode); Cowork is its knowledge-work counterpart.
- [../claude-code/dispatch-remote-routines.md](../claude-code/dispatch-remote-routines.md) — Dispatch, Remote Control, Routines, Agent View (remote/cloud automation).
- [../capabilities/connectors.md](../capabilities/connectors.md) — Connectors catalog, OAuth, admin controls (including Zoom).
- [../capabilities/mcp.md](../capabilities/mcp.md) — Model Context Protocol used to extend Cowork.
- [../capabilities/skills.md](../capabilities/skills.md) — Agent Skills that Cowork can load.

## Open questions / to verify
- **Undo / version history:** whether any Cowork-native rollback, "restore previous version," or checkpoint feature exists or is planned (none documented today; edits are in-place and durable).
- **Folder grant UI & scope:** the literal "Add folder"/picker flow, whether a granted folder includes **subfolders**, whether multiple folders can attach to one task, and whether grants are **per-task or persistent across sessions**.
- **Revoke a folder:** the actual in-app disconnect/remove-folder control (not documented in the current Help Center).
- **Per-action deny label & keyboard shortcuts:** the exact wording of the reject/deny affordance (only "Allow" is documented, on the delete prompt) and whether any keyboard shortcuts exist for approve/deny/pause/stop (none documented).
- Exact **cadence/scheduling granularity** in the `/schedule` UI (daily/weekly presets vs. cron-style).
- Whether **mobile/browser Dispatch for Cowork** has graduated from research preview to GA, and whether it extends beyond Pro/Max.
- Precise **5-hour and weekly usage caps** per plan (Anthropic does not publish fixed numbers).
- Exact gating of **group spend limits** and **per-connector action control** (confirmed Team+Enterprise here; reconfirm whether either is Enterprise-only in newer docs).
- Whether **Code** mode appears in the same selector for all users or only when Claude Code desktop is installed.

## Sources
- Anthropic — Claude Cowork product page: https://www.anthropic.com/product/claude-cowork
- claude.com — Cowork ("Delegate to Claude, delight in the result"): https://claude.com/product/cowork
- Claude by Anthropic — Making Claude Cowork ready for enterprise (GA Apr 9, features): https://claude.com/blog/cowork-for-enterprise
- Claude Help Center — Get started with Claude Cowork: https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork
- Claude Help Center — Use Claude Cowork safely (backups, delete-prompt "Allow", dedicated folder, stop the task): https://support.claude.com/en/articles/13364135-use-claude-cowork-safely
- Claude Help Center — Let Claude use your computer in Cowork (per-app prompts, app blocklist, "stop Claude at any point"): https://support.claude.com/en/articles/14128542-let-claude-use-your-computer-in-cowork
- Claude Help Center — Claude Cowork desktop architecture overview (native file I/O vs. isolated VM for code; connected-folder permission enforcement): https://support.claude.com/en/articles/14479288-claude-cowork-desktop-architecture-overview
- Claude Help Center — Use Claude Cowork on Team and Enterprise plans: https://support.claude.com/en/articles/13455879-use-claude-cowork-on-team-and-enterprise-plans
- Claude Help Center — Monitor Claude Cowork activity with OpenTelemetry: https://support.claude.com/en/articles/14477985-monitor-claude-cowork-activity-with-opentelemetry
- Claude Help Center — Release notes (preview Jan 12, Pro Jan 16, scheduling Feb 25, GA Apr 9): https://support.claude.com/en/articles/12138966-release-notes
- The New Stack — Why Anthropic doubled Cowork limits: https://thenewstack.io/anthropic-claude-cowork-promotion/
