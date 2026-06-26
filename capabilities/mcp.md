---
type: Protocol
title: Model Context Protocol (MCP)
description: The open JSON-RPC standard that lets Claude connect to external tools, data, and prompts, and how to configure MCP servers across Claude Code and the Claude apps.
domain: capabilities
tags: [mcp, model-context-protocol, tools, resources, prompts, stdio, streamable-http, sse, oauth, connectors, claude-code, json-rpc, mcp-servers, agent-tools]
related: [connectors, settings, slash-commands, plugins, claude-code-overview, capabilities-and-modes, hooks, cli-shortcuts, skills, subagents]
resource: https://code.claude.com/docs/en/mcp
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://code.claude.com/docs/en/mcp
  - https://modelcontextprotocol.io/docs/concepts/architecture
  - https://modelcontextprotocol.io/docs/develop/build-server
  - https://modelcontextprotocol.io/specification/2025-11-25/basic/transports
  - https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/
---

# Model Context Protocol (MCP)

The Model Context Protocol (MCP) is the open standard that lets Claude (and any other LLM app) talk to external tools, data sources, and prompt libraries through a single, uniform interface. It is often described as "USB-C for AI": instead of writing a bespoke integration for every tool, you run an MCP **server** that exposes capabilities, and any MCP-aware **host** — Claude Code, Claude Desktop, claude.ai, Cursor, VS Code — can plug into it. MCP is what powers Claude Code's `claude mcp add` workflow and the [Connectors](./connectors.md) catalog inside the Claude apps; they are two front ends over the same protocol.

> **At a glance**
> - **What it is:** An open, vendor-neutral protocol (JSON-RPC 2.0) for connecting LLM applications to tools, resources, and prompts. Created by Anthropic (announced Nov 2024), now community-governed at `modelcontextprotocol.io`.
> - **Where you find it:** Claude Code (`claude mcp …` CLI, `.mcp.json`, the `/mcp` panel); Claude Desktop and claude.ai (surfaced as **Connectors**); the Claude / Anthropic API (MCP connector); third-party hosts.
> - **Who can use it by plan:** MCP in Claude Code is available on any plan that has Claude Code. Remote connectors in the apps are available on Pro, Max, Team, and Enterprise; on Team/Enterprise, **admins** control which connectors org members may add. (See [Connectors](./connectors.md).)
> - **Status:** Stable and widely adopted. Current spec revision **2025-11-25**. A next-version **release candidate was locked 2026-05-21** with the final spec dated **2026-07-28**; it introduces a *stateless core* (drops the `initialize`/`initialized` handshake and `Mcp-Session-Id`, adds `Mcp-Method`/`Mcp-Name` routing headers), OAuth/OIDC hardening, **MCP Apps** (server-rendered UIs), a **Tasks** extension, full JSON Schema 2020-12, and a 12-month deprecation policy. As of this page's date (2026-06-26) the final spec is **not yet published**, so treat next-version details as forward-looking — WARN: verify against `modelcontextprotocol.io` before relying on them.

---

## 1. What MCP is (and is not)

MCP standardizes **how an AI application gets context and takes actions** — nothing more. It defines the message schema and lifecycle between a client and a server; it deliberately does **not** dictate how the host uses the LLM, how it manages context windows, or which model runs. That separation is why one MCP server (say, a GitHub server) works unchanged across every compliant host.

Concretely, MCP replaces the N×M integration problem (N apps × M tools) with N+M: each app implements an MCP client once, each tool implements an MCP server once, and they interoperate.

| MCP **is** | MCP **is not** |
| --- | --- |
| A wire protocol (JSON-RPC 2.0) for tools/resources/prompts | A model or an inference API |
| Transport-agnostic (stdio, HTTP) | Tied to Anthropic or any one vendor |
| A capability-negotiation handshake | A permissions/sandboxing system (the host enforces that) |
| The plumbing under "Connectors" and `claude mcp add` | A replacement for [Skills](./skills.md) or [Plugins](./plugins.md) (those compose *with* MCP) |

---

## 2. Architecture: host, client, server

MCP uses a **client–server** architecture with three roles. The key subtlety: a single host runs **one client per server**, each holding a dedicated 1:1 connection.

