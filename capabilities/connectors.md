---
type: Extension System
title: Connectors
description: The user-facing catalog of OAuth-based integrations and Model Context Protocol servers that let Claude read data and take actions inside external apps like Google Drive, Gmail, Slack, Linear, Jira, GitHub, and Zoom.
domain: capabilities
tags: [connectors, mcp, integrations, oauth, google-workspace, desktop-extensions, mcpb, enterprise-governance, remote-mcp, custom-connectors]
related: [mcp, claude-ai, cowork, projects, skills, plugins, claude-code-overview]
resource: https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities
  - https://support.claude.com/en/articles/10166901-use-google-workspace-connectors
  - https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp
  - https://support.claude.com/en/articles/11503834-build-custom-connectors-via-remote-mcp-servers
  - https://support.claude.com/en/articles/14503689-mcp-connectors
  - https://support.claude.com/en/articles/13930458-set-up-role-based-permissions-on-enterprise-plans
  - https://support.claude.com/en/articles/15537633-authorize-mcp-connectors-for-your-entire-organization
  - https://support.claude.com/en/articles/12592343-enabling-and-using-the-desktop-extension-allowlist
  - https://support.claude.com/en/articles/12922929-building-desktop-extensions-with-mcpb
  - https://support.claude.com/en/articles/10167454-use-the-github-integration
  - https://support.claude.com/en/articles/11725091-when-to-use-desktop-and-web-connectors
  - https://claude.com/docs/connectors/building/mcpb
  - https://claude.com/docs/connectors/directory
  - https://claude.com/blog/integrations
  - https://claude.com/blog/connectors-directory
  - https://claude.com/blog/enterprise-managed-auth
  - https://claude.com/connectors
---

# Connectors

**Connectors** are the user-facing way to plug external apps, data, and services into Claude so it can both *read* your information (search Drive, read a Gmail thread, list Linear issues) and *take actions* (create a Calendar event, send a Slack message, open a Jira ticket). Under the hood, nearly every connector is a **Model Context Protocol (MCP) server** — Connectors is the friendly catalog, OAuth flow, and governance layer wrapped around the raw protocol described on [../capabilities/mcp.md](../capabilities/mcp.md). A connector inherits *your* permissions from the source system: if you cannot see a file in Google Drive, Claude cannot retrieve it through the Drive connector either.

## At a glance

| | |
|---|---|
| **What it is** | A directory of OAuth-authenticated integrations (built on MCP) that give Claude tools to access external apps/data and act on your behalf. |
| **Where you find it** | In any chat: the `+` button (lower left) or `/` -> hover **Connectors** -> **Manage connectors**. Or **Settings -> Customize -> Connectors**. Public catalog at `claude.com/connectors`. |
| **Who can use it** | Google Workspace + browsing the directory: all plans. Remote app/service (Integrations) connectors: **Pro, Max, Team, Enterprise**. Custom remote-MCP connectors: **Free (limited to one), Pro, Max, Team, Enterprise**. Org-wide enablement on Team/Enterprise requires an Owner or Primary Owner. |
| **Surfaces** | Claude.ai (web), Claude Desktop, Claude Mobile (iOS/Android), [Cowork](../platform/cowork.md), [Claude Code](../platform/claude-code.md), and the API (via the MCP connector). |
| **Status** | GA for first-party connectors and the directory; **custom remote-MCP connectors are GA** (Free=1); **enterprise-managed auth is in beta** (Team/Enterprise). |

> Terminology: **Connectors** is the catalog/product users interact with. **MCP** is the open *protocol* the connectors speak. **Desktop Extensions (MCPB)** are locally-installed connectors packaged as a `.mcpb` file. **Plugins** ([../capabilities/plugins.md](../capabilities/plugins.md)) are a separate bundling mechanism that can *include* MCP servers but also ship slash commands, agents, and hooks.

---

## What a connector actually is

A connector is a registered MCP server plus an OAuth (or local) authentication handshake. When you connect it, Claude loads the server's **tools** (callable actions like `search_files` or `create_event`), and during a conversation Claude decides which tools to call to satisfy your request. Three transport flavours exist:

| Type | Runs where | Auth | Plans | Example |
|---|---|---|---|---|
| **Remote (web) connector** | Provider's servers over HTTPS, brokered through Anthropic's infrastructure | OAuth | Pro, Max, Team, Enterprise | Slack, Linear, Atlassian, Notion, Zoom |
| **Desktop Extension (MCPB)** | Locally on your machine via stdio | None / local SSO | All plans (Claude Desktop) | Filesystem, local Git, Figma desktop |
| **Custom connector** | Any remote MCP server you point Claude at by URL | Optional OAuth client ID/secret | Free (1), Pro, Max, Team, Enterprise | Your internal API behind a remote MCP server |

Because remote connectors are brokered through Anthropic's servers, a remote custom connector's MCP endpoint must be **publicly internet-reachable** — it is not called from your machine's network interface. MCPB extensions are the opposite: they run on your machine and can reach firewalled, internal, or SSO-protected resources. (See the help-center guide [When to use desktop and web connectors](https://support.claude.com/en/articles/11725091-when-to-use-desktop-and-web-connectors).)

---

## The connector directory (catalog)

The **Connectors Directory** is a browsable, categorized catalog of "verified and reviewed MCP servers that work across all Claude products — Claude.ai, Claude Desktop, Claude Mobile, Claude Code, and Cowork." Every integration is **vetted by Anthropic for security, reliability, and compatibility** before listing. Each connector has its own page describing its use cases, its read/write capabilities, and its availability. Browse it three ways:

1. In chat: `+` -> hover **Connectors** -> **Manage connectors** -> `+` next to Connectors -> **Browse connectors**.
2. **Settings -> Customize -> Connectors -> `+`**.
3. Public web: **`claude.com/connectors`** (the directory launched **July 14, 2025**).

> WARN: verify — exact category names and the total connector count on `claude.com/connectors` change as connectors are added; the live site is the source of truth.

### First-party / popular connectors (verified)

These appeared in official launch announcements, help-center articles, or the public directory. Most connectors are **built and maintained by third-party developers** (often the app vendor itself, e.g. Zoom built the Zoom connector) using MCP; Anthropic curates and vets the directory but is not the publisher of most connectors.

**Google Workspace (available on all plans):**

- **Google Drive** — search/retrieve Docs, read Sheets/Slides/PDFs/images/MS Office files, upload, create/copy files, create folders, view file permissions and recent files.
- **Gmail** — search and read email and threads, create/list drafts (Claude **cannot send**), create/update/delete labels, label/unlabel messages and threads.
- **Google Calendar** — list calendars/events, get/create/update/delete events, respond to invites, suggest meeting times, manage attendees and recurring meetings.

**Original Integrations launch partners (May 2025):** Atlassian (**Jira** and **Confluence**), **Zapier**, **Cloudflare**, **Intercom**, **Asana**, **Square**, **Sentry**, **PayPal**, **Linear**, **Plaid** — with **Stripe**, **GitLab**, and **Box** named as coming. Integrations launched on Max/Team/Enterprise and, per a June 3, 2025 update, became available on **Pro, Max, Team, and Enterprise**.

**Connectors Directory launch examples (July 2025):** remote — **Notion**, **Canva**, **Stripe**; desktop extensions — **Figma**, **Socket**, **Prisma**.

**Other officially named connectors:** **Slack** (official MCP server), **GitHub** (see below), **Zoom** (built by Zoom — search meetings, retrieve AI summaries/docs/recordings/whiteboards, transcripts, next steps, playback links; read & write), **Supabase** (database/auth/storage/edge functions), **Microsoft 365** (SharePoint, OneDrive, Outlook mail + calendar, Teams chat), **DocuSign**, **Apollo**, **Clay**, **Outreach**, **Similarweb**, **MSCI**, **LegalZoom**, **FactSet**, **WordPress**, **Harvey**, **Granola**, and a built-in **Web Search** connector.

### The GitHub integration (corrected)

