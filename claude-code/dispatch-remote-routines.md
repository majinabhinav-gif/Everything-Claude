---
type: Feature
title: Claude Code — Dispatch, Remote Control & Routines
description: The full remote/cloud execution surface for Claude Code — Dispatch from your phone, Remote Control into a live local session, cloud Routines on a schedule, Claude Code on the web, Channels, and GitHub Actions.
domain: claude-code
tags: [dispatch, remote-control, routines, claude-code-on-the-web, channels, github-actions, cron, cloud-sessions, scheduled-tasks, mobile, teleport, ultraplan]
related: [claude-code-overview, cli-shortcuts, cowork, connectors, slash-commands, settings]
resource: https://code.claude.com/docs/en/remote-control
timestamp: 2026-06-26T00:00:00Z
confidence: high
verified: 2026-06-26
okf_version: "0.1"
sources: [
  "https://code.claude.com/docs/en/remote-control",
  "https://code.claude.com/docs/en/routines",
  "https://code.claude.com/docs/en/claude-code-on-the-web",
  "https://code.claude.com/docs/en/desktop",
  "https://code.claude.com/docs/en/channels",
  "https://code.claude.com/docs/en/github-actions",
  "https://support.claude.com/en/articles/13947068-assign-tasks-from-anywhere-in-claude-cowork",
  "https://platform.claude.com/docs/en/api/claude-code/routines-fire"
]
---

# Claude Code — Dispatch, Remote Control & Routines

Claude Code in 2026 is no longer "the thing that only runs while your terminal is open." A family of features lets you start, steer, and schedule agentic coding work from a phone, a browser, a cron timer, an HTTP webhook, a GitHub event, or a chat app. The crucial distinction across all of them is **where Claude actually runs** — on *your machine* (Dispatch, Remote Control, Channels, Desktop scheduled tasks) versus in *Anthropic-managed cloud infrastructure* (Claude Code on the web, Routines, Slack, GitHub Actions on GitHub runners).

This page documents that whole surface, keeps the lookalikes clearly separated, and gives copy-pasteable examples for each.

## At a glance