```mermaid
graph TB
    subgraph Host["MCP Host (e.g. Claude Code / Claude Desktop)"]
        C1["MCP Client 1"]
        C2["MCP Client 2"]
        C3["MCP Client 3"]
    end
    SA["Server A — local (Filesystem) · stdio"]
    SB["Server B — local (Postgres) · stdio"]
    SC["Server C — remote (Sentry) · Streamable HTTP"]
    C1 --- SA
    C2 --- SB
    C3 --- SC
```

- **MCP Host** — the AI application that coordinates one or more clients and owns the LLM conversation. Examples: Claude Code, Claude Desktop, claude.ai, VS Code.
- **MCP Client** — the connector component *inside* the host. The host instantiates one client object per server and that client maintains the connection, performs capability negotiation, and routes requests.
- **MCP Server** — a program that exposes capabilities (tools/resources/prompts). "Local" servers run on your machine over stdio; "remote" servers run as a hosted service over HTTP. The word "server" refers to the program's *role*, not where it runs.

### Two layers

MCP is conceptually two nested layers:

- **Data layer (inner):** the JSON-RPC 2.0 protocol — lifecycle management, the primitives (tools/resources/prompts), client features (sampling/elicitation/logging), and notifications.
- **Transport layer (outer):** how bytes move — stdio or Streamable HTTP — plus connection setup, message framing, and authentication. The same JSON-RPC messages flow identically over either transport, so a stdio server can be lifted to HTTP without touching its core logic.

---

## 3. Transports

MCP defines two standard transports. A third (SSE) is legacy/deprecated. Claude Code additionally supports WebSocket as a non-spec extension for push servers.

| Transport | Best for | Auth | Claude Code `--transport` value | Notes |
| --- | --- | --- | --- | --- |
| **stdio** | Local tools (filesystem, shell, local DB), custom scripts | Process env / none | `stdio` | Host launches the server as a subprocess; talks over stdin/stdout. No network overhead. |
| **Streamable HTTP** | Remote/hosted SaaS servers | Bearer / API key / custom header / **OAuth 2.0** | `http` (JSON alias: `streamable-http`) | Single HTTP endpoint (POST + GET) with optional Server-Sent Events for streaming. Introduced in spec 2025-03-26; the **recommended** remote transport. |
| **SSE** *(deprecated)* | Older remote servers | header | `sse` | Legacy Server-Sent Events transport. Use HTTP where available. |
| **WebSocket** *(Claude Code extension)* | Servers that push events to Claude unprompted | header-only (`headers` / `headersHelper`) | not available — configure via JSON `"type":"ws"` | Persistent bidirectional connection; no OAuth, no `--transport` flag. |

The WebSocket (`type:"ws"`) entry accepts the same `url`, `headers`, `headersHelper`, `timeout`, and `alwaysLoad` fields as `http`; only the auth mechanism differs (header-only, no OAuth).

> **stdio logging gotcha:** in a stdio server, never `print()` to stdout — it corrupts the JSON-RPC stream. Log to **stderr** instead (`print(..., file=sys.stderr)` or the `logging` module). Per spec, the client may capture/forward/ignore stderr and **should not** treat stderr output as an error signal.

> **Streamable HTTP security (spec):** servers **must** validate the `Origin` header (respond `403` if invalid) to prevent DNS-rebinding attacks, and when running locally **should** bind to `127.0.0.1` rather than `0.0.0.0`. The endpoint is a single path supporting both POST and GET; sessions may carry an `MCP-Session-Id`.

---

## 4. Server primitives (what servers expose)

MCP defines three core primitives a **server** offers. Each has discovery (`*/list`), retrieval/execution methods, and optional `list_changed` notifications.

| Primitive | What it is | Key methods | In Claude Code you use it as… |
| --- | --- | --- | --- |
| **Tools** | Executable functions the model can call (file ops, API calls, DB queries) — usually with user approval | `tools/list`, `tools/call` | Regular tools, named `mcp__<server>__<tool>` |
| **Resources** | Read-only data sources (file contents, DB records, API responses), addressed by URI | `resources/list`, `resources/read` | `@server:protocol://path` mentions |
| **Prompts** | Reusable interaction templates (system prompts, few-shot examples) | `prompts/list`, `prompts/get` | Slash commands `/mcp__<server>__<prompt>` |

A single server can mix all three — e.g. a database server exposes a `query` **tool**, a `schema://` **resource**, and a `few_shot_examples` **prompt**.

### Tool definition shape

A tool advertised in a `tools/list` response carries a name, human title, description, and a JSON Schema for its arguments:

```json
{
  "name": "weather_current",
  "title": "Weather Information",
  "description": "Get current weather information for any location worldwide",
  "inputSchema": {
    "type": "object",
    "properties": {
      "location": { "type": "string", "description": "City name or coordinates" },
      "units": { "type": "string", "enum": ["metric", "imperial", "kelvin"], "default": "metric" }
    },
    "required": ["location"]
  }
}
```

The model calls it via `tools/call` with `{ "name": "weather_current", "arguments": { "location": "San Francisco", "units": "imperial" } }`, and the server returns a `content` array (text, images, or embedded resources).

---

## 5. Client features (what hosts expose back to servers)

MCP is bidirectional: servers can call **back** into the host. These let server authors build richer interactions without bundling an LLM SDK.

| Client feature | What it does | Method |
| --- | --- | --- |
| **Sampling** | Server asks the host's LLM for a completion — stays model-independent | `sampling/createMessage` |
| **Elicitation** | Server asks the **user** for structured input or confirmation mid-task | `elicitation/create` |
| **Roots** | Server queries which filesystem directories it is scoped to | `roots/list` |
| **Logging** | Server sends log/debug messages to the host | `notifications/message` |

**Elicitation in Claude Code** is automatic — no config needed. A server can request input two ways: **Form mode** (Claude Code renders the server's form fields, e.g. username/password) or **URL mode** (Claude Code opens a browser URL, you complete the flow, then confirm in the CLI). To auto-respond without a dialog, use the [`Elicitation` hook](../claude-code/hooks.md).

**Roots in Claude Code:** for stdio servers, Claude Code sets `CLAUDE_PROJECT_DIR` in the spawned server's environment to the project root, and the server can also call `roots/list` to learn the launch directory.

> **Experimental — Tasks:** beyond server and client primitives, the spec defines a cross-cutting utility primitive, **Tasks**, as a durable execution wrapper for deferred result retrieval and status tracking of long-running requests (expensive computations, batch jobs, multi-step workflows). Marked *experimental* in the current concepts docs and promoted to an extension in the next-version RC; WARN: verify support per host before relying on it.

---

## 6. Lifecycle: capability negotiation

Every connection opens with an `initialize` handshake that negotiates the protocol version and declares which capabilities each side supports — so neither party attempts an unsupported operation.

```json
// Client → Server
{ "jsonrpc": "2.0", "id": 1, "method": "initialize",
  "params": {
    "protocolVersion": "2025-11-25",
    "capabilities": { "elicitation": {} },
    "clientInfo": { "name": "claude-code", "version": "2.1.x" }
  } }

// Server → Client
{ "jsonrpc": "2.0", "id": 1, "result": {
    "protocolVersion": "2025-11-25",
    "capabilities": { "tools": { "listChanged": true }, "resources": {} },
    "serverInfo": { "name": "weather", "version": "1.0.0" }
  } }

// Client → Server (ready)
{ "jsonrpc": "2.0", "method": "notifications/initialized" }
```

The `protocolVersion` is *negotiated*: the client proposes a version (the latest it supports, e.g. `2025-11-25`), and if the server only speaks an older one, both fall back to a mutually supported revision (the MCP examples themselves still show `2025-06-18`). If no common version exists, the connection is terminated. Over HTTP the client then stamps every subsequent request with an `MCP-Protocol-Version:` header; a server that receives none assumes `2025-03-26` for backwards compatibility.

A server that declared `"tools": { "listChanged": true }` may later push `notifications/tools/list_changed`; Claude Code honors these **dynamic tool updates** and refreshes available tools/prompts/resources from that server without a disconnect/reconnect cycle.

---

## 7. Configuring MCP in Claude Code

The CLI surface is `claude mcp …`. The general add form is:

```bash
claude mcp add [options] <name> -- <command> [args...]
```

All options (`--transport`, `--scope`, `--env`, `--header`, OAuth flags) come **before** the server name. For stdio servers, the `--` separates Claude's own flags from the command that runs the server; everything after `--` is passed to the server untouched.

### 7.1 Adding servers by transport

```bash
# Remote HTTP server (recommended for cloud services)
claude mcp add --transport http notion https://mcp.notion.com/mcp

# HTTP server with a static Bearer token
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  --header "Authorization: Bearer YOUR_GITHUB_PAT"

# Remote SSE server (legacy)
claude mcp add --transport sse asana https://mcp.asana.com/sse

# Local stdio server with an env var (note: an option must sit between --env and the name)
claude mcp add --env AIRTABLE_API_KEY=YOUR_KEY --transport stdio airtable \
  -- npx -y airtable-mcp-server

# stdio server with its own flags after --
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"
```

| Flag | Purpose |
| --- | --- |
| `--transport <stdio\|http\|sse>` | Transport type (default `stdio`). JSON `type` also accepts `streamable-http` and `ws`. |
| `--scope <local\|project\|user>` | Where the config is stored (see below). Default `local`. |
| `--env KEY=value` | Pass an env var to a stdio server (repeatable). |
| `--header "Name: value"` | Static header for HTTP/SSE/WS auth (repeatable). |
| `--client-id` / `--client-secret` / `--callback-port` | Pre-configured OAuth credentials (HTTP/SSE only). |

Manage configured servers:

```bash
claude mcp list                   # list all servers (and status)
claude mcp get github             # details for one server (shows scope, transport, OAuth state)
claude mcp remove github          # delete a server
claude mcp reset-project-choices  # re-prompt for .mcp.json approvals
claude mcp serve                  # run Claude Code itself as a stdio MCP server (see §10)
```

`claude mcp list` shows pending project servers as `⏸ Pending approval`; `claude mcp get` shows pending as `⏸ Pending approval` and rejected as `✗ Rejected`.

> **Reserved name:** the server name `workspace` is reserved for internal use. If your config defines a server called `workspace`, Claude Code skips it at load time and warns you to rename it.

> **Startup vs. tool timeouts:** `MCP_TIMEOUT` (env var, ms) bounds how long Claude Code waits for a server to *start/connect* — e.g. `MCP_TIMEOUT=10000 claude` for 10s. That is distinct from the per-server `timeout` field and `MCP_TOOL_TIMEOUT`, which bound a single *tool call* (see §7.3).

If your prompt needs a tool from a server that is still connecting, Claude waits for it: with Tool Search on (default) the wait happens inside the `ToolSearch` call; in non-Tool-Search configs (Vertex AI, custom `ANTHROPIC_BASE_URL`, or `ENABLE_TOOL_SEARCH=false`) Claude uses a `WaitForMcpServers` tool instead.

### 7.2 The three scopes

The scope controls which projects a server loads in and whether it is shared. When the same server name appears in multiple scopes, the highest-precedence definition wins **whole** (fields are not merged).

| Scope | Loads in | Shared with team? | Stored in |
| --- | --- | --- | --- |
| **local** *(default)* | Current project only | No | `~/.claude.json` (under that project's path) |
| **project** | Current project only | Yes — via version control | `.mcp.json` at project root |
| **user** | All your projects | No | `~/.claude.json` |

Precedence (highest → lowest): **local → project → user → plugin-provided → claude.ai connectors.** Scopes match duplicates by name; plugins and connectors match by endpoint (URL/command).

> Naming caveat: MCP "local scope" lives in `~/.claude.json`, which is *not* the same as general "local settings" (`.claude/settings.local.json`). See [Settings](../claude-code/settings.md).

### 7.3 `.mcp.json` (project scope, committed to git)

`claude mcp add --scope project …` writes a standardized `.mcp.json` at the repo root. Check it in so every teammate gets the same tools. Claude Code prompts for approval before first use of any `.mcp.json` server (security).

```json
// .mcp.json — a mix of stdio + remote, with env-var expansion
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "${CLAUDE_PROJECT_DIR:-.}/src"],
      "env": {}
    },
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": { "Authorization": "Bearer ${API_KEY}" },
      "alwaysLoad": true,
      "timeout": 600000
    }
  }
}
```

**Environment-variable expansion** works in `command`, `args`, `env`, `url`, and `headers`:

- `${VAR}` → value of `VAR` (config fails to parse if unset and no default)
- `${VAR:-default}` → `VAR` if set, else `default`

Useful per-server fields: `timeout` (ms, hard wall-clock per tool call — overrides `MCP_TOOL_TIMEOUT` for that server), `alwaysLoad` (exempt from Tool Search deferral — see §11), `headersHelper` (dynamic auth — see §8), `oauth` (OAuth overrides — see §8).

**Timeout details (`timeout` / `MCP_TOOL_TIMEOUT`):**

- The per-server `timeout` is a hard wall-clock cap per tool call; progress notifications from the server do **not** extend it.
- Values below `1000` are ignored and fall through to `MCP_TOOL_TIMEOUT`, whose default is **~28 hours** when unset. (Before v2.1.162, sub-1000 values were floored to 1s instead.)
- For HTTP/SSE servers the per-request first-byte fetch budget has a **60-second minimum**.
- **Idle timeout (v2.1.187+):** a call to a remote HTTP/SSE/WebSocket/claude.ai-connector server that sends no response and no progress notification for **5 minutes** aborts with an error rather than waiting out the wall-clock limit. Tune with `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` (ms), or set it to `0` to disable. Stdio servers (local processes) are exempt.

### 7.4 `claude mcp add-json` and importing

```bash
# Add directly from a JSON blob
claude mcp add-json weather-api \
  '{"type":"http","url":"https://api.weather.com/mcp","headers":{"Authorization":"Bearer token"}}'

# WebSocket (push) server — only configurable via JSON
claude mcp add-json events-server \
  '{"type":"ws","url":"wss://mcp.example.com/socket","headers":{"Authorization":"Bearer YOUR_TOKEN"}}'

# Import everything you already configured in Claude Desktop (macOS / WSL only)
claude mcp add-from-claude-desktop
```

`add-from-claude-desktop` shows an interactive picker, reads Claude Desktop's config from its standard location, and suffixes name collisions (`server_1`). Use `--scope user` to import into your user config.

### 7.4a Plugin-provided servers and tool names

[Plugins](./plugins.md) can bundle MCP servers (declared in a `.mcp.json` at the plugin root or inline in `plugin.json`); they start automatically when the plugin is enabled and are managed through plugin install, not `/mcp`. Run `/reload-plugins` to connect/disconnect a plugin's servers mid-session. Plugin server configs may use `${CLAUDE_PLUGIN_ROOT}` (bundled files), `${CLAUDE_PLUGIN_DATA}` (persistent state across updates), and `${CLAUDE_PROJECT_DIR}` (no default needed).

A plugin-bundled tool's **callable name embeds both the plugin and server key**:

```text
mcp__plugin_<plugin-name>_<server-name>__<tool-name>
# e.g. a `query` tool from server `database-tools` in plugin `my-plugin`:
mcp__plugin_my-plugin_database-tools__query
```

Use that full form in permission rules, a skill's `allowed-tools`, or a [subagent's `tools` field](../claude-code/subagents.md).

### 7.4b Push messages with Channels

An MCP server can push messages **into** your session (CI results, monitoring alerts, chat) so Claude reacts to external events while you are away. The server declares the `claude/channel` capability and you opt it in with the `--channels` flag at startup. This is the mechanism behind Telegram/Discord/webhook integrations. See the Channels docs (`code.claude.com/docs/en/channels`).

### 7.5 The `/mcp` command (inside a session)

Run `/mcp` to open the MCP panel. It shows, per server: connection **status**, the **tool count** (and flags servers that advertise tools but expose none), whether the server came from a **plugin** or a **claude.ai connector**, and **OAuth** controls (Authenticate / Clear authentication). Project servers awaiting approval show as `⏸ Pending approval`; rejected ones as `✗ Rejected`.

Reconnection: if an HTTP/SSE server drops mid-session, Claude Code auto-reconnects with exponential backoff (up to 5 attempts, 1s doubling); after that you retry manually from `/mcp`. Stdio servers are local processes and are not auto-reconnected.

---

## 8. Authenticating remote servers (OAuth)

Claude Code supports OAuth 2.0 for remote servers. A server is flagged for auth when it returns `401 Unauthorized` or `403 Forbidden` (or a `WWW-Authenticate` header pointing at its authorization server). Tokens are stored securely (system keychain on macOS) and refreshed automatically.

**Interactive flow (in a session):**

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
# then inside Claude Code:
/mcp           # pick the server → browser opens → log in → done
```

**From the shell (v2.1.186+):**

```bash
claude mcp login sentry              # run the OAuth flow without opening /mcp
claude mcp login sentry --no-browser # SSH / headless: prints URL, paste callback back
claude mcp logout sentry             # clear stored credentials
```

**Pre-configured credentials** (when a server doesn't support Dynamic Client Registration — error: *"Incompatible auth server: does not support dynamic client registration"*):

```bash
claude mcp add --transport http \
  --client-id your-client-id --client-secret --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

`--callback-port` fixes the OAuth redirect URI to `http://localhost:PORT/callback` (some servers require a pre-registered port). It can be used **standalone** (with dynamic client registration) or together with `--client-id` (pre-configured credentials). `--client-secret` prompts for masked input, or set `MCP_CLIENT_SECRET` in CI. For a public client with no secret, pass only `--client-id`. These flags apply to HTTP/SSE only — they have no effect on stdio servers. Verify with `claude mcp get <name>`.

Claude Code also supports servers that use a **Client ID Metadata Document (CIMD)** instead of DCR and discovers those automatically. Its default discovery chain is RFC 9728 Protected Resource Metadata at `/.well-known/oauth-protected-resource`, then RFC 8414 authorization-server metadata at `/.well-known/oauth-authorization-server`; override it with `oauth.authServerMetadataUrl`.

**OAuth tuning in `.mcp.json`** via an `oauth` object:

```json
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "oauth": {
        "scopes": "channels:read chat:write search:read",
        "authServerMetadataUrl": "https://auth.example.com/.well-known/openid-configuration"
      }
    }
  }
}
```

- `oauth.scopes` — space-separated string pinning the requested scopes (matches RFC 6749 §3.3 `scope` format; security teams use this to restrict). Takes precedence over both `authServerMetadataUrl` and the scopes discovered at `/.well-known`. Leave unset to let the server decide. If the auth server advertises `offline_access`, Claude Code appends it so tokens can refresh without a fresh sign-in. If a tool call later returns `403 insufficient_scope`, Claude Code re-authenticates with the *same* pin — widen `oauth.scopes` if the tool needs a scope outside it.
- `oauth.authServerMetadataUrl` — override the OAuth metadata discovery chain (must be `https://`; v2.1.64+). Its `scopes_supported` overrides what the upstream server advertises.
- `oauth.clientId` / `oauth.callbackPort` — the JSON-config equivalents of `--client-id` / `--callback-port`, usable via `claude mcp add-json` (pass `--client-secret` as a separate flag).

**Non-OAuth schemes** (Kerberos, short-lived tokens, internal SSO) use `headersHelper` — a command that prints a JSON object of headers to stdout, run fresh on each connect (10s timeout):

```json
{
  "mcpServers": {
    "internal-api": {
      "type": "http",
      "url": "https://mcp.internal.example.com",
      "headersHelper": "/opt/bin/get-mcp-auth-headers.sh"
    }
  }
}
```

The helper receives `CLAUDE_CODE_MCP_SERVER_NAME` and `CLAUDE_CODE_MCP_SERVER_URL`. Because it runs arbitrary shell, at project/local scope it executes only after you accept the workspace-trust dialog.

---

## 9. Referencing resources and prompts in Claude Code

### Resources via `@` mentions

Type `@` to autocomplete resources from all connected servers (alongside files). The format is `@server:protocol://resource/path`:

```text
Can you analyze @github:issue://123 and suggest a fix?
Please review the API docs at @docs:file://api/authentication
Compare @postgres:schema://users with @docs:file://database/user-model
```

Referenced resources are fetched and attached automatically; paths are fuzzy-searchable in the `@` autocomplete, and Claude Code auto-provides list/read tools when a server supports resources. Content can be any type the server returns (text, JSON, structured data).

### Prompts as slash commands

Server prompts surface as `/mcp__<server>__<prompt>`. Type `/` to discover them; pass args space-separated:

```text
/mcp__github__list_prs
/mcp__github__pr_review 456
/mcp__jira__create_issue "Bug in login flow" high
```

Prompt output is injected directly into the conversation. See [Slash commands](../claude-code/slash-commands.md).

---

## 10. MCP in Claude Desktop and the apps (= Connectors)

In Claude Desktop and claude.ai, MCP servers appear under **Settings → Connectors** rather than a CLI. The browser apps support **remote** (HTTP) connectors; Claude Desktop also runs **local** stdio servers via `claude_desktop_config.json`:

```json
// claude_desktop_config.json
{
  "mcpServers": {
    "claude-code": { "type": "stdio", "command": "claude", "args": ["mcp", "serve"], "env": {} }
  }
}
```

If you sign in to Claude Code with a claude.ai account, connectors you added at `claude.ai/customize/connectors` become available in Claude Code automatically and show in `/mcp` with a claude.ai indicator. (On Team/Enterprise, only admins add connectors.) Disable them with the `disableClaudeAiConnectors` setting or `ENABLE_CLAUDEAI_MCP_SERVERS=false`. Some Anthropic-hosted connectors (Microsoft 365, Gmail, Google Calendar) must be authenticated **on claude.ai**, not locally. Full catalog and admin controls live on the [Connectors](./connectors.md) page.

You can also run **Claude Code itself as an MCP server** so other hosts can use its tools:

```bash
claude mcp serve   # exposes Claude Code's View/Edit/LS/etc. tools over stdio
```

---

## 11. Tool Search (scaling many servers)

Claude Code's **Tool Search** keeps context low by deferring MCP tool *definitions* until needed — at session start only tool names and server instructions load, so adding more servers barely touches your context window. It's on by default; Claude uses a `ToolSearch` tool to pull in relevant tools on demand.

| `ENABLE_TOOL_SEARCH` | Behavior |
| --- | --- |
| *(unset)* | All MCP tools deferred, loaded on demand (default; falls back to upfront on Vertex AI / non-first-party `ANTHROPIC_BASE_URL`) |
| `true` | Always defer (sends the beta header even on Vertex/proxies) |
| `auto` | Load upfront if tools fit within 10% of context, defer the overflow |
| `auto:N` | Threshold mode with custom percent (`N` is 0–100), e.g. `auto:5` for 5% |
| `false` | Load all tools upfront, no deferral |

Set the value via the [`settings.json` `env` field](../claude-code/settings.md) for persistence. You can also disable the search tool itself with `{"permissions":{"deny":["ToolSearch"]}}`.

Set `"alwaysLoad": true` on a server (all transports; v2.1.121+) or `"anthropic/alwaysLoad": true` in a tool's `_meta` to exempt it from deferral so it's always visible. Note `alwaysLoad` also **blocks startup** until that server connects (capped at the 5s connect timeout), since its tools must exist when the first prompt is built. Tool Search needs a model that supports `tool_reference` blocks: **Haiku does not**, and on **Vertex AI** it requires Sonnet 4.5+ or Opus 4.5+. Server authors should write clear **server instructions** describing when Claude should search for their tools; Claude Code truncates both tool descriptions and server instructions at **2KB each**, so put critical details first.

> **Output limits:** Claude Code warns when an MCP tool returns >10,000 tokens; the configurable cap is `MAX_MCP_OUTPUT_TOKENS` (**default 25,000**), which applies to tools that don't declare their own limit. A server can raise a specific tool's *text* threshold by setting `_meta["anthropic/maxResultSizeChars"]` in its `tools/list` entry (hard ceiling **500,000 chars**); results above the default threshold are otherwise persisted to disk and replaced with a file reference in the conversation. The annotation does **not** apply to image content — image-returning tools are always bounded by `MAX_MCP_OUTPUT_TOKENS`.

---

## 12. Building a minimal MCP server

A complete stdio tool server in Python with the official SDK (`mcp[cli]`), using FastMCP — type hints and the docstring auto-generate the tool schema:

```python
# weather.py  —  run with: uv run weather.py
import sys
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("weather")

@mcp.tool()
async def get_alerts(state: str) -> str:
    """Get weather alerts for a US state.

    Args:
        state: Two-letter US state code (e.g. CA, NY)
    """
    # ...fetch from an API, format, and return text...
    print(f"fetching alerts for {state}", file=sys.stderr)  # log to STDERR, never stdout
    return f"No active alerts for {state}."

if __name__ == "__main__":
    mcp.run(transport="stdio")   # speak MCP over stdin/stdout
```

Register it in Claude Code (or any host):

```bash
claude mcp add --transport stdio weather -- uv run /abs/path/to/weather.py
```

…or hand-write the host config:

```json
{
  "mcpServers": {
    "weather": { "command": "uv", "args": ["run", "/abs/path/to/weather.py"] }
  }
}
```

In TypeScript the equivalent uses `@modelcontextprotocol/sdk` with `McpServer` + `StdioServerTransport`. To scaffold a server interactively, install the official plugin and run the build skill:

```text
/plugin install mcp-server-dev@claude-plugins-official
/mcp-server-dev:build-mcp-server
```

Test any server with the **MCP Inspector** (`npx @modelcontextprotocol/inspector`).

---

## 13. Security considerations

MCP servers run **with your privileges** and can read data and take actions. Treat every server as code you are installing.

- **Trust the source.** Only add servers you trust. Project `.mcp.json` servers require explicit approval before first use precisely because a committed config could otherwise auto-run on `clone`. Reset with `claude mcp reset-project-choices`.
- **Prompt injection is the headline risk.** Any server that fetches external content (web pages, issues, emails) can return text that tries to hijack Claude into unintended actions. Never let an untrusted server's output silently trigger destructive tools; rely on Claude Code's [permissions](../claude-code/settings.md) and review tool calls.
- **Least privilege on auth.** Pin `oauth.scopes` to the minimum needed; prefer fine-grained tokens; store secrets in the keychain (not in committed config). `headersHelper` runs arbitrary shell — only after workspace trust.
- **Confused-deputy / token passthrough.** Don't let a server forward your tokens to third parties; check that each server's `Authorization` is consumed only by its intended endpoint.
- **Enterprise control.** Admins can centrally allow/deny servers via `managed-mcp.json` (`allowedMcpServers` / `deniedMcpServers`, by name or URL pattern). See [Managed MCP configuration](./connectors.md).
- **Output hygiene.** Large or attacker-controlled outputs can flood context; `MAX_MCP_OUTPUT_TOKENS` and the per-tool size cap bound this.

> The community spec is hardening this area. The next-version RC (locked 2026-05-21, final dated 2026-07-28) tightens OAuth/OIDC alignment — validating the `iss` parameter and declaring `application_type` at client registration — and introduces a **stateless core** so servers can scale behind load balancers without a session store (drops `initialize`/`initialized` and `Mcp-Session-Id`, adds `Mcp-Method`/`Mcp-Name` routing headers). It also adds **MCP Apps** (server-rendered UIs, a new injection surface to review) and a 12-month deprecation policy. WARN: as of 2026-06-26 the final spec is not yet published — verify specifics against `modelcontextprotocol.io` before depending on them.

---

## Related pages

- [Connectors](./connectors.md) — the Claude-apps front end for remote MCP servers, the directory catalog, and admin controls.
- [Plugins](./plugins.md) — bundle MCP servers with commands/skills for one-step team distribution.
- [Skills](./skills.md) — model-invoked capabilities that complement (and can wrap) MCP tools.
- [Settings](../claude-code/settings.md) — `disableClaudeAiConnectors`, permissions, env vars, settings file precedence.
- [Slash commands](../claude-code/slash-commands.md) — how MCP prompts become `/mcp__server__prompt` commands.
- [Hooks](../claude-code/hooks.md) — the `Elicitation` hook for auto-responding to server input requests.
- [CLI and shortcuts](../claude-code/cli-and-shortcuts.md) — full `claude mcp …` subcommand reference and flags.
- [Claude Code overview](../platform/claude-code.md) — the host that exposes the `claude mcp` surface.
- [Capabilities and modes](../models/capabilities-and-modes.md) — tool use, extended thinking, and how tools enter context.
- [Subagents](../claude-code/subagents.md) — the Task tool and how subagents receive plugin/MCP tool names in their `tools` field.

## Open questions / to verify

- The next-version spec (RC locked 2026-05-21, dated 2026-07-28) is **not yet published** as of 2026-06-26 — confirm the final `initialize`-removal, `Mcp-Method`/`Mcp-Name` header semantics, and MCP Apps surface against `modelcontextprotocol.io` once it ships, and confirm when/if Claude Code adopts it.
- Status of the **experimental Tasks** primitive and which hosts (if any) implement it; whether **MCP Apps** ship in Claude Code.
- Whether WebSocket (`type:"ws"`) remains a Claude-Code-only extension or is ever folded into the spec (it is not in 2025-11-25).
- `MAX_MCP_OUTPUT_TOKENS` default is **25,000**; `MCP_TOOL_TIMEOUT` default is **~28 hours** (both confirmed against `code.claude.com/docs/en/mcp`). Re-confirm if Anthropic changes defaults in a later release.
- Precise plan/seat gating for adding custom remote connectors on Team vs Enterprise — cross-check [Connectors](./connectors.md).

## Sources

- Claude Code — Connect Claude Code to tools via MCP: https://code.claude.com/docs/en/mcp
- MCP — Architecture overview: https://modelcontextprotocol.io/docs/concepts/architecture
- MCP — Build an MCP server: https://modelcontextprotocol.io/docs/develop/build-server
- MCP spec 2025-11-25 — Transports: https://modelcontextprotocol.io/specification/2025-11-25/basic/transports
- MCP blog — 2026-07-28 specification release candidate: https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/
- Claude Code — Managed MCP configuration: https://code.claude.com/docs/en/managed-mcp
- Claude Code — Channels: https://code.claude.com/docs/en/channels