A first-party **GitHub integration** exists and is documented in the help center. Unlike a typical remote OAuth connector that exposes a broad tool set, the consumer GitHub integration is oriented around **syncing repository files as context**:

- It syncs **file names and contents in a repo on a specific branch**. It does **not** retrieve commit history, pull requests, or other metadata.
- **In a chat:** `+` (lower left) -> **Add from GitHub** -> browse and pick specific files/folders.
- **In a Project:** `+` in the upper-right of the project knowledge section -> **GitHub** -> search a repo or paste a URL -> select files/folders -> optionally **Sync** to refresh to the latest version.
- **Private repos** require authenticating with GitHub; you may need an org admin to grant the **Claude GitHub App** access.
- (Distinct from [Claude Code](../platform/claude-code.md), which has its own deeper GitHub workflow — PRs, Actions, the `/install-github-app` flow.)

---

## Enabling a connector and the OAuth flow

### From a chat

1. Click the **`+`** in the lower left (or type **`/`**).
2. Hover **Connectors** -> **Manage connectors**.
3. Click the **`+`** next to **Connectors** to browse the directory.
4. Click a connector, review its capabilities, then click **Connect** (remote) or **Install** (desktop extension).
5. Follow the OAuth prompts in the popup to **grant Claude access to your account**. Claude mirrors your existing permissions in that service.

### From settings

**Settings -> Customize -> Connectors -> `+`** to browse and connect. Once connected, the connector appears under **Connectors** in the `+` menu where you can **toggle it on/off** per chat.

### What OAuth grants

