---
type: Surface
title: Claude Code — Overview
description: A terminal-native, agentic coding tool from Anthropic that reads your codebase, edits files, runs commands, and works across the CLI, IDEs, desktop, web, and mobile.
domain: platform
tags: [claude-code, cli, agentic-coding, terminal, ide, claude-md, permissions, tools, anthropic]
related: [claude-ai, model-families, capabilities-and-modes, slash-commands, settings, hooks, cli-shortcuts, subagents, dispatch-remote, skills, plugins, mcp, connectors, artifacts]
resource: https://code.claude.com/docs/en/overview
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://code.claude.com/docs/en/overview
  - https://code.claude.com/docs/en/setup
  - https://code.claude.com/docs/en/tools-reference
  - https://code.claude.com/docs/en/permission-modes
  - https://code.claude.com/docs/en/memory
  - https://code.claude.com/docs/en/authentication
  - https://claude.com/product/claude-code
---

# Claude Code — Overview

Claude Code is Anthropic's agentic coding tool: an AI agent that reads your whole codebase, edits files, runs shell commands, and integrates with your dev tooling, then loops on the results until a task is done. It started as a terminal CLI and now spans the terminal, VS Code/JetBrains IDEs, a desktop app, the browser, and mobile — all backed by the same engine, so your `CLAUDE.md` files, settings, and MCP servers carry across surfaces. This page is the navigational map: it covers install, surfaces, auth, the agent loop, built-in tools, permission modes, and memory, then links out to the deep-reference pages for each subsystem.

**At a glance**