| | |
|---|---|
| **What it is** | The remote/cloud execution surface for Claude Code: Dispatch (phone → Desktop), Remote Control (phone/browser → live local session), Routines (cloud cron/API/GitHub agents), Claude Code on the web (cloud sandbox sessions), Channels (push events into a local session), and GitHub Actions (`@claude` in CI). |
| **Where you find it** | CLI (`claude remote-control`, `/remote-control`, `/schedule`, `--channels`, `/install-github-app`), the Desktop app (Cowork → Dispatch; Code → Routines), the web at [claude.ai/code](https://claude.ai/code) and [claude.ai/code/routines](https://claude.ai/code/routines), the Claude mobile apps, and GitHub. |
| **Who can use it by plan** | Remote Control: Pro/Max/Team/Enterprise (off-by-default toggle on Team/Enterprise). Routines & Claude Code on the web: Pro/Max/Team + premium Enterprise seats. Dispatch: Pro/Max only (not Team/Enterprise). GitHub Actions: anyone with the GitHub App + an API key (or Bedrock/Vertex). |
| **Status** | All in **research preview** as of mid-2026 — behavior, limits, flags, and beta headers may change. GitHub Actions (`claude-code-action`) is GA at `@v1`. |

> **One-line mental model.** *Dispatch and Remote Control keep Claude on your computer. Routines, Claude Code on the web, Slack, and GitHub Actions move Claude to the cloud.* Channels also keep it local but let outside systems push events in.

---

## The decision table (read this first)

The official docs ship this comparison; it's the fastest way to pick the right surface. (Source: [remote-control docs](https://code.claude.com/docs/en/remote-control).)

| Feature | What triggers it | Claude runs on | Setup | Best for |
|---|---|---|---|---|
| **Dispatch** | Message a task from the Claude mobile app | Your machine (Desktop) | Pair mobile app with Desktop | Delegating work while away, minimal setup |
| **Remote Control** | Drive a running session from claude.ai/code or the mobile app | Your machine (CLI or VS Code) | `claude remote-control` | Steering in-progress work from another device |
| **Channels** | Push events from Telegram/Discord/iMessage or your own server | Your machine (CLI) | Install a channel plugin | Reacting to external events (CI failures, chat) |
| **Slack** | Mention `@Claude` in a team channel | **Anthropic cloud** | Install the Slack app + Claude Code on the web | PRs and reviews from team chat |
| **Scheduled tasks** | Set a schedule | CLI, Desktop, or **cloud** | Pick a frequency | Recurring automation like daily reviews |
| **Routines** (cloud scheduled tasks) | Schedule / API call / GitHub event | **Anthropic cloud** | Create at claude.ai/code/routines or `/schedule` | Unattended, repeatable, outcome-tied automation |
| **Claude Code on the web** | Submit a task in the browser | **Anthropic cloud** | Connect GitHub, submit task | Async self-contained work, no local setup, parallel runs |

---

## 1) Dispatch — fire a task from your phone, runs on your Desktop

**Dispatch** is a persistent, cross-device conversation that lives in the **Cowork** tab of the Claude **Desktop** app. You message it a task from your phone (or desktop), and it decides how to handle it — executing on *your own computer*, then pushing you a notification when done. It is the lowest-setup way to delegate, but it has a hard dependency: **your machine must stay awake and Claude Desktop must stay running** (no cloud fallback).

> **Surface note:** Dispatch lives in **Cowork**, but a Dispatch task can *spawn a Claude Code session* when the work is software development. See [cowork.md](../platform/cowork.md) for the Cowork/knowledge-work side.

### Where a Dispatch task ends up

A Dispatch task becomes a **Code** session in two ways:

1. **You ask directly** — e.g. *"open a Claude Code session and fix the login bug."*
2. **Dispatch decides** it's development work and spawns one on its own. Bug fixes, dependency updates, running tests, and opening PRs typically route to **Code**; research, document editing, and spreadsheet work stay in **Cowork**.

Either way, the Code session appears in the Code tab's sidebar with a **Dispatch** badge, and you get a push notification on your phone when it finishes or needs approval.

### Set up / pair

| Requirement | Detail |
|---|---|
| Plan | **Pro or Max only** — Dispatch is *not* available on Team or Enterprise |
| Desktop app | Latest version, **macOS or Windows x64**, must be running, computer awake |
| Mobile app | Latest Claude app for iOS/Android, signed into the same account |

Pairing steps (Cowork → Dispatch):

1. Open **Cowork** on phone or desktop.
2. Click **Dispatch** in the left sidebar.
3. Select **Get started**.
4. Toggle permissions for **file access** and **computer wake** settings.
5. Confirm **Finish setup**. The conversation now syncs across devices.

### Example: kick a job from your phone

From the Claude mobile app, inside the Dispatch thread:

```text
Open a Claude Code session in my dotfiles repo, bump the eslint config to flat config,
run the linter, and open a PR. Notify me when it's done or if it needs a decision.
```

Claude routes this to a Desktop **Code** session (Dispatch badge), does the work locally against your real files and connectors, and pushes a notification on completion.

### Computer use + Dispatch caveat

If [computer use](../platform/cowork.md) is enabled, Dispatch-spawned Code sessions can use it too — but **app approvals in those sessions expire after 30 minutes** and re-prompt, rather than lasting the whole session like a normal Code session.

### Dispatch limitations

- Desktop computer must remain active and Claude Desktop must stay open — **no cloud fallback**; if the Mac sleeps or you quit the app, Dispatch stops.
- A single continuous thread — no multiple-thread management.
- Computer use behaves differently from permission-gated file access.

---

## 2) Remote Control — your phone/browser as a window into a *live local* session

**Remote Control** connects [claude.ai/code](https://claude.ai/code), the iOS app, or the Android app to a Claude Code session **running on your machine**. You start a task at your desk and pick it up from the couch — but *nothing moves to the cloud*: your filesystem, MCP servers, tools, and project config stay local. The web/mobile UI is just a window into the local process. (Requires **Claude Code v2.1.51+**; check with `claude --version`.)

> **vs. Claude Code on the web:** same `claude.ai/code` interface, opposite execution model. Remote Control = your machine. Claude Code on the web = Anthropic's cloud. Use Remote Control to *continue* in-progress local work from another device; use the web to *start* a task with no local setup.

### Requirements

- **Plan:** Pro, Max, Team, Enterprise. **API keys are not supported.** On Team/Enterprise an **Owner** must enable the **Remote Control** toggle at [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code) (off by default).
- **Auth:** sign in via `/login` through claude.ai. A `claude setup-token` / `CLAUDE_CODE_OAUTH_TOKEN` "inference-only" token is **not** sufficient — Remote Control needs a full-scope login token.
- **Workspace trust:** run `claude` in the project dir at least once to accept the trust dialog.

### Three CLI invocation modes + VS Code

```bash
# Server mode — stays running, accepts multiple remote connections, shows a QR code
claude remote-control

# Interactive session with Remote Control enabled (type locally AND control remotely)
claude --remote-control            # alias: --rc
claude --remote-control "My Project"

# From inside an existing session — carries over conversation history
/remote-control                    # alias: /rc
/remote-control My Project
```

In the **VS Code** extension (v2.1.79+), type `/remote-control` or `/rc`; a banner shows connection status with an **Open in browser** link. The VS Code command takes no name argument and shows no QR code.

#### Server-mode flags (the rich ones)

| Flag | Description |
|---|---|
| `--name "My Project"` | Custom session title shown in the claude.ai/code session list |
| `--remote-control-session-name-prefix <prefix>` | Prefix for auto-generated names (default: hostname → `myhost-graceful-unicorn`). Env: `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX` |
| `--spawn <mode>` | How the server creates sessions: `same-dir` (default, shared CWD), `worktree` (each session gets its own [git worktree](./cli-and-shortcuts.md)), or `session` (single-session, rejects extra connections). Press `w` at runtime to toggle same-dir/worktree. |
| `--capacity <N>` | Max concurrent sessions (default **32**); cannot combine with `--spawn=session` |
| `--verbose` | Detailed connection/session logs |
| `--sandbox` / `--no-sandbox` | Enable/disable filesystem+network [sandboxing](./settings.md) (off by default) |

> The `--verbose`, `--sandbox`, and `--no-sandbox` flags are **not** available with the `/remote-control` command form.

### Connect from another device

Once a session is active you have three paths:

- **Open the session URL** in any browser → goes straight to the session on claude.ai/code.
- **Scan the QR code** to open it in the Claude app. In `claude remote-control`, press **spacebar** to toggle the QR display.
- **Open claude.ai/code or the app** and find the session by name. In the mobile app, tap **Code** in the nav. Remote Control sessions show a computer icon with a green status dot when online.

Session title precedence: `--name`/`--remote-control`/`/remote-control` arg → `/rename` → last meaningful message → auto-generated `myhost-graceful-unicorn`. As of v2.1.176, auto-titles match the conversation language (or the `language` setting). No app yet? Run `/mobile` to show a download QR.

### Connection status & enabling for all sessions

A `/rc active` indicator sits in the footer while connected (hidden if the terminal is too narrow); selecting it opens a status panel with the URL and QR. To enable Remote Control for *every* interactive session, run `/config` → set **Enable Remote Control for all sessions** to `true` (Desktop: **Settings → Claude Code → Enable remote control by default**).

### Security model

- Local Claude Code makes **outbound HTTPS only** — never opens inbound ports on your machine.
- It registers with the Anthropic API and **polls for work**; messages route between the remote client and your local session over a streaming connection, all over TLS.
- Uses **multiple short-lived credentials**, each scoped to a single purpose and expiring independently.

#### Trusted Devices (Team/Enterprise, beta)

An org-wide setting (off by default; admin-enabled at `claude.ai/admin-settings/claude-code` → **Require trusted devices**) that gates *viewing or steering* Remote Control sessions on **(a)** an enrolled device and **(b)** a sign-in ≤ **18 hours** old, refreshed via Face ID / Touch ID / Windows Hello / passkey. Biometrics run locally; Anthropic stores only the device's **public key** + metadata (name, platform, enrollment time). Members manage devices at [claude.ai/settings/account#trusted-devices](https://claude.ai/settings/account#trusted-devices); admins can **Sign out everywhere** for a lost device.

### Mobile push notifications (v2.1.110+)

When Remote Control is active, Claude can push to your phone — typically when a long task finishes or it needs a decision. Request one in-prompt: `notify me when the tests finish`. Enable in `/config`: **Push when Claude decides** and/or **Push when actions required**. Pushes are skipped while you're typing in the connected terminal; as of v2.1.181, set [`CLAUDE_CLIENT_PRESENCE_FILE`](./settings.md) to a marker path to suppress pushes whenever you're at the machine.

### Which commands work remotely

Interactive pickers (`/plugin`, `/resume`) are **local-only**. Text-output commands work from mobile/web: `/compact`, `/clear`, `/context`, `/usage`, `/exit`, `/usage-credits`, `/recap`, `/reload-plugins`. As of v2.1.166, `/mcp` returns a text status summary (and accepts `reconnect`/`enable`/`disable`); as of v2.1.181, `/config key=value` sets a setting remotely.

### Remote Control limitations

- **One remote session per interactive process** (use server mode for multiple).
- **Local process must keep running** — close the terminal or quit VS Code and the session ends.
- **Network outage > ~10 min** (machine awake but offline) times out the session; re-run `claude remote-control`.
- **Ultraplan disconnects Remote Control** — both occupy the claude.ai/code interface; only one can connect at a time.

---

## 3) Routines — cloud cron/API/GitHub agents that run while you're away

A **Routine** is a **saved Claude Code configuration** — a *prompt*, *one or more GitHub repositories*, an *environment*, and a set of *connectors* — packaged once and run automatically on **Anthropic-managed cloud infrastructure**. They keep working with your laptop closed. Manage them at [claude.ai/code/routines](https://claude.ai/code/routines), the Desktop app, or the CLI `/schedule`. Available on Pro/Max/Team/Enterprise **with Claude Code on the web enabled**; Team/Enterprise Owners can kill-switch all routines with the **Routines** admin toggle.

> Routines belong to your **individual** claude.ai account (not shared with teammates), count against *your* daily run allowance, and act *as you* — commits/PRs carry your GitHub user; connector actions use your linked accounts.

### Triggers (mix and match on one routine)

| Trigger | Fires when |
|---|---|
| **Scheduled** | Recurring cadence (hourly/daily/weekdays/weekly) or a one-off at a future timestamp |
| **API** | An authenticated HTTP `POST` to the routine's per-routine `/fire` endpoint |
| **GitHub** | Repo events — **Pull request** or **Release** actions, with filters |

A single PR-review routine can run nightly, fire from a deploy script, *and* react to every new PR.

### Create from the CLI (`/schedule`)

```text
# Recurring
/schedule daily PR review at 9am

# One-off, natural language (Claude confirms the absolute UTC timestamp)
/schedule tomorrow at 9am, summarize yesterday's merged PRs
/schedule in 2 weeks, open a cleanup PR that removes the feature flag

# Manage existing routines
/schedule list
/schedule update          # change config, or set a custom cron expression
/schedule run             # trigger immediately
```

`/schedule` in the CLI creates **scheduled** routines only — add API or GitHub triggers on the web. Times are entered in your **local zone** and converted automatically (so it runs at that wall-clock time regardless of where the cloud infra is). **Minimum interval is one hour**; sub-hourly cron is rejected. Runs may start a few minutes late due to consistent per-routine **stagger**. One-off runs **auto-disable** after firing (marked **Ran**) and **do not count against the daily routine cap** (they draw down normal subscription usage instead).

> **`/schedule` says "No commands match"?** It's hidden when you're authed via a Console/Bedrock/Vertex/Foundry key, when `ANTHROPIC_API_KEY`/`ANTHROPIC_AUTH_TOKEN`/`apiKeyHelper` is set, when `DISABLE_TELEMETRY`/`DO_NOT_TRACK`/`CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`/`DISABLE_GROWTHBOOK` is set, inside a web session, or on CLI < v2.1.81. Manage at claude.ai/code/routines instead.

### Create from the web (the form fields)

At [claude.ai/code/routines](https://claude.ai/code/routines) → **New routine**:

1. **Name** + **prompt** (the prompt must be *self-contained* — routines run autonomously with **no approval prompts** mid-run). The prompt input includes a **model selector**.
2. **Repositories** — each is cloned from its default branch on every run; Claude creates `claude/`-prefixed branches.
3. **Environment** — a [cloud environment](#4-claude-code-on-the-web--cloud-sandbox-sessions) controlling **network access**, **environment variables** (secrets), and a cached **setup script**. The **Default** env uses **Trusted** network access.
4. **Trigger** — Schedule / GitHub event / API (one or several).
5. **Connectors & Permissions** tabs — all connected MCP connectors included by default (remove what's not needed; Claude can use *any* tool from an included connector, including writes, without asking). Enable **Allow unrestricted branch pushes** per repo to let Claude push beyond `claude/` branches.
6. **Create** — then **Run now** on the detail page to fire immediately.

In the **Desktop app**: **Routines** in the sidebar → **New routine** → choose **Remote** (choosing **Local** instead makes a *Desktop scheduled task* that runs on your machine).

### API trigger — the `/fire` endpoint

Add an API trigger on the web (Edit routine → **Add another trigger** → **API**), **Generate token** (shown once — store it securely), then POST:

```bash
curl -X POST https://api.anthropic.com/v1/claude_code/routines/trig_01ABCDEFGHJKLMNOPQRSTUVW/fire \
  -H "Authorization: Bearer sk-ant-oat01-xxxxx" \
  -H "anthropic-beta: experimental-cc-routine-2026-04-01" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"text": "Sentry alert SEN-4521 fired in prod. Stack trace attached."}'
```

Response:

```json
{
  "type": "routine_fire",
  "claude_code_session_id": "session_01HJKLMNOPQRSTUVWXYZ",
  "claude_code_session_url": "https://claude.ai/code/session_01HJKLMNOPQRSTUVWXYZ"
}
```

The optional `text` field is **freeform** (passed verbatim alongside the saved prompt — JSON in `text` arrives as a literal string) and is capped at **65,536 characters** (exceeding it returns `400 invalid_request_error`). The endpoint ships under the `experimental-cc-routine-2026-04-01` **beta header** (omitting it returns `400`); the two most recent prior header versions keep working for migration. The CLI **cannot** create or revoke API tokens — rotate/revoke from the same web modal (**Regenerate** / **Revoke**; **generating a new token revokes the previous one**). Full reference: [Trigger a routine via API](https://platform.claude.com/docs/en/api/claude-code/routines-fire). (`/fire` is **claude.ai-only**, not part of the Claude Platform API surface — no SDK support, billed to your Claude Code subscription, path namespace `/v1/claude_code/...`.)

> **Path-param gotcha:** the reference calls the path parameter `routine_id`, but the value you paste is **prefixed `trig_`** (e.g. `trig_01ABCDEF…`), not `routine_`. Use exactly the URL the modal shows. The token is prefixed `sk-ant-oat01-`, scoped to that one routine, and grants **no read access** to anything else.

#### `/fire` responses & error codes

| HTTP | `error.type` | Cause |
|---|---|---|
| `200` | — | Session created (returns `claude_code_session_id` + `_url`); does **not** stream output or wait for completion |
| `400` | `invalid_request_error` | Missing/invalid `anthropic-beta` header, `text` > 65,536 chars, or the routine is **paused** |
| `401` | `authentication_error` | No bearer token, or token doesn't match this routine |
| `403` | `permission_error` | Account/org lacks access to the endpoint |
| `404` | `not_found_error` | Routine doesn't exist |
| `429` | `rate_limit_error` | Daily routine-run cap or usage limit hit; response carries a **`Retry-After`** header |
| `500` | `api_error` | Unexpected server error |
| `503` | `overloaded_error` | Temporarily overloaded — retry shortly (note: this endpoint returns **503**, not the Platform's `529`) |

**No idempotency key:** each successful POST creates a *new* session, so a webhook that retries fires the routine multiple times. Guard your callers (e.g. only fire on `if: failure()` once) if duplicate runs are costly.

### GitHub trigger — events + filters

Requires the **Claude GitHub App** installed on the repo (note: `/web-setup` grants cloning access but does **not** install the App or enable webhooks). Configure web-only.

| Event | Triggers when |
|---|---|
| Pull request | Opened, closed, assigned, labeled, synchronized, or otherwise updated |
| Release | Created, published, edited, or deleted |

PR filter fields: **Author, Title, Body, Base branch, Head branch, Labels, Is draft, Is merged** — each paired with an operator (`equals`, `contains`, `starts with`, `is one of`, `is not one of`, `matches regex`). `matches regex` tests the *whole* value, so use `.*hotfix.*` for "contains hotfix." Each matching event starts its **own** session (no session reuse). Webhook events are subject to per-routine and per-account **hourly caps** during the preview.

### Example: a nightly PR-review routine

- **Name:** Nightly review checklist
- **Prompt:** *"Apply our review checklist to PRs merged since yesterday: flag missing tests, check for secrets, leave inline comments, and post a summary to #eng-reviews via the Slack connector."*
- **Repo:** `acme/backend` (default branch)
- **Environment:** Default (Trusted network)
- **Triggers:** Schedule = weekdays 22:00 local; **+** GitHub = `pull_request.opened`, filter `is draft = false`
- **Connectors:** Slack only (remove the rest)

### Example: alert-triage via API

Monitoring tool POSTs to the routine's `/fire` endpoint when an error threshold trips, passing the alert body as `text`. The routine pulls the stack trace, correlates with recent commits, and opens a **draft PR** with a proposed fix plus a link back to the alert.

### Routine usage, limits & management

- Routines draw down **subscription usage** like interactive sessions, plus a **daily cap on runs per account**. See remaining runs at claude.ai/code/routines or [claude.ai/settings/usage](https://claude.ai/settings/usage).
- Beyond the cap, orgs with **usage credits** on can keep running on metered overage (Settings → Billing); without credits, extra runs are rejected until the window resets.
- Reported per-plan daily caps from 2026 write-ups: roughly **~5/day Pro, ~15/day Max, ~25/day Team & Enterprise** — **WARN: verify** against your live limits page; these are community figures, not the official docs.
- **Manage** from the detail page: **Run now**, pause/resume via the **Repeats** toggle, **Edit** (pencil), **Delete** (past sessions remain). A green run status means it *started and exited without infra error* — **not** that the task succeeded; open the run transcript to confirm.

---

## 4) Claude Code on the web — cloud sandbox sessions

**Claude Code on the web** runs tasks on **Anthropic-managed cloud infrastructure** at [claude.ai/code](https://claude.ai/code). Sessions persist if you close the browser, and you can monitor them from the mobile app. This is the engine that Routines and Slack sessions also run on. Research preview for **Pro/Max/Team** and **Enterprise users with premium or Chat + Claude Code seats**.

> **The key contrast with Remote Control:** both use the `claude.ai/code` UI, but the web *executes in the cloud* (no local MCP/tools/filesystem), whereas Remote Control *executes on your machine*. Use the web to start fresh async work, work on an uncloned repo, or run **many tasks in parallel**.

### The cloud environment

Each session runs in an **isolated, Anthropic-managed VM**. An **environment** controls three things:

- **Network access** level (see below)
- **Environment variables** — `.env` format, one `KEY=value` per line, no quotes (quotes become part of the value); use for API keys/secrets
- **Setup script** — installs deps/tools; the result is **cached** so it doesn't re-run each session. Keep it under ~5 min so the cache can build; parallelize with `&`/`wait`; append `|| true` to non-critical commands.

Docker is available (`docker compose up`); pulled images persist in the cached environment (files only — Claude restarts containers each session). Pre-installed runtimes include **Python 3.x** (pip/poetry/uv/ruff/black/mypy/pytest), **Node 20/21/22** via nvm (npm/yarn/pnpm/bun¹/eslint/prettier), **Ruby 3.1–3.3**, **PHP 8.4 + Composer**, **OpenJDK 21 + Maven/Gradle**, **Go**, **Rust**, **C/C++** (gcc/clang/cmake), plus **PostgreSQL 16** and **Redis 7** (not running by default — `service postgresql start` / `service redis-server start`). The `gh` CLI is **not** pre-installed (add it in a setup script + a `GH_TOKEN` env var). Run `check-tools` (cloud-only) for exact versions. ¹Bun has known proxy-compatibility issues for package fetching.

**Resource ceilings** (approximate, may change): **4 vCPUs, 16 GB RAM, 30 GB disk**. Memory-heavy builds/tests may be terminated — for those, run on your own hardware via [Remote Control](#2-remote-control--your-phonebrowser-as-a-window-into-a-live-local-session).

**Setup script vs. SessionStart hook.** A **setup script** is attached to the *environment* (cloud-only), runs as root on Ubuntu 24.04 before Claude launches, and is **cached** (rebuilt only when you change the script or allowed hosts, or after ~7 days). A **SessionStart hook** lives in the repo's `.claude/settings.json`, runs in *both* local and cloud sessions on every start/resume (no caching), and can gate on `CLAUDE_CODE_REMOTE=true` to run cloud-only. Replacing the base image with your own Docker image is **not** supported — install on top of the provided image instead.

**Linking output back to a session.** A cloud session can read its own ID from `CLAUDE_CODE_REMOTE_SESSION_ID` (convert the `cse_` prefix to `session_` to build the transcript URL). As of v2.1.179, Claude's commits carry a `Claude-Session: <url>` git trailer and PR bodies include the session URL; set [`attribution.sessionUrl`](./settings.md) to `false` (v2.1.182+) to omit both.

#### Network access levels

| Level | Outbound connections |
|---|---|
| **None** | No outbound network access |
| **Trusted** *(default)* | Allowlisted domains only: package registries (npm, PyPI, RubyGems, crates.io), GitHub, cloud SDKs, container registries (Docker Hub), common dev domains |
| **Full** | Any domain |
| **Custom** | Your own allowlist (`*.` wildcards), optionally including the defaults |

Blocked hosts fail with `403` + `x-deny-reason: host_not_allowed`. **MCP connector traffic is routed through Anthropic's servers**, so connectors work without adding their hosts to the allowlist. All git operations go through a **GitHub proxy** that translates a scoped in-sandbox credential to your real token and **restricts pushes to the current working branch**.

### GitHub auth & moving sessions

Cloud sessions need GitHub access to clone and push. Grant it via the connect flow or `/web-setup` in the CLI. Session handoff from the CLI is **one-way**:

```bash
# terminal → web: create a NEW cloud session for the current repo (clones the GitHub remote at your
# current branch — push local commits first, the VM clones from GitHub, not your disk). Single repo only.
claude --remote "Fix the authentication bug in src/auth/login.ts"

# run several in parallel — each --remote is its own independent cloud session:
claude --remote "Fix the flaky test in auth.spec.ts"
claude --remote "Update the API documentation"

# web → terminal: pull a cloud session down to continue locally (interactive picker, or pass an ID)
claude --teleport
claude --teleport <session-id>

# set the default env used by --remote:
/remote-env
```

**Four ways to teleport** (all need claude.ai auth — not API key/Bedrock/Vertex/Foundry):

- `claude --teleport` (picker) or `claude --teleport <session-id>` (direct) from the shell — prompts to stash if your tree is dirty.
- `/teleport` (alias `/tp`) from **inside** an open CLI session — same picker without restarting.
- `/tasks` → press **`t`** to teleport into a listed background session.
- From the web UI: **Open in CLI** copies a paste-able command.

`--teleport` requires a **clean working tree** (or stash when prompted), a checkout of the **same repo** (not a fork), the branch **pushed to the remote** (teleport fetches + checks it out for you), and the **same claude.ai account**. It is distinct from `--resume` (which only reopens *local* history and never lists cloud sessions). Internally `--teleport` rides the same Remote Control session infra, so expiry/auth failures can surface with Remote Control wording (`Remote Control session expired`, `Access denied`) — fix with `/login`.

> **No-GitHub fallback:** running `claude --remote` from a repo not connected to GitHub **bundles** the local repo (full history across branches + uncommitted tracked changes) and uploads it. Force it with `CCR_FORCE_BUNDLE=1`. Bundles must be a git repo with ≥1 commit and **under 100 MB** (larger falls back to current-branch-only, then a squashed snapshot, then fails); **untracked files aren't included** (`git add` first); and bundle-created sessions **can't push back** unless GitHub auth is also configured.

> **Push an *existing* terminal session to the web:** use the Desktop app's **Continue in → Claude Code on the Web** menu — requires a clean tree, not available for SSH sessions. (CLI handoff is one-way; there is no `--remote` for an already-running local session.)

### Managing all your sessions in one place ("Agent View")

Every coding session — local Remote Control, cloud web tasks, Routine runs, Slack/Dispatch-spawned sessions — appears in the **session list/sidebar at claude.ai/code** (and under **Code** in the mobile nav). From there you can:

- **Review changes** — each session shows a diff indicator like `+42 -18`; open it, leave inline comments, and send them to Claude.
- **Share** — Enterprise/Team toggle **Private**/**Team** (Team = visible to your org, with repo-access verification on by default); Max/Pro toggle **Private**/**Public** (Public = visible to *any* logged-in claude.ai user, repo-access verification **off** by default — check for credentials before sharing). [Claude in Slack](https://code.claude.com/docs/en/slack) sessions auto-share with Team visibility. Tighten via **Settings → Claude Code → Sharing settings**.
- **Archive** (hover → archive icon) and **Delete** (filter archived → delete, or session dropdown → **Delete**, with confirmation).

> "Agent View" is the informal name for this unified session manager. **WARN: verify** the exact in-product label — the docs refer to it simply as the session list/sidebar at claude.ai/code.

### Auto-fix pull requests

Claude can watch a PR and respond automatically to **CI failures** and **review comments** (requires the Claude GitHub App). Turn it on via the CI status bar **Auto-fix** toggle in a web session, `/autofix-pr` from the PR's branch in the terminal (spawns a web session), the mobile app ("watch this PR and fix any CI failures or review comments"), or by pasting a PR URL. It's a **per-PR toggle**.

### Cloud session limitations

- **Rate limits:** shares your account's Claude/Claude Code limits; parallel tasks consume proportionately more. **No separate compute charge** for the VM.
- **Platform:** cloning + PR creation require **GitHub** (GitHub Enterprise Server supported for Team/Enterprise). GitLab/Bitbucket can be sent as a **local bundle** but can't push back.
- **IP allowlisting:** cloud sessions call the API from Anthropic infra, so org **IP allowlisting** breaks them (same for Code Review and Routines) — contact support to exempt Anthropic-hosted services.
- **Environment expiry:** sessions stop after inactivity and the env is reclaimed; reopen from claude.ai/code to re-provision with history restored.

---

## 5) Channels — push external events into a *running local* session

A **channel** is an **MCP server that pushes events into your already-open local Claude Code session**, so Claude reacts to things that happen while you're away — a CI webhook, a chat message, a monitoring alert. Channels can be two-way (Claude reads the event and replies through the same channel, like a chat bridge). Events only arrive while the session is **open**, so for always-on use run Claude in a background/persistent terminal. Requires **v2.1.80+** and Anthropic auth (claude.ai or Console key); **not** on Bedrock/Vertex/Foundry. Team/Enterprise must enable `channelsEnabled`.

> **vs. Remote Control / Dispatch:** Remote Control = *you* drive a local session remotely. Channels = an *external system* pushes events into a local session. Both keep execution local; only Routines/web/Slack go to the cloud.

Bundled channels (each a plugin needing [Bun](https://bun.sh)): **Telegram, Discord, iMessage**, plus a **fakechat** localhost demo.

### Telegram example (the shape of all of them)

```text
# 1. /newbot in BotFather → copy token
# 2. install + activate
/plugin install telegram@claude-plugins-official
/reload-plugins
# 3. configure (saved to ~/.claude/channels/telegram/.env)
/telegram:configure <token>
```

```bash
# 4. restart with the channel enabled (space-separate multiple plugins)
claude --channels plugin:telegram@claude-plugins-official
```

```text
# 5. message the bot → it returns a pairing code; then lock it down:
/telegram:access pair <code>
/telegram:access policy allowlist
```

An inbound message arrives in the session as a `<channel source="telegram">` event; Claude does the work and calls the channel's `reply` tool (you see "sent" in the terminal; the actual reply appears on the platform). Every channel keeps a **sender allowlist**; a server must be both in `.mcp.json` *and* named in `--channels` to push. For unattended runs, [`--dangerously-skip-permissions`](./settings.md) bypasses prompts other than explicit ask rules (trusted environments only); channels declaring the permission-relay capability can forward approval prompts to you remotely (anyone who can reply can then approve/deny — only allowlist senders you trust with that authority). In non-interactive `-p` mode, terminal-input tools (multiple-choice, plan-mode approval) are disabled so the session never stalls. Enterprise admins gate availability with `channelsEnabled` (master switch; Owner-enabled at the admin console — blocked by default on Team/Enterprise, allowed by default on Console-with-API-key) and restrict plugins with `allowedChannelPlugins` (when set, **replaces** Anthropic's allowlist entirely). Build your own via the Channels reference and test it with `--dangerously-load-development-channels`. See also [plugins](../capabilities/plugins.md) and [mcp](../capabilities/mcp.md).

> **Discord & iMessage differ from Telegram.** Discord needs **Message Content Intent** + bot scopes (View Channels, Send Messages, Read Message History, Attach Files, Add Reactions), then the same `/discord:configure <token>` → DM-to-pair → `/discord:access policy allowlist` flow. **iMessage** (macOS-only) needs **no token**: it reads `~/Library/Messages/chat.db` (requires **Full Disk Access**) and replies via AppleScript. Texting *yourself* bypasses the gate automatically; add other contacts with `/imessage:access allow +15551234567` (phone in `+country` format, or an Apple ID email).

---

## 6) GitHub Actions / `@claude` — Claude Code in CI

Claude Code GitHub Actions brings `@claude` into your GitHub workflow. Mention **`@claude`** in any PR or issue comment and it analyzes code, implements features, fixes bugs, and opens PRs — running on **GitHub-hosted runners** (your code stays on GitHub). Built on the [Claude Agent SDK](../platform/claude-code.md); the action is `anthropics/claude-code-action@v1` (GA).

### Quick setup

```text
/install-github-app
```

Run it in the Claude Code terminal — it installs the **Claude GitHub App** (requesting read & write on Contents, Issues, Pull requests) and walks you through workflow + `ANTHROPIC_API_KEY` secret setup. (v2.1.187+ offers **Skip for now** to stop after just installing the App.) You must be a repo admin. Manual path: install [github.com/apps/claude](https://github.com/apps/claude), add the `ANTHROPIC_API_KEY` secret, copy the example `claude.yml` into `.github/workflows/`.

### Minimal workflow — respond to `@claude`

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # auto-detects "tag" mode and responds to @claude mentions
```

In a comment: `@claude implement this feature based on the issue description` / `@claude fix the TypeError in the user dashboard component`.

### PR-review action (headless, runs a skill on every PR)

```yaml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          plugin_marketplaces: "https://github.com/anthropics/claude-code.git"
          plugins: "code-review@claude-code-plugins"
          prompt: "/code-review:code-review ${{ github.repository }}/pull/${{ github.event.pull_request.number }}"
```

### Key v1 inputs

| Parameter | Description |
|---|---|
| `prompt` | Instructions (plain text or a [skill](../capabilities/skills.md) name); **optional** — omit for comment events to respond to the trigger phrase |
| `claude_args` | Pass-through CLI args, e.g. `--max-turns 5 --model claude-sonnet-4-6 --mcp-config /path/config.json --allowedTools "..."` (default `--max-turns` is 10) |
| `trigger_phrase` | Custom trigger (default `@claude`) |
| `anthropic_api_key` | Required for direct API (not for Bedrock/Vertex) |
| `github_token` | GitHub token for API access (e.g. from `actions/create-github-app-token` when using a custom App) |
| `use_bedrock` / `use_vertex` | Use Amazon Bedrock / Google Vertex AI instead |
| `plugin_marketplaces` / `plugins` | Newline-separated marketplace Git URLs / plugin names to install before execution |
| `settings` | JSON settings (replaces the old `claude_env` for passing env/config) |

> **v1 vs beta migration:** change `@beta`→`@v1`; drop `mode:` (auto-detected); `direct_prompt`→`prompt`; `override_prompt`→`prompt` with GitHub variables; and move the rest into `claude_args` — `custom_instructions`→`--append-system-prompt`, `max_turns`→`--max-turns`, `model`→`--model`, `allowed_tools`→`--allowedTools`, `disallowed_tools`→`--disallowedTools` (and `claude_env`→the `settings` JSON input). **Costs:** GitHub Actions minutes (runner) **+** API tokens — control with `--max-turns`, workflow timeouts, and concurrency limits. Bedrock model IDs carry a region prefix, e.g. `us.anthropic.claude-sonnet-4-6` (Vertex uses dated IDs like `claude-sonnet-4-5@20250929`). The `--model opus` shorthand also works. See [model-families](../models/model-families.md) for current IDs.

---

## Quick cross-feature cheat sheet

| You want to… | Use | Command / entry point |
|---|---|---|
| Delegate from your phone, run on your computer, minimal setup | **Dispatch** | Cowork → Dispatch (mobile message) |
| Continue a *live local* session from phone/browser | **Remote Control** | `claude remote-control` / `/rc` |
| Run a job in the cloud on a schedule/API/GitHub event | **Routines** | `/schedule` or claude.ai/code/routines |
| Start a fresh cloud task with no local setup | **Claude Code on the web** | claude.ai/code or `claude --remote "…"` |
| Pull a cloud session down to your terminal | **Teleport** | `claude --teleport`, `/teleport` (`/tp`), or `/tasks` → `t` |
| React to a CI webhook / chat message in an open local session | **Channels** | `claude --channels plugin:…` |
| `@claude` in PRs/issues, or headless PR review in CI | **GitHub Actions** | `/install-github-app`, `claude-code-action@v1` |

---

## Related pages

- [../platform/claude-code.md](../platform/claude-code.md) — Claude Code overview (CLI/IDE/desktop/web)
- [./cli-and-shortcuts.md](./cli-and-shortcuts.md) — CLI flags (`--remote-control`, `--remote`, `--teleport`, `--channels`) + keyboard shortcuts + worktrees
- [./slash-commands.md](./slash-commands.md) — `/remote-control`, `/schedule`, `/teleport` (`/tp`), `/tasks`, `/autofix-pr`, `/remote-env`, `/mobile`, `/web-setup`, `/install-github-app`
- [./settings.md](./settings.md) — `settings.json`, managed settings (`disableRemoteControl`, `channelsEnabled`, `allowedChannelPlugins`), env vars, sandboxing
- [../platform/cowork.md](../platform/cowork.md) — Cowork desktop agent (home of Dispatch)
- [../capabilities/connectors.md](../capabilities/connectors.md) — connectors used by Routines and cloud sessions
- [../capabilities/mcp.md](../capabilities/mcp.md) — MCP (channels are MCP servers)
- [../capabilities/plugins.md](../capabilities/plugins.md) — plugins (channel plugins, marketplace installs)
- [../models/model-families.md](../models/model-families.md) — model IDs referenced in actions/routines

## Open questions / to verify

- **Exact per-plan daily Routine run caps** — community write-ups say ~5/15/25 (Pro/Max/Team-Enterprise) but the official docs only point to the live usage page. Treat the numbers as unverified.
- **The in-product name "Agent View"** — the docs describe a unified session list/sidebar at claude.ai/code but I did not find that exact label; confirm whether Anthropic uses "Agent View" officially.
- **GitHub trigger hourly caps** — docs say per-routine and per-account hourly caps exist during the preview but don't publish the numbers ("see your current limits at claude.ai/code/routines").
- **VM isolation technology** — docs say "isolated, Anthropic-managed VM" without naming the sandbox tech (gVisor/Firecracker/microVM); not stated.
- **Whether Dispatch will reach Team/Enterprise** — currently Pro/Max-only; status may change.
- **The per-account daily routine-run number** — the `/fire` API reference confirms the daily allowance "varies by plan" and that `429`s carry a `Retry-After`, but still publishes no integer; the `text` field cap (65,536 chars) is now confirmed.

## Sources

- [Continue local sessions with Remote Control — code.claude.com/docs/en/remote-control](https://code.claude.com/docs/en/remote-control)
- [Automate work with routines — code.claude.com/docs/en/routines](https://code.claude.com/docs/en/routines)
- [Use Claude Code on the web — code.claude.com/docs/en/claude-code-on-the-web](https://code.claude.com/docs/en/claude-code-on-the-web)
- [Desktop application (Dispatch sessions) — code.claude.com/docs/en/desktop](https://code.claude.com/docs/en/desktop)
- [Push events into a session with channels — code.claude.com/docs/en/channels](https://code.claude.com/docs/en/channels)
- [Claude Code GitHub Actions — code.claude.com/docs/en/github-actions](https://code.claude.com/docs/en/github-actions)
- [Assign tasks from anywhere in Claude Cowork (Dispatch) — support.claude.com](https://support.claude.com/en/articles/13947068-assign-tasks-from-anywhere-in-claude-cowork)
- [Trigger a routine via API (full reference: error table, 65,536-char `text` cap, no idempotency) — platform.claude.com/docs/en/api/claude-code/routines-fire](https://platform.claude.com/docs/en/api/claude-code/routines-fire)
- [Claude in Slack (spawns a cloud web session) — code.claude.com/docs/en/slack](https://code.claude.com/docs/en/slack)