Authentication is a standard OAuth handshake directly with the provider (e.g., Google's consent screen for Workspace). The connector inherits your access; it cannot exceed it. You can **revoke** access at any time from Claude's connector settings or from the third-party service's own security/connected-apps page.

---

## Using a connector in chat (examples)

You rarely call tools by name. You ask in natural language and Claude **automatically detects which connector tools it needs** and requests approval before write actions.

**Search Google Drive and summarize:**

```
You: Find the Q2 planning doc in my Drive and give me the 5 key decisions.
Claude: [calls Google_Drive.search_files -> read_file_content]
        Found "Q2 Planning v3". Top 5 decisions: ...
```

**Summarize a Gmail thread (draft only — no send):**

```
You: Summarize the email thread with Priya about the vendor contract.
Claude: [Gmail.search_threads -> get_thread] Here's the thread in 4 bullets ...
        Want me to draft a reply? (I can create a draft; I can't send it.)
```

**Create a Calendar event (with write approval):**

```
You: Schedule a 30-min sync with the design team Thursday at 2pm.
Claude: [Google_Calendar.suggest_time / create_event]
        Created "Design sync" Thu 2:00-2:30pm. (Approve action?) [Allow] [Deny]
```

**Act across apps (Linear + Slack):**

```
You: Open a Linear bug for the login crash and post the link in #eng.
Claude: [Linear.create_issue] Created ENG-482.
        [Slack.send_message] Posted to #eng. Done.
```

**Recap a meeting (Zoom):**

```
You: Find my Zoom call with Acme last week and give me the action items.
Claude: [Zoom.search_meetings -> get summary/next steps]
        Acme sync, Jun 18. Next steps: send revised SOW, schedule pilot ...
```

Connectors are only available in **private projects**; chats that contain synced connector content **can't be shared** (on Team and Enterprise plans). To attach Drive files directly to a chat or [Project](../capabilities/projects.md), use `+` -> **Add from Google Drive** and paste a URL or pick a recent doc.

---

## Tool access (managing many connectors)

When you connect many servers, the tool list can get large. Each connector has a **Tool access** setting controlling *how connectors load in your conversation*:

| Mode | Behavior |
|---|---|
| **Auto** (default) | Claude loads the connector's tools as it deems relevant. |
| **On demand** | Tools are loaded only when the conversation needs them. |

**On demand** is recommended *if you have 10 or more connectors active*, to give conversations more context room.

---

## Admin & organization controls (Team / Enterprise)

On **Team** and **Enterprise** plans, connectors are governed centrally. A **Member can connect their own account** to a connector that is already enabled org-wide, but **cannot add new connectors to the org catalog** — only an **Owner or Primary Owner** can. Connectors are enabled org-wide from **Admin / Organization settings**.

> Nuance: the general connectors help article states that *per-user or per-group connector gating is not currently available* on standard Team/Enterprise settings. **Enterprise role-based permissions** (below) are the supported mechanism for gating connectors and individual tools by role.

### Enabling org-wide

**Organization settings -> Connectors -> Add**. For a custom remote MCP server: hover **Custom -> Web**, paste the **Remote MCP server URL**, optionally set **OAuth Client ID** and **OAuth Client Secret** under **Advanced settings**, then **Add**. Allow ~1 minute for changes to propagate.

### Role-based permissions (Enterprise)

On **Enterprise** plans, the custom-role editor includes a **Connectors tab** (next to **Permissions**) that controls, per role, which connectors — and which tools on those connectors — members may use. Each connector offers four modes:

| Mode | Behavior |
|---|---|
| **Always allow** | Every tool on the connector is available; members can set their own approval to "Always allow." |
| **Needs approval** | Every tool is available, but members confirm each call. |
| **Blocked** | The connector is hidden from members with this role. |
| **Custom** | The connector expands to show its tools as individual rows, each independently set to **Always allow / Needs approval / Blocked**. |

Use **Custom** to permit read operations while disabling writes — e.g., allow `search`/`read` but block `create`/`delete` org-wide for a given role. This complements **capabilities** (which gate Claude's built-in features) by gating connected apps like Slack, Google Drive, or Jira.

### Verified-domain restriction

Owners can restrict connectors that support **verified domains** so they only authenticate with **organizational accounts**, preventing employees from connecting personal accounts.

### Desktop Extension allowlist (Team / Enterprise)

Owners and Primary Owners can enforce an **allowlist** for locally-installed Desktop Extensions (MCPB), managed under **Organization settings -> Connectors -> Desktop tab** (toggle labeled **Allowlist**). It is **disabled by default**. When enabled it:

- Blocks installation of any extension not on the approved list.
- Forces removal of existing unapproved extensions from user machines.
- Prevents drag-or-click installation of MCPs outside the sanctioned registry.

Admins curate the list with **Add to your team** and a kebab-menu **Remove from allowlist** in the browse-extensions UI. Requires **Claude Desktop 0.13.91 or higher**. (It does not prevent a user from tampering with the local file contents of an already-installed extension — see Security model.)

### Enterprise-managed auth (beta)

Instead of every user running their own OAuth flow, **enterprise-managed authorization** lets admins **authorize a connector once** through the org's identity provider (Okta at launch; additional IdPs "coming soon"). Users then **inherit access through the IdP groups and roles they already have**, and the connector is present the first time they open Claude — consistent across Claude chat, Claude Code, and Cowork. It is the first implementation of the **Enterprise-Managed Authorization extension to MCP**. Available in **beta** on Team and Enterprise (apply for access / join the waitlist). Connectors supporting it at launch: **Asana, Atlassian, Canva, Figma, Granola, Linear, Supabase** (Slack "coming soon").

---

## Custom connectors (add a remote MCP server by URL)

Point Claude at **any** remote MCP server you trust. Available on **Free** (limited to **one** custom connector), **Pro**, **Max**, **Team**, and **Enterprise**, across Claude, Cowork, and Claude Desktop.

**Individual (Pro/Max):**

1. **Settings -> Customize -> Connectors**.
2. **`+` -> Add custom connector**.
3. Enter the **Remote MCP server URL**.
4. *(Optional)* **Advanced settings** -> **OAuth Client ID** + **OAuth Client Secret**.
5. **Add**, then complete the OAuth connection.

**Team/Enterprise:** an Owner adds it org-wide (above); members then go to **Customize -> Connectors**, find it under the **Custom** label, and **Connect**.

Requirements & safety:

- The MCP server must be **publicly internet-accessible** (it's brokered by Anthropic, not your machine).
- **Only connect to trusted servers** and **review requested permissions carefully** — a malicious MCP server can embed hidden instructions attempting unauthorized actions (a prompt-injection vector). See the security model in [../capabilities/mcp.md](../capabilities/mcp.md).

---

## Desktop Extensions (MCPB / `.mcpb`)

A **Desktop Extension** is a connector that runs **locally** inside Claude Desktop. It is packaged as an **`.mcpb`** file — a zip archive containing a local (stdio) MCP server plus a **`manifest.json`** — enabling **single-click install**, bundled dependencies, offline operation, and **no OAuth**. (MCPB is the successor naming for the earlier **`.dxt`** "Desktop Extension" bundles; the spec/tooling lives in `modelcontextprotocol/mcpb`.)

**Install (any of three ways):**

1. Double-click the `.mcpb` file.
2. Drag-and-drop it onto the Claude Desktop window.
3. **Settings -> Extensions -> Advanced -> Install Extension**.

Each method opens an install UI to **review permissions, configure required settings, and complete setup per-user**.

**Build one with the MCPB CLI:**

```bash
npm install -g @anthropic-ai/mcpb   # install CLI
mcpb init                            # interactively generate manifest.json
mcpb pack                            # bundle the folder into a .mcpb
# double-click the generated .mcpb to test in Claude Desktop
```

`manifest.json` is required metadata describing what the extension does, how to run it, which tools it provides, and what configuration it needs. It declares the server's runtime/entry point, tool definitions, a **`user_config`** section (Claude Desktop auto-generates a settings UI from it, with **sensitive-value** handling for secrets), `compatibility` (Node.js is recommended since it ships with Claude Desktop on macOS/Windows), and an icon (256x256 min, 512x512 recommended). The full field reference is `MANIFEST.md` in the `modelcontextprotocol/mcpb` repo.

**MCPB vs remote connector — when to choose which:**

| Choose **MCPB** when you need | Choose **remote connector** when you need |
|---|---|
| Access behind a firewall (private DBs, internal Jira/Confluence) | Cloud SaaS over OAuth |
| Existing SSO without token management | Multi-platform distribution + centralized updates |
| Direct filesystem / local Git / Docker / hardware | Zero local install footprint |
| Privacy-sensitive, fully-local operation | Server-side compute you control |

To reach more users, working MCPBs (and remote connectors) can be **submitted to the Connectors Directory**. Submission requires a **Team or Enterprise organization with directory management access** (org Owners by default): review the submission guidelines, ensure the server meets security/compatibility standards, and submit through the submission portal. The submission guide also calls for tool annotations, a privacy policy, working examples per tool, and test credentials.

---

## Security & permission model

- **Permission inheritance** — connectors never exceed your access in the source system.
- **Explicit write approval** — actions that change data prompt before running (per the org's approval mode); Gmail in particular **reads and drafts only — it does not send**.
- **Data residency caveat** — retrieved data is stored on Anthropic's servers (encrypted in transit and at rest), but **third-party services run on their own infrastructure under their own terms**, possibly outside the US. Settings like Enterprise **US-only inference do not change where the third-party service operates**.
- **Training** — connector data isn't used for model training (with the usual consumer-plan opt-in exceptions).
- **Custom/remote MCP risk** — untrusted servers can carry hidden prompt-injection instructions; vet them and prefer least-privilege scopes.
- **Local MCPB caveat** — the desktop-extension allowlist controls *what can be installed* but does not stop a user from modifying the local files of an already-installed extension; treat local-server contents as part of the endpoint's trust boundary.
- **Revocation** — anytime, from Claude's connector settings or the provider's security page.

---

## Connectors vs MCP vs Plugins (disambiguation)

| Concept | What it is | Where defined |
|---|---|---|
| **Connectors** | The user-facing catalog + OAuth + governance for integrations | this page |
| **MCP** | The open protocol (tools/resources/prompts, transports) connectors speak | [../capabilities/mcp.md](../capabilities/mcp.md) |
| **Desktop Extension (MCPB)** | A locally-installed connector packaged as `.mcpb` | this page |
| **Plugins** | Bundles that may include MCP servers *plus* slash commands, agents, hooks | [../capabilities/plugins.md](../capabilities/plugins.md) |
| **Skills** | Model-invoked instruction folders (`SKILL.md`); not a network integration | [../capabilities/skills.md](../capabilities/skills.md) |

---

## Related pages

- [../capabilities/mcp.md](../capabilities/mcp.md) — the underlying Model Context Protocol (transports, primitives, config).
- [../platform/claude-ai.md](../platform/claude-ai.md) — where the Connectors UI lives in the web/desktop/mobile apps.
- [../platform/cowork.md](../platform/cowork.md) — Cowork uses the same connectors for autonomous knowledge work.
- [../capabilities/projects.md](../capabilities/projects.md) — connectors feed Project knowledge bases; connectors are private-project only.
- [../capabilities/plugins.md](../capabilities/plugins.md) — how plugins differ from connectors and can ship MCP servers.
- [../capabilities/skills.md](../capabilities/skills.md) — Agent Skills, a separate extensibility mechanism.
- [../platform/claude-code.md](../platform/claude-code.md) — Claude Code's MCP/connector configuration and deeper GitHub workflow.

## Open questions / to verify

- Exact current **category names and total connector count** on `claude.com/connectors` (the live site changes frequently and the docs do not enumerate categories).
- Whether enterprise-managed auth has moved from **beta to GA** and added IdPs beyond Okta since launch.
- Precise per-plan caps (e.g., max number of remote connectors a paid user can connect beyond the documented Free=1 custom-connector limit) — not stated in reviewed sources.
- Current GA status of the originally-promised **Stripe / GitLab / Box** rollouts (Stripe is confirmed live in the directory; GitLab and Box status in the consumer directory not re-confirmed here).
- Whether the consumer **GitHub integration** will expand beyond file-content sync (issues/PRs) in claude.ai, separate from Claude Code's GitHub app.

## Sources

- [Use connectors to extend Claude's capabilities (Help Center)](https://support.claude.com/en/articles/11176164-use-connectors-to-extend-claude-s-capabilities)
- [Use Google Workspace connectors (Help Center)](https://support.claude.com/en/articles/10166901-use-google-workspace-connectors)
- [Get started with custom connectors using remote MCP (Help Center)](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)
- [Build custom connectors via remote MCP servers (Help Center)](https://support.claude.com/en/articles/11503834-build-custom-connectors-via-remote-mcp-servers)
- [MCP connectors (Help Center)](https://support.claude.com/en/articles/14503689-mcp-connectors)
- [Set up role-based permissions on Enterprise plans (Help Center)](https://support.claude.com/en/articles/13930458-set-up-role-based-permissions-on-enterprise-plans)
- [Authorize MCP connectors for your entire organization (Help Center)](https://support.claude.com/en/articles/15537633-authorize-mcp-connectors-for-your-entire-organization)
- [Enabling and using the desktop extension allowlist (Help Center)](https://support.claude.com/en/articles/12592343-enabling-and-using-the-desktop-extension-allowlist)
- [Building Desktop Extensions with MCPB (Help Center)](https://support.claude.com/en/articles/12922929-building-desktop-extensions-with-mcpb)
- [Use the GitHub integration (Help Center)](https://support.claude.com/en/articles/10167454-use-the-github-integration)
- [When to use desktop and web connectors (Help Center)](https://support.claude.com/en/articles/11725091-when-to-use-desktop-and-web-connectors)
- [Build a desktop extension with MCPB (Developer docs)](https://claude.com/docs/connectors/building/mcpb)
- [Connectors directory (Developer docs)](https://claude.com/docs/connectors/directory)
- [Claude can now connect to your world / Integrations (Blog)](https://claude.com/blog/integrations)
- [Discover tools that work with Claude / Connectors Directory (Blog)](https://claude.com/blog/connectors-directory)
- [Centrally manage authorization for MCP connectors (Blog)](https://claude.com/blog/enterprise-managed-auth)
- [Connectors catalog](https://claude.com/connectors)