- **What it is:** A terminal-native, agentic coding agent that plans, edits across many files, runs commands/tests, and uses tools (search, web, git, MCP) in an autonomous loop — composable in the Unix sense (`tail -f app.log | claude -p "..."`).
- **Where you find it:** Terminal CLI (`claude`), VS Code extension (incl. Cursor), JetBrains plugin, standalone Desktop app, the web at [claude.ai/code](https://claude.ai/code), and the Claude iOS app / Remote Control on mobile.
- **Who can use it by plan:** Requires a paid account — **Claude Pro, Max, Team, or Enterprise** (claude.ai subscription) or a **Claude Console** (API-billed) account; or a third-party provider (**Amazon Bedrock**, **Google Vertex AI**, **Microsoft Foundry**). The free Claude.ai plan does **not** include Claude Code.
- **Status:** GA and actively shipping (rapid release cadence; the 2.1.x series as of mid-2026 — confirm with `claude --version` / release notes, WARN: exact current build floats). Some features noted below are research previews — flagged inline.

---

## What Claude Code is (and isn't)

Claude Code is an **agent**, not an autocomplete. You describe an outcome in natural language; Claude plans an approach, reads the relevant files, makes edits across the codebase, runs commands to verify (tests, linters, builds), and iterates until the goal is met or it needs your input. It works directly with `git` (staging, commit messages, branches, PRs) and can reach external systems through the Model Context Protocol (MCP).

What it is good for, with concrete invocations:

```bash
# One-shot task in the current repo
claude "write tests for the auth module, run them, and fix any failures"

# Git workflow
claude "commit my changes with a descriptive message"

# Unix pipe / headless mode (-p = print, non-interactive)
tail -200 app.log | claude -p "Slack me if you see any anomalies"

# Bulk operation in CI
git diff main --name-only | claude -p "review these changed files for security issues"
```

It is **not** a chat-only assistant: it has filesystem and shell access (gated by permissions), maintains its own task list, and can spawn subagents and agent teams to parallelize work.

---

## Surfaces (where Claude Code runs)

Every surface connects to the same Claude Code engine. Pick by where your code lives and how you want to review changes.

| Surface | How to get it | Highlights |
| :--- | :--- | :--- |
| **Terminal CLI** | `claude` after install | Full feature set; the reference surface for everything in these docs |
| **VS Code extension** | Extensions view → "Claude Code", `vscode:extension/anthropic.claude-code`, or `cursor:extension/anthropic.claude-code`; also Cursor | Inline diffs, `@`-mentions, plan review, conversation history; **Open in New Tab** from the Command Palette |
| **JetBrains plugin** | JetBrains Marketplace (IntelliJ, PyCharm, WebStorm, …) — still **beta**-labeled in the marketplace; requires the CLI installed separately | Runs Claude Code in the IDE terminal; interactive diff viewer, selection-context sharing |
| **Desktop app** | Download for macOS (universal) / Windows (x64) / Windows ARM64; launch Claude → **Code** tab | Visual diff review, multiple sessions side by side, scheduled tasks, kick off cloud sessions; paid subscription required |
| **Web** | [claude.ai/code](https://claude.ai/code) — no local setup | Long-running cloud tasks, work on repos you don't have locally, run tasks in parallel; on desktop browsers + the Claude iOS app |
| **Mobile / Remote** | Claude iOS app; **Remote Control** from a local session | Start on one device, continue on another; message **Dispatch** a task from your phone |

Sessions are portable across surfaces:

```bash
# Hand a terminal session to the Desktop app for visual diff review
/desktop

# Pull a web/iOS session into your terminal (requires a claude.ai subscription)
claude --teleport
```

You can also drive Claude Code from **Slack** (`@Claude` a bug → get a PR), **GitHub Actions / GitLab CI/CD**, **GitHub Code Review** on every PR, **Chrome** for live web-app debugging, and **Channels** (push events from Telegram, Discord, iMessage, or your own webhooks into a session). For fully custom orchestration there's the **Agent SDK** (build your own agents on Claude Code's tools). See `../claude-code/dispatch-remote-routines.md` for the remote/mobile/scheduled surfaces and `./claude-ai.md` for the consumer apps.

---

## Install

Claude Code ships as a single native binary. The native installer is recommended and **auto-updates in the background**; package-manager installs (Homebrew, WinGet, apt/dnf/apk, npm) do not auto-update by default.

### Native install (recommended)

```bash
# macOS, Linux, WSL
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell
irm https://claude.ai/install.ps1 | iex

# Windows CMD
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

Pin a channel or version at install time (the channel you choose becomes your auto-update default):

```bash
curl -fsSL https://claude.ai/install.sh | bash -s stable     # stable channel (~1 week behind)
curl -fsSL https://claude.ai/install.sh | bash -s 2.1.89      # a specific version  (WARN: verify the exact build you want)
```

On Windows PowerShell the equivalent passes the argument to the downloaded script:

```powershell
& ([scriptblock]::Create((irm https://claude.ai/install.ps1))) stable   # or a version like 2.1.89
```

### Other installers

```bash
brew install --cask claude-code         # macOS/Linux; `claude-code@latest` cask tracks the latest channel
winget install Anthropic.ClaudeCode     # Windows
npm install -g @anthropic-ai/claude-code # cross-platform; requires Node.js 18+
```

The npm package installs the **same native binary** via a per-platform optional dependency (e.g. `@anthropic-ai/claude-code-darwin-arm64`); the `claude` binary does not invoke Node at runtime. Upgrade npm installs with `npm install -g @anthropic-ai/claude-code@latest` (avoid `npm update -g`). **Do not** use `sudo npm install -g`. Linux package managers (apt/dnf/apk) are also published with signed repositories.

### System requirements

| Requirement | Detail |
| :--- | :--- |
| **OS** | macOS 13.0+, Windows 10 1809+/Server 2019+, Ubuntu 20.04+, Debian 10+, Alpine 3.19+ |
| **Hardware** | 4 GB+ RAM, x64 or ARM64 |
| **Shell** | Bash, Zsh, PowerShell, or CMD |
| **Node.js** | Only for the npm install path: **Node 18+** |
| **Network** | Internet connection required; must be in an [Anthropic-supported country](https://www.anthropic.com/supported-countries) |
| **Search** | `ripgrep` (bundled; on Alpine/musl install `libgcc libstdc++ ripgrep` and set `USE_BUILTIN_RIPGREP=0`) |

On **native Windows**, installing [Git for Windows](https://git-scm.com/downloads/win) is optional but recommended: it enables the Bash tool via Git Bash (point at it explicitly with `"env": { "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe" }` if it isn't found). Without it, Claude Code uses the **PowerShell tool** instead. **Sandboxing is not supported on native Windows** — WSL 2 is the path to sandboxed command execution (WSL 1 also runs Claude Code but without sandboxing).

### Verify and update

```bash
claude --version          # confirm install
claude doctor             # detailed health check + last update result
claude update             # apply an update immediately (native installs auto-update in background)
```

Control updates with settings: `"autoUpdatesChannel": "latest" | "stable"`, `"minimumVersion": "2.1.100"` (a floor that prevents downgrades), or disable the background check with `DISABLE_AUTOUPDATER=1` (use `DISABLE_UPDATES=1` to block manual updates too). For a hard start/stop range, managed settings add `requiredMinimumVersion`/`requiredMaximumVersion`. To let Claude Code run the package-manager upgrade for you (Homebrew/WinGet), set `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE=1`. Manifest signatures (verifiable with the Anthropic GPG key, fingerprint `31DD DE24 DDFA B679 F42D 7BD2 BAA9 29FF 1A7E CACE`) ship for releases from `2.1.89` onward. Full install matrix, signed-binary verification, and uninstall steps live on the [Advanced setup](https://code.claude.com/docs/en/setup) page.

---

## Authentication

Run `claude` in a project; on first launch it opens a browser to log in. Use `/login` to authenticate or switch accounts, `/logout` to sign out, and `/status` to see which method is active.

| Account type | How |
| :--- | :--- |
| **Claude Pro / Max** | Log in with your Claude.ai account (subscription OAuth) |
| **Claude for Teams / Enterprise** | Log in with the Claude.ai account your admin invited; Enterprise adds SSO, domain capture, RBAC, compliance API, managed policy settings |
| **Claude Console (API-billed)** | Log in with Console credentials; admin assigns a **Claude Code** role (can make Claude Code API keys) or **Developer** role |
| **Amazon Bedrock** | Set `CLAUDE_CODE_USE_BEDROCK=1` + provider creds; no browser login |
| **Google Vertex AI** | Set `CLAUDE_CODE_USE_VERTEX=1` + provider creds |
| **Microsoft Foundry** | Set `CLAUDE_CODE_USE_FOUNDRY=1` + provider creds |

**Credential precedence** (first match wins): cloud-provider creds (`CLAUDE_CODE_USE_BEDROCK`/`USE_VERTEX`/`USE_FOUNDRY`) → `ANTHROPIC_AUTH_TOKEN` (Bearer, for gateways) → `ANTHROPIC_API_KEY` (X-Api-Key, direct API) → `apiKeyHelper` script → `CLAUDE_CODE_OAUTH_TOKEN` → subscription OAuth from `/login`. A stray `ANTHROPIC_API_KEY` in your env can override your subscription once approved — `unset ANTHROPIC_API_KEY` to fall back, and check `/status`. (`apiKeyHelper` is re-invoked after 5 min or on HTTP 401; tune with `CLAUDE_CODE_API_KEY_HELPER_TTL_MS`. Claude Desktop and cloud sessions ignore `apiKeyHelper`/`ANTHROPIC_API_KEY`/`ANTHROPIC_AUTH_TOKEN` and always use OAuth; **Claude Code on the web always uses subscription credentials**.)

For CI/headless where browser login isn't available, mint a one-year token:

```bash
claude setup-token                       # walks OAuth, prints a token (not saved)
export CLAUDE_CODE_OAUTH_TOKEN=your-token # use in CI; requires Pro/Max/Team/Enterprise
```

That token is **scoped to inference only** — it can't establish [Remote Control](../claude-code/dispatch-remote-routines.md) sessions, and **bare mode** (`--bare`) doesn't read it (use `ANTHROPIC_API_KEY` or `apiKeyHelper` there instead).

Credentials are stored in the macOS Keychain, or `~/.claude/.credentials.json` (mode `0600`) on Linux, or `%USERPROFILE%\.claude\.credentials.json` on Windows (inheriting your profile's ACLs); set `CLAUDE_CONFIG_DIR` to relocate the file on Linux/Windows. See `../models/model-families.md` for model IDs/pricing and `./claude-ai.md` for plan details.

---

## The agent loop

Each turn, Claude Code runs an iterative loop: **read context → reason/plan → call a tool → observe the result → repeat**, stopping to ask you only when a permission gate or an explicit question requires it. It maintains a structured task list (via the `TaskCreate`/`TaskList`/`TaskUpdate` tools) so multi-step work survives across turns, and it can delegate to **subagents** (separate context windows) or **agent teams** for parallel work. Concretely a "fix this bug" request might: `Grep` for the symptom → `Read` the suspect files → form a plan → `Edit` the fix → `Bash` to run the tests → read the failure → `Edit` again → `Bash` to commit.

Key loop mechanics worth knowing:

- **Context window**: starts fresh each session; `CLAUDE.md` and auto memory are re-injected at launch (see [Memory](#memory-claudemd-and-auto-memory)). `/compact` summarizes when context fills; project-root `CLAUDE.md` is re-read after compaction.
- **Background work**: long-running commands can run with `run_in_background: true`; manage them with `/tasks`. The `Monitor` tool watches logs/CI and interjects when something changes.
- **Plan mode**: lets Claude research and propose before touching source (see below).

---

## Built-in tools

These are the tools Claude calls during the loop. The names below are the exact strings used in permission rules, subagent `tools` lists, and hook matchers. "Permission required" means the action prompts unless your mode/rules pre-approve it.

| Tool | What it does | Prompts? |
| :--- | :--- | :--- |
| `Read` | Reads file contents (text, images, PDFs, `.ipynb`) with line numbers; absolute paths | No |
| `Edit` | Exact-string replacement in a file (read-before-edit + uniqueness checks) | Yes |
| `Write` | Creates or overwrites a whole file (no append/merge) | Yes |
| `Bash` | Runs shell commands (2-min default timeout, up to 10 min; 30k-char output cap) | Yes |
| `PowerShell` | Runs PowerShell natively (Windows default w/o Git Bash; opt-in elsewhere) | Yes |
| `Glob` | Finds files by name pattern (`**/*.ts`); sorted by mtime, capped at 100 | No |
| `Grep` | Searches file contents via ripgrep regex; modes `files_with_matches`/`content`/`count` | No |
| `WebSearch` | Queries Anthropic's web-search backend; returns titles + URLs | Yes |
| `WebFetch` | Fetches a URL, converts to Markdown, runs your extraction prompt (lossy by design) | Yes |
| `Agent` | Spawns a subagent with its own context window to handle a delegated task (also launches forked subagents in fork mode) | No |
| `AskUserQuestion` | Asks multiple-choice questions to gather requirements or clarify ambiguity | No |
| `Skill` | Runs an Agent Skill (`SKILL.md`) inside the conversation | Yes |
| `TaskCreate` / `TaskList` / `TaskUpdate` / `TaskGet` | Manage the session task list (default task system as of v2.1.142) | No |
| `TaskStop` | Kills a running background task by ID | No |
| `TaskOutput` | **Deprecated** — retrieves background-task output; prefer `Read` on the output file path | No |
| `TodoWrite` | Legacy checklist tool; disabled by default since v2.1.142 (re-enable, counter-intuitively, with `CLAUDE_CODE_ENABLE_TASKS=0`) | No |
| `NotebookEdit` | Edits Jupyter notebook cells by `cell_id` (`replace`/`insert`/`delete`); rules use the `Edit(...)` path format | Yes |
| `LSP` | Language-server intelligence: definitions, references, type errors (needs a code-intelligence plugin) | No |
| `Monitor` | Runs a command in the background and feeds each output line back to Claude (shares Bash permission rules; not on Bedrock/Vertex/Foundry) | Yes |
| `EnterPlanMode` / `ExitPlanMode` | Enter/leave plan mode | No / Yes |
| `EnterWorktree` / `ExitWorktree` | Create/switch isolated git worktrees (stored under `.claude/worktrees/`) | No |
| `ListMcpResourcesTool` / `ReadMcpResourceTool` | Discover and read MCP server resources | No |
| `CronCreate` / `CronList` / `CronDelete` | Session-scoped scheduled prompts (restored on `--resume`/`--continue`) | No |
| `ScheduleWakeup` | Reschedules the next iteration of a self-paced `/loop` (Claude calls it, not you) | No |
| `RemoteTrigger` | Create/update/run/list Routines on claude.ai (backs `/schedule`; Pro/Max/Team/Enterprise, Anthropic-hosted) | No |
| `SendMessage` | Message an agent-team teammate or resume a subagent by agent ID | No |
| `ShareOnboardingGuide` | Uploads `ONBOARDING.md` and returns a share link (backs `/team-onboarding`; Pro/Max/Team/Enterprise) | Yes |
| `PushNotification` | Desktop + phone push when a long task finishes (Anthropic-hosted; not on Bedrock/Vertex/Foundry) | No |
| `Artifact` | Publishes HTML/Markdown as a shareable claude.ai artifact (Team/Enterprise + `/login`) | Yes |
| `Workflow` | Runs a dynamic workflow that orchestrates many subagents in the background | Yes |
| `ToolSearch` / `WaitForMcpServers` | Load deferred MCP tools / wait on still-connecting MCP servers (tool-search infra) | No |

Add your own tools by connecting an **MCP server** (`../capabilities/mcp.md`); package reusable prompt workflows as **Skills** (`../capabilities/skills.md`) which run through the existing `Skill` tool rather than adding new tool entries. Disable any tool by adding its name to `permissions.deny`. The full per-tool behavior reference is at [tools-reference](https://code.claude.com/docs/en/tools-reference); subagents are covered in `../claude-code/subagents.md`.

---

## Permission modes

Permission modes set the baseline for how often Claude pauses to ask. Cycle them in the CLI with **Shift+Tab** (`default → acceptEdits → plan`); the current mode shows in the status bar. Enabled optional modes slot in **after `plan`**, with `bypassPermissions` first and `auto` last (so with both on you pass through `bypassPermissions` on the way to `auto`). `dontAsk` **never** appears in the cycle — set it only with `--permission-mode dontAsk`. In VS Code/Desktop/web use the mode selector; the web/cloud dropdown shows **Accept edits / Plan / Auto** (Accept edits maps to `default` because the cloud env pre-approves edits), and `bypassPermissions`/`dontAsk` from settings files are ignored for cloud sessions.

| Mode | Runs without asking | Best for |
| :--- | :--- | :--- |
| `default` | Reads only | Getting started, sensitive work |
| `acceptEdits` | Reads, file edits, and common filesystem commands (`mkdir`, `touch`, `rm`, `rmdir`, `mv`, `cp`, `sed`) in-scope | Iterating on code you review after the fact |
| `plan` | Reads only — researches and proposes, never edits source | Exploring a codebase before changing it |
| `auto` | Everything, with a background safety classifier per action | Long tasks, reducing prompt fatigue (research preview) |
| `dontAsk` | Only pre-approved (`allow`-rule) tools; everything else auto-denied | Locked-down CI/scripts |
| `bypassPermissions` | Everything; skips checks (the "yolo" mode) | **Isolated containers/VMs only** |

Set the mode at startup, persistently, or mid-session:

```bash
claude --permission-mode plan                 # start in plan mode
claude --permission-mode acceptEdits          # start auto-accepting edits
claude --permission-mode dontAsk              # locked-down: only allow-rule + read-only commands run
claude --dangerously-skip-permissions         # == --permission-mode bypassPermissions
# --allow-dangerously-skip-permissions adds bypass to the Shift+Tab cycle without activating it
```

```json
// .claude/settings.json — make a mode the default
{ "permissions": { "defaultMode": "acceptEdits" } }
```

Notes that bite people:

- **Protected paths** — writes to a fixed set of sensitive paths are never auto-approved (in `default`/`acceptEdits`/`plan` they prompt; under `auto` they route to the classifier even with a matching `allow` rule; under `dontAsk` they're denied). They include directories `.git`, `.config/git`, `.vscode`, `.idea`, `.husky`, `.cargo`, `.devcontainer`, `.yarn`, `.mvn`, and `.claude` (except `.claude/worktrees`); plus files like `.gitconfig`/`.gitmodules`, every shell rc (`.bashrc`, `.zshrc`, `.profile`, `.envrc`, …), package configs (`.npmrc`, `.yarnrc`, `bunfig.toml`, `.pnpmfile.cjs`), `.pre-commit-config.yaml`/lefthook, gradle/maven wrappers, `.ripgreprc`, `pyrightconfig.json`, `.mcp.json`, and `.claude.json`. A `.claude/` write prompt offers **Yes, and allow Claude to edit its own settings for this session**.
- **`auto` mode** (research preview, requires v2.1.83+; **all plans**) routes risky actions to a separate classifier that blocks escalations: `curl | bash`, prod deploys/migrations, force-push or pushing directly to `main`, mass cloud-storage deletes, IAM grants, destructive git (`git reset --hard`, `git checkout -- .`, `git clean -fd`, etc.), and `terraform/pulumi/cdk destroy`. It allows local edits, lockfile installs, read-only HTTP, and pushing to your branch. Entering auto mode **drops blanket code-exec allow rules** (`Bash(*)`, `Bash(python*)`, `Agent` rules) and restores them on exit. Boundaries you state in chat ("don't push") are block signals re-read from the transcript each check (lost if compaction drops them — use a deny rule for a hard guarantee). After 3 consecutive or 20 total blocks, auto mode pauses and resumes prompting. Models: Opus 4.6+/Sonnet 4.6 on the Anthropic API; **only Opus 4.7/4.8** on Bedrock/Vertex/Foundry, where it also needs `CLAUDE_CODE_ENABLE_AUTO_MODE=1` (v2.1.158+). Owners enable it on Team/Enterprise; admins lock it off with `permissions.disableAutoMode: "disable"`. `defaultMode: "auto"` is ignored from project/local settings (v2.1.142+) — put it in `~/.claude/settings.json`.
- **`bypassPermissions`** refuses to start as root/sudo on Linux/macOS (skipped inside a recognized sandbox / dev container) and offers no protection against prompt injection — use it only in throwaway sandboxes. As of **v2.1.126** it also auto-approves protected-path writes (earlier versions still prompted); `rm -rf /` and `rm -rf ~` still prompt as a circuit breaker, and explicit `ask` rules still fire. You can't enter it mid-session without an enabling flag at startup. Admins block it with `permissions.disableBypassPermissionsMode: "disable"`.

In **plan mode**, when the plan is ready Claude offers to approve-and-start-in-auto, approve-and-accept-edits, approve-and-review-manually, keep planning, or refine with Ultraplan. Press `Ctrl+G` to edit the plan in your `$EDITOR` first. Deep dive: `../claude-code/cli-and-shortcuts.md` (keys) and `../claude-code/settings.md` (rule syntax).

---

## Memory: CLAUDE.md and auto memory

Two systems carry knowledge across sessions, both loaded at the start of every conversation:

**CLAUDE.md files** — instructions *you* write. They load in scope order from broadest to most specific (later = higher effective priority within a directory):

| Scope | Location |
| :--- | :--- |
| Managed policy (org-wide) | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`; Linux/WSL `/etc/claude-code/CLAUDE.md`; Windows `C:\Program Files\ClaudeCode\CLAUDE.md` |
| User (all your projects) | `~/.claude/CLAUDE.md` |
| Project (team, in version control) | `./CLAUDE.md` or `./.claude/CLAUDE.md` |
| Local (personal, gitignored) | `./CLAUDE.local.md` |

Claude walks up the directory tree concatenating every `CLAUDE.md`/`CLAUDE.local.md` it finds (root-down ordering, `CLAUDE.local.md` last within a directory); subdirectory files load on demand when Claude reads files there. Use `@path/to/file` imports (relative or absolute, max **4 hops**; wrap a path in backticks to mention it without importing), keep files under ~200 lines, and scope topic-specific guidance with `.claude/rules/*.md` (optionally `paths:`-filtered via frontmatter so they load only when matching files are touched). Block-level HTML comments (`<!-- ... -->`) are stripped before injection. By default `--add-dir` directories do **not** contribute CLAUDE.md — set `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` to load them. Org admins can ship `claudeMd` content directly inside `managed-settings.json` (loads before user/project, can't be excluded) and use `claudeMdExcludes` (glob patterns, merged across layers) to skip unrelated monorepo files. The `InstructionsLoaded` hook logs exactly which instruction files loaded — handy for debugging path-scoped rules.

For repos already using `AGENTS.md`, Claude Code reads **`CLAUDE.md`, not `AGENTS.md`** — bridge them with a top-line `@AGENTS.md` import (then add Claude-specific notes below) or a symlink (`ln -s AGENTS.md CLAUDE.md`; on Windows prefer the import since symlinks need admin/Developer Mode). `/init` also folds in `AGENTS.md`, `.cursorrules`, `.devin/rules/`, and `.windsurfrules`.

```text
# CLAUDE.md
See @README for project overview and @package.json for npm commands.

## Conventions
- Use 2-space indentation
- Run `npm test` before committing
- API handlers live in `src/api/handlers/`
```

Useful commands: `/init` generates a starter `CLAUDE.md` from your codebase (and reads existing `AGENTS.md`/`.cursorrules`); `/memory` lists loaded memory files and toggles auto memory; tell Claude "add this to CLAUDE.md" to append.

**Auto memory** — notes *Claude* writes itself (build commands, debugging insights, preferences). On by default (v2.1.59+), stored per repo at `~/.claude/projects/<project>/memory/` with a `MEMORY.md` index (first 200 lines / 25 KB loaded each session). Toggle via `/memory`, `"autoMemoryEnabled": false`, or `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`. Full reference: see `../capabilities/projects.md` for the broader projects/knowledge-base story and [memory docs](https://code.claude.com/docs/en/memory).

---

## How the deep-reference pages fit together

This overview is intentionally a map. The detailed subsystem references:

| Topic | Page |
| :--- | :--- |
| Every built-in + custom slash command (`/init`, `/memory`, `/permissions`, `/model`, …) | `../claude-code/slash-commands.md` |
| `settings.json`, permission rules, env vars (full reference) | `../claude-code/settings.md` |
| Hook events, I/O contracts, examples | `../claude-code/hooks.md` |
| CLI flags (`-p`, `--add-dir`, `--teleport`, …) + interactive keys + vim mode | `../claude-code/cli-and-shortcuts.md` |
| Subagents, agent teams, the `Agent`/`Task` tools | `../claude-code/subagents.md` |
| Dispatch, Remote Control, Routines, Code on the web, Agent View | `../claude-code/dispatch-remote-routines.md` |
| Agent Skills (`SKILL.md`, progressive disclosure) | `../capabilities/skills.md` |
| Plugins, marketplaces, `/plugin` | `../capabilities/plugins.md` |
| Model Context Protocol (transports, primitives, config) | `../capabilities/mcp.md` |
| Connectors catalog, OAuth, custom MCP connectors | `../capabilities/connectors.md` |
| Artifacts (publish/share via the `Artifact` tool) | `../capabilities/artifacts.md` |

---

## Plans, pricing, and metering

- **Subscription billing (claude.ai):** Pro, Max, Team, Enterprise. Usage draws on your plan's Claude Code allowance/limits rather than per-token API charges. Max plans have the largest allowances; Team/Enterprise add admin controls and centralized billing. (WARN: verify exact usage limits and prices per plan at [claude.com/pricing](https://claude.com/pricing) — these change.)
- **API billing (Claude Console):** metered per input/output token against the model you run; admins control access via the Claude Code vs Developer role. Good when you want hard cost control or to route through a gateway.
- **Cloud providers:** Bedrock/Vertex/Foundry bill through the cloud account, not Anthropic. Some Anthropic-hosted features (push notifications, Routines, Monitor, server-side web search on some providers) are unavailable there.

Track spend and usage in-session with `/cost` and `/status`. For model selection and exact context/pricing, see `../models/model-families.md` and `../models/capabilities-and-modes.md`.

---

## Related pages

- `./claude-ai.md` — Claude.ai web/desktop/mobile apps, settings, plans
- `./cowork.md` — Claude Cowork desktop agent for knowledge work
- `./claude-design.md` — Claude Design prompt-to-prototype workspace
- `../models/model-families.md` — model IDs, pricing, context windows
- `../models/capabilities-and-modes.md` — extended thinking, 1M context, vision, tools
- `../claude-code/slash-commands.md` — every slash command
- `../claude-code/settings.md` — settings.json, permissions, env vars
- `../claude-code/hooks.md` — hook events and examples
- `../claude-code/cli-and-shortcuts.md` — CLI flags + keyboard shortcuts + vim
- `../claude-code/subagents.md` — subagents and the Task tool
- `../claude-code/dispatch-remote-routines.md` — remote, web, scheduled surfaces
- `../capabilities/skills.md` — Agent Skills
- `../capabilities/plugins.md` — plugins and marketplaces
- `../capabilities/mcp.md` — Model Context Protocol
- `../capabilities/connectors.md` — connectors and custom MCP
- `../capabilities/artifacts.md` — artifacts
- `../glossary.md` — term definitions

## Open questions / to verify

- Exact Claude Code usage limits per plan (Pro/Max/Team/Enterprise) and current pricing — change frequently; not pinned to a single number here (verify at [claude.com/pricing](https://claude.com/pricing)).
- Current "latest"/"stable" version string at time of reading (2.1.x line as of mid-2026; confirm with `claude --version` / release notes — specific build numbers in examples float).
- The `auto`-mode model list will keep moving (Opus 4.6+/Sonnet 4.6 on the Anthropic API; only Opus 4.7/4.8 on Bedrock/Vertex/Foundry as of these docs).
- The JetBrains plugin is still **beta**-labeled in the Marketplace as of these docs; whether that and the Desktop app drop the beta/preview tag by the reader's date is unconfirmed.

## Sources

- [Claude Code — Overview](https://code.claude.com/docs/en/overview)
- [Advanced setup](https://code.claude.com/docs/en/setup)
- [Tools reference](https://code.claude.com/docs/en/tools-reference)
- [Choose a permission mode](https://code.claude.com/docs/en/permission-modes)
- [How Claude remembers your project (memory)](https://code.claude.com/docs/en/memory)
- [Authentication](https://code.claude.com/docs/en/authentication)
- [Claude Code product page](https://claude.com/product/claude-code)
