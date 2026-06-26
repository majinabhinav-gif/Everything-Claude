---
type: Configuration Reference
title: Claude Code — Settings
description: Exhaustive reference for Claude Code settings.json — file locations and precedence, every key, the permissions model, sandboxing, status line, output styles, and all environment variables.
domain: claude-code
tags: [claude-code, settings, settings.json, permissions, managed-settings, environment-variables, statusline, configuration, mcp, hooks, sandboxing]
related: [hooks, slash-commands, mcp, subagents, cli-shortcuts, plugins, connectors]
resource: https://code.claude.com/docs/en/settings
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://code.claude.com/docs/en/settings
  - https://code.claude.com/docs/en/permissions
  - https://code.claude.com/docs/en/permission-modes
  - https://code.claude.com/docs/en/statusline
  - https://code.claude.com/docs/en/sandboxing
---

# Claude Code — Settings

Claude Code reads its configuration from a layered stack of `settings.json` files (plus a handful of dedicated files like `.mcp.json`, `CLAUDE.md`, and `keybindings.json`). This page is the exhaustive reference: where the files live, the precise precedence order, every documented key with an example, the full permissions model, sandbox settings, status-line and output-style configuration, and the environment-variable catalog. The settings format is identical at every layer — what changes is *who* can set a key and *which* layer wins a conflict.

> **At a glance**
> - **What it is:** JSON configuration that controls models, permissions, hooks, MCP servers, the sandbox, the status line, telemetry, auto-updates, and dozens of feature toggles for the Claude Code agent.
> - **Where you find it:** `~/.claude/settings.json` (user), `.claude/settings.json` and `.claude/settings.local.json` (project), system **managed settings** (admin), and `--flag` CLI overrides. Edit interactively via `/config` and `/permissions`.
> - **Who can use it by plan:** Everyone with Claude Code. Some keys are *managed-only* (Team/Enterprise admin deployments). Auto mode and Remote Control may require an Owner to enable them org-wide.
> - **Status:** Stable and actively evolving. Some keys carry minimum-version notes (e.g. auto mode needs `v2.1.83+`). Quoted version floors below are accurate as of 2026-06; verify any exact version number against the live docs before relying on it.

---

## Settings files and their locations

Claude Code merges settings from several scopes. Each scope is a JSON file (or an OS policy store) with the same schema.

| Scope | File / location | Shared? | Notes |
| :--- | :--- | :--- | :--- |
| **Managed (policy)** | macOS `/Library/Application Support/ClaudeCode/managed-settings.json` (or plist domain `com.anthropic.claudecode`); Linux/WSL `/etc/claude-code/managed-settings.json`; Windows `C:\Program Files\ClaudeCode\managed-settings.json` or registry `HKLM\SOFTWARE\Policies\ClaudeCode` (a `Settings` REG_SZ/REG_EXPAND_SZ value holding JSON) | Yes (IT-deployed) | Highest priority; cannot be overridden, even by CLI flags. A per-user `HKCU\SOFTWARE\Policies\ClaudeCode` is also read at lowest policy priority. |
| **Command-line flags** | e.g. `--model`, `--permission-mode`, `--allowedTools` | N/A | Temporary, session-only overrides. |
| **Local project** | `.claude/settings.local.json` | No (gitignore it) | Your personal overrides for one repo. Auto-gitignored. |
| **Shared project** | `.claude/settings.json` | Yes (commit it) | Team-shared, version-controlled project config. |
| **User** | `~/.claude/settings.json` (`%USERPROFILE%\.claude` on Windows) | No | Your global defaults across all projects. |

**Server-managed settings** (a remotely fetched policy) are also supported on Team/Enterprise; `forceRemoteSettingsRefresh: true` (managed-only) blocks startup until that fetch succeeds and exits if it fails. On Windows, `wslInheritsWindowsSettings: true` (managed-only, set in the HKLM key or `C:\Program Files\ClaudeCode\managed-settings.json`) makes WSL also read the Windows policy chain in addition to `/etc/claude-code`. The legacy Windows path `C:\ProgramData\ClaudeCode\managed-settings.json` is **no longer supported as of v2.1.75** — administrators must migrate to `C:\Program Files\ClaudeCode\managed-settings.json`.

### Precedence (highest wins)

```
1. Managed settings          (admin policy — overrides everything, including CLI flags)
2. Command-line arguments     (--model, --permission-mode, --allowedTools, …)
3. .claude/settings.local.json   (local project)
4. .claude/settings.json         (shared project)
5. ~/.claude/settings.json       (user)
```

For most scalar keys (e.g. `model`), the highest-priority layer that defines the key wins. For **permission rules**, the layers do **not** simply override — `allow`/`ask`/`deny` arrays from *all* scopes are merged and then evaluated **deny → ask → allow**, so a deny at *any* layer blocks a tool no matter what a lower layer allows (see [Permissions](#permissions)). Array-valued sandbox keys (`allowWrite`, `allowRead`, `excludedCommands`, …) also merge across scopes. The SDK `managedSettings` option plus `parentSettingsBehavior: "merge"` lets an embedding host tighten — never loosen — policy.

---

## The `.claude/` directory

The per-project and per-user `.claude/` folder holds far more than `settings.json`:

```
.claude/
├── settings.json            # shared project settings (commit)
├── settings.local.json      # personal project settings (gitignore)
├── agents/                  # custom subagents (*.md) — see subagents.md
├── commands/                # custom slash commands (*.md) — see slash-commands.md
├── skills/                  # Agent Skills (SKILL.md) — see ../capabilities/skills.md
├── hooks/                   # hook scripts referenced from settings — see hooks.md
├── output-styles/           # custom output styles (*.md)
├── rules/                   # additional rule files merged into context
├── plans/                   # plan files (also configurable via plansDirectory)
├── statusline.sh            # common location for a status-line script
└── worktrees/               # git worktrees Claude creates (the one .claude path it may write to)
```

Sibling files at the project root (not inside `.claude/`): `.mcp.json` (project MCP servers — see [mcp.md](../capabilities/mcp.md)), `CLAUDE.md` / `CLAUDE.local.md` (memory), and `.claude.json` (legacy global state, also where per-project MCP server entries live). The user home holds `~/.claude/CLAUDE.md` (global memory), `~/.claude.json` (user/global MCP config), and `~/.claude/keybindings.json` (key remaps — see [cli-and-shortcuts.md](./cli-and-shortcuts.md)).

---

## Core settings keys

A `$schema` line gives IDEs autocomplete:

```json
{ "$schema": "https://json.schemastore.org/claude-code-settings.json" }
```

| Key | Description | Example |
| :--- | :--- | :--- |
| `model` | Default model for the main thread. Accepts an alias or full ID. | `"claude-sonnet-4-6"` |
| `outputStyle` | Adjusts the system prompt (verbosity / register). | `"Explanatory"` |
| `editorMode` | Input-box key bindings (`"normal"` default, or `"vim"`). | `"vim"` |
| `theme` | Terminal color theme. | `"dark"` |
| `language` | Preferred response language. | `"japanese"` |
| `effortLevel` | Persist the reasoning effort level across sessions. | `"medium"` (`low`/`medium`/`high`/`xhigh`) |
| `agent` | Run the main thread *as* a named subagent. | `"code-reviewer"` |
| `defaultShell` | Default shell for `!` bang commands. | `"powershell"` |
| `cleanupPeriodDays` | Delete local session transcripts older than N days. | `20` |
| `apiKeyHelper` | Script that prints an auth value (used as `Authorization`/`X-Api-Key`). | `"/bin/generate_temp_api_key.sh"` |
| `env` | Environment variables injected into every session (see [Environment variables](#environment-variables)). | `{ "BASH_DEFAULT_TIMEOUT_MS": "30000" }` |
| `verboseMode` | Enable verbose logging. | `true` |
| `tutorialsDisabled` | Hide tutorial hints. | `true` |
| `feedbackSurveyRate` | Probability (0–1) of showing a feedback survey. | `0.05` |

> The official docs do not state a default for `cleanupPeriodDays`; community references commonly cite 30 days, but treat the exact default as unverified.

### Model and inference

| Key | Description | Example |
| :--- | :--- | :--- |
| `availableModels` | Restrict which models the user can pick. | `["sonnet", "haiku"]` |
| `enforceAvailableModels` | Extend the `availableModels` allowlist to the Default model too. | `true` |
| `fallbackModel` | Fallback chain when the primary is overloaded/unavailable. | `["claude-sonnet-4-6", "claude-haiku-4-5"]` |
| `alwaysThinkingEnabled` | Turn extended thinking on by default. | `true` |
| `modelOverrides` | Map Anthropic model IDs to provider-specific IDs (Bedrock/Vertex ARNs). | `{ "claude-opus-4-6": "arn:aws:bedrock:..." }` |
| `advisorModel` | Model for the server-side advisor tool. | `"opus"` |
| `fastModePerSessionOptIn` | Require a per-session opt-in before fast mode is used. | `true` |

See [model-families.md](../models/model-families.md) for IDs/pricing and [capabilities-and-modes.md](../models/capabilities-and-modes.md) for thinking/effort.

### Memory and context

| Key | Description | Example |
| :--- | :--- | :--- |
| `autoMemoryEnabled` | Auto-write memory. | `false` |
| `autoMemoryDirectory` | Where auto memory is stored. | `"~/my-memory-dir"` |
| `autoCompactEnabled` | Auto-compact near the context limit. | `false` |
| `claudeMdExcludes` | Glob patterns of `CLAUDE.md` files to skip. | `["**/vendor/**/CLAUDE.md"]` |
| `claudeMd` | Org-injected memory (**managed-only**). | `"Always run make lint before committing."` |
| `fileCheckpointingEnabled` | Snapshot files before each edit so `/rewind` works. | `false` |
| `plansDirectory` | Where plan files are stored. | `"./plans"` |
| `showClearContextOnPlanAccept` | Offer to clear planning context when a plan is approved. | `true` |
| `respectGitignore` | Honor `.gitignore` in the `@` file picker. | `true` |
| `fileSuggestion` | Custom `@`-file autocomplete script. | `{ "type": "command", "command": "~/.claude/file-suggestion.sh" }` |

### Skills and plugins

| Key | Description | Example |
| :--- | :--- | :--- |
| `maxSkillDescriptionChars` | Per-skill cap on description characters. | `2048` |
| `skillListingBudgetFraction` | Fraction of the context budget reserved for skill listings. | `0.15` |
| `disableBundledSkills` | Disable skills/workflows shipped with Claude Code. | `true` |
| `disableSkillShellExecution` | Disable inline shell execution inside skills. | `true` |
| `strictPluginOnlyCustomization` | Block user/project skills/agents/hooks/MCP — `true` or an array (**managed-only**). | `["skills", "hooks"]` |
| `pluginSuggestionMarketplaces` | Allowlist for plugin suggestions (**managed-only**). | `["acme-corp-plugins"]` |
| `pluginTrustMessage` | Custom plugin trust warning (**managed-only**). | `"All plugins are approved by IT"` |
| `blockedMarketplaces` | Blocklist of marketplace sources (**managed-only**). | `[{ "source": "github", "repo": "untrusted/plugins" }]` |
| `strictKnownMarketplaces` | Enforce a known-marketplace list (**managed-only**). | `[{ "name": "acme-plugins", "source": "github", "repo": "acme-corp/plugins" }]` |

See [plugins.md](../capabilities/plugins.md) and [skills.md](../capabilities/skills.md).

### Attribution and version control

| Key | Description | Example |
| :--- | :--- | :--- |
| `attribution` | Customize git commit / PR attribution strings. | `{ "commit": "🤖 Generated with Claude Code", "pr": "" }` |
| `includeCoAuthoredBy` | **Deprecated** — use `attribution`. Controls the `Co-Authored-By: Claude` trailer. (Still listed as a deprecated key in the live docs.) | `false` |
| `includeGitInstructions` | Inject git-workflow guidance into the system prompt. | `false` |
| `prUrlTemplate` | URL template for the PR footer badge. | `"https://reviews.example.com/{owner}/{repo}/pull/{number}"` |

### UI, display, and notifications

| Key | Description | Example |
| :--- | :--- | :--- |
| `spinnerTipsEnabled` | Show rotating tips next to the spinner. | `false` |
| `awaySummaryEnabled` | Recap what happened while you were away. | `true` |
| `autoScrollEnabled` | Follow new output to the bottom in fullscreen. | `false` |
| `prefersReducedMotion` | Reduce/disable animations. | `true` |
| `axScreenReader` | Screen-reader-friendly rendering. | `true` |
| `preferredNotifChannel` | Notification channel. | `"terminal_bell"` (also `auto`, `iterm2`, `iterm2_with_bell`, `kitty`, `ghostty`, `notifications_disabled`) |
| `agentPushNotifEnabled` | Allow proactive push notifications. | `true` |
| `inputNeededNotifEnabled` | Push when Claude needs input. | `true` |
| `companyAnnouncements` | Startup banners (managed deployments). | `["Welcome to Acme Corp! See docs.acme.com"]` |
| `footerLinksRegexes` | Turn IDs in the transcript into clickable footer badges. | see [Status line / footer](#status-line) |

### Auto-updates

| Key | Description | Example |
| :--- | :--- | :--- |
| `autoUpdatesChannel` | Release channel to follow. | `"stable"` (or `"latest"`) |
| `minimumVersion` | Floor for auto-updates. | `"2.1.100"` |
| `requiredMinimumVersion` / `requiredMaximumVersion` | Hard startup floor/ceiling (**managed-only**). | `"2.1.150"` |

To disable updates entirely, set the env var `DISABLE_AUTOUPDATER=1` (see below). Detailed update channels and CLI flags live in [cli-and-shortcuts.md](./cli-and-shortcuts.md).

### Feature toggles

Many features can be turned off via settings (each also has an env-var equivalent, listed in [Environment variables](#environment-variables)):

| Key | Disables | Example |
| :--- | :--- | :--- |
| `disableAgentView` | Background agents / Agent View. | `true` |
| `disableArtifact` | The Artifact (publish-web-page) tool. | `true` |
| `disableWorkflows` | Dynamic workflows. | `true` |
| `disableRemoteControl` | Remote Control (managed device key — see note below). | `true` |
| `disableDeepLinkRegistration` | Protocol-handler (deep-link) registration. | `"disable"` |
| `disableAllHooks` | All hooks **and** the status line. | `true` |
| `remoteControlAtStartup` | Auto-connect Remote Control at startup (`false` to opt out). | `false` |

> On Team/Enterprise, an Owner enables/disables [Remote Control](./dispatch-remote-routines.md) and web sessions org-wide in Claude Code admin settings. `disableRemoteControl` additionally disables Remote Control **per device** as a managed setting; web sessions have no per-device key.

See [dispatch-remote-routines.md](./dispatch-remote-routines.md) for Agent View / Remote Control and [plugins.md](../capabilities/plugins.md) / [skills.md](../capabilities/skills.md) for the things these toggles gate.

### Channels (managed)

| Key | Description |
| :--- | :--- |
| `channelsEnabled` | Allow [channels](./dispatch-remote-routines.md) for the org (**managed-only**; default varies by plan). |
| `allowedChannelPlugins` | Allowlist of channel plugins that may push messages (**managed-only**; requires `channelsEnabled`). |

---

## Permissions

The `permissions` object is the heart of Claude Code's safety model. It holds rule arrays and the default permission mode.

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)",
      "Edit(/src/**/*.ts)"
    ],
    "ask": [
      "Bash(npm run deploy *)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "WebFetch(domain:internal.example.com)"
    ],
    "defaultMode": "acceptEdits",
    "additionalDirectories": ["../shared-lib", "/path/to/allowed/dir"],
    "disableBypassPermissionsMode": "disable"
  }
}
```

### Evaluation order

Rules are evaluated **deny → ask → allow**; the first match wins, and specificity does **not** change the order. A broad `deny` like `Bash(aws *)` blocks even a call that also matches a narrow `allow` like `Bash(aws s3 ls)` — deny rules carry no allowlist exceptions. The same holds for `ask` vs `allow`: a matching `ask` prompts even if a more specific `allow` exists. Because arrays merge across all scopes, **a deny at any layer is final** (a managed deny cannot be overridden by `--allowedTools`; `--disallowedTools` can only add restrictions).

A bare tool name as a deny rule (`"Bash"`) removes the tool from Claude's context entirely; a scoped rule (`"Bash(rm *)"`) leaves the tool available but blocks matching calls.

Permission rules are enforced by Claude Code, **not by the model** — instructions in your prompt or `CLAUDE.md` shape what Claude *tries*, not what is allowed.

### The `Tool(specifier)` syntax

| Form | Meaning | Example |
| :--- | :--- | :--- |
| `Tool` | Matches **every** use of the tool. | `Bash`, `WebFetch`, `Read` |
| `Tool(*)` | Same as bare tool name. | `Bash(*)` |
| `Tool(specifier)` | Fine-grained match. | `Bash(npm run build)` |
| `Tool(param:value)` | Match a scalar top-level input parameter (**deny/ask only**). | `Agent(model:opus)`, `Agent(isolation:worktree)`, `Bash(run_in_background:true)` |

Per-tool specifier rules:

- **`Bash(...)`** — glob with `*` at any position. A space before `*` enforces a word boundary: `Bash(ls *)` matches `ls -la` but not `lsof`; `Bash(ls*)` matches both. `Bash(ls:*)` is an equivalent trailing-wildcard form (the `:*` suffix only works at the **end** of a pattern). Claude Code is shell-aware: a rule must match **each** subcommand of a compound command (separators `&& || ; | |& &` and newlines). Approving a compound command with "Yes, don't ask again" saves a separate rule per subcommand (up to 5). Process wrappers `timeout`, `time`, `nice`, `nohup`, `stdbuf`, and bare `xargs` (no flags) are stripped before matching; environment-runner wrappers (`npx`, `docker exec`, `devbox run`, `direnv exec`, `mise exec`) are **not**, so write `Bash(devbox run npm test)` for those. Exec wrappers (`watch`, `setsid`, `ionice`, `flock`) and `find -exec`/`-delete` always prompt and cannot be prefix-approved. A built-in read-only set (`ls cat echo pwd head tail grep find wc which diff stat du cd`, read-only `git`) never prompts.
- **`PowerShell(...)`** — same shape as Bash rules; `:*` ≡ trailing ` *`. Cmdlet aliases are canonicalized (`PowerShell(Get-ChildItem *)` also matches `gci`/`ls`/`dir`); matching is case-insensitive. The AST is parsed so each subcommand of a pipeline/compound command must match.
- **`Read(...)` / `Edit(...)` / `Write(...)`** — gitignore-style path patterns with four anchors: `//abs` (filesystem root), `~/home`, `/proj` (project root — **not** absolute!), and `path` or `./path` (cwd). `*` matches within a segment, `**` across segments. Bare filenames match at any depth, so `Read(.env)` ≡ `Read(**/.env)`. On Windows, paths are normalized to POSIX (`C:\Users\alice` → `/c/Users/alice`; use `//c/**/.env` or `//**/.env`). Symlinks check both the link and its target: allow rules require **both** to match (else prompt); deny rules block if **either** matches. `Edit` rules apply to all built-in file-edit tools; `Read` rules also best-effort cover Grep/Glob, `@file` mentions, and IDE selection/open-file context, plus recognized file commands in Bash (`cat`, `head`, `tail`, `sed`) — but not arbitrary scripts that open files (use the sandbox for that).
- **`WebFetch(domain:host)`** — matches the URL hostname (case-insensitive; trailing `.` stripped). `WebFetch(domain:*.example.com)` matches any subdomain at any depth but **not** the apex; `WebFetch(domain:*)` ≡ bare `WebFetch`. A wildcard in any non-leading position matches only between two dots (`example.*` matches `example.org` but not `example.evil.com`).
- **`mcp__server` / `mcp__server__tool`** — `mcp__github` matches all tools from the `github` server; `mcp__github__get_*` matches its `get_` tools. Deny/ask also accept tool-name globs (`mcp__*` = every MCP tool; `*` = every tool); allow globs must be anchored after a literal `mcp__<server>__` (the server segment must be glob-free). Unanchored allow globs are skipped with a warning.
- **`Agent(Name)`** — gate subagents: `Agent(Explore)`, `Agent(Plan)`, `Agent(my-custom-agent)`. Add to `deny` (or use `--disallowedTools`) to disable. See [subagents.md](./subagents.md).
- **`Cd(path)`** — controls where `/cd` may move the session (not model-invocable). Any `Cd` allow rule switches `/cd` to allowlist mode. Patterns use the `//`/`~/`/`/` anchors but match the whole directory path (`*` = one segment, `**` = across; a trailing `/**` also matches its root).

> **`Tool(param:value)` caveat.** This matches a *scalar, direct* input field, before normalization (`Agent(model:opus)` matches the alias `opus`, not a full ID). It is **deny/ask only**. Fields a tool already canonicalizes are **not** matchable this way — `command` (Bash/PowerShell), `file_path` (Read/Edit/Write), `path` (Grep/Glob), `notebook_path` (NotebookEdit), `url` (WebFetch). A rule like `Bash(command:rm *)` is ignored with a startup warning; use `Bash(rm *)` instead.

> **Argument-constraining Bash rules are fragile.** `Bash(curl http://github.com/ *)` is trivially bypassed (`-X` before URL, https, redirects, `$VAR`, extra spaces). Prefer a `deny` on `curl`/`wget` plus `WebFetch(domain:...)` allows, or a `PreToolUse` hook. Note WebFetch allows do not block network access — if Bash is allowed, Claude can still `curl` any URL.

### `defaultMode` and permission modes

`defaultMode` sets the starting mode; cycle live with **Shift+Tab** (`default → acceptEdits → plan`, with optional modes slotted after `plan`: `bypassPermissions` first, `auto` last).

| Mode | Runs without asking | Best for |
| :--- | :--- | :--- |
| `default` | Reads only | Sensitive work, getting started |
| `acceptEdits` | Reads + file edits + common fs commands (`mkdir touch rm rmdir mv cp sed`) in-scope | Iterating on reviewed code |
| `plan` | Reads only; proposes a plan, makes no edits | Exploring before changing |
| `auto` | Everything, with a background safety **classifier** | Long tasks, fewer prompts (research preview) |
| `dontAsk` | Only pre-approved (`allow`) tools + read-only Bash; everything else auto-**denied** | Locked-down CI |
| `bypassPermissions` | Everything (skips checks) | Isolated containers/VMs only |

Notes:
- Explicit `ask` and `deny` rules apply in **every** mode, including `bypassPermissions`. `rm -rf /` and `rm -rf ~` still prompt as a circuit breaker.
- **`acceptEdits`** auto-approves the filesystem commands above (and, with the PowerShell tool enabled, `Set-Content`/`Add-Content`/`Clear-Content`/`Remove-Item`) only inside the working dir or `additionalDirectories`; safe env-var prefixes (`LANG=C`, `NO_COLOR=1`) and process wrappers are tolerated.
- **`auto`** requires `v2.1.83+`; on the Anthropic API it needs Opus 4.6+/Sonnet 4.6, on Bedrock/Vertex/Foundry only Opus 4.7/4.8 with `CLAUDE_CODE_ENABLE_AUTO_MODE=1` (`v2.1.158+`). A separate classifier model (independent of `/model`) reviews each escalating action; broad allow rules (`Bash(*)`, wildcarded interpreters, `Agent` allows) are temporarily dropped on entry. After 3 consecutive or 20 total blocks, auto mode falls back to prompting. `defaultMode: "auto"` is **ignored** in project/local files (set it in `~/.claude/settings.json` or managed) — enforced since `v2.1.142`.
- **Protected paths** (see below) are never auto-approved in any mode except `bypassPermissions` — even an `allow` rule for `.claude/**` won't pre-approve them (the safety check runs before allow rules).
- Admins lock modes off with `permissions.disableAutoMode: "disable"` and `permissions.disableBypassPermissionsMode: "disable"` (both work from any scope but are most useful in managed settings; a user can also lock *themselves* out of bypass mode). `bypassPermissions` is reachable only after starting with `--permission-mode bypassPermissions` / `--dangerously-skip-permissions` (and refuses to start as root/sudo on Linux/macOS unless in a recognized sandbox).

Full mode mechanics: [permission-modes docs](https://code.claude.com/docs/en/permission-modes).

### Protected paths

Writes to a small fixed set of paths are never auto-approved in any mode except `bypassPermissions` (in `auto` they route to the classifier; in `dontAsk` they are denied). `permissions.allow` rules do **not** pre-approve them. The exact list:

- **Directories:** `.git`, `.config/git`, `.vscode`, `.idea`, `.husky`, `.cargo`, `.devcontainer`, `.yarn`, `.mvn`, and `.claude` (except `.claude/worktrees`, where Claude stores its own git worktrees).
- **Files:** `.gitconfig`, `.gitmodules`; shell dotfiles `.bashrc`, `.bash_profile`, `.bash_login`, `.bash_aliases`, `.bash_logout`, `.zshrc`, `.zprofile`, `.zshenv`, `.zlogin`, `.zlogout`, `.profile`, `.envrc`; package configs `.npmrc`, `.yarnrc`, `.yarnrc.yml`, `.pnp.cjs`, `.pnp.loader.mjs`, `.pnpmfile.cjs`, `bunfig.toml`, `.bunfig.toml`; `.bazelrc`, `.bazelversion`, `.bazeliskrc`; `.pre-commit-config.yaml`, `lefthook.{yml,yaml}`, `.lefthook.{yml,yaml}`; `gradle-wrapper.properties`, `maven-wrapper.properties`; `.devcontainer.json`; `.ripgreprc`, `pyrightconfig.json`; and `.mcp.json`, `.claude.json`.

In a prompting mode, a `.claude/` write offers **"Yes, and allow Claude to edit its own settings for this session,"** which approves later `.claude/` writes for that session.

### `additionalDirectories`

Grants read/edit access to directories beyond the launch dir (same as `--add-dir` / `/add-dir` at runtime). Note: directories listed in `permissions.additionalDirectories` grant **file access only** — they do **not** load `.claude/` config from those dirs. The `--add-dir` flag (or `/add-dir`) additionally loads skills (with live reload) and subagents from those dirs, plus `enabledPlugins`/`extraKnownMarketplaces` keys, and `CLAUDE.md` files only when `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`. To relocate the session's primary working directory instead, use `/cd` (`v2.1.169+`), which loads the new dir's `CLAUDE.md`.

### Managed permission keys

| Key | Effect |
| :--- | :--- |
| `allowManagedPermissionRulesOnly` | Only managed `allow`/`ask`/`deny` rules apply; user/project rules ignored (does not affect the MCP allowlist). |
| `permissions.disableBypassPermissionsMode` | `"disable"` blocks bypass mode (works from any scope; usually managed). |
| `permissions.disableAutoMode` | `"disable"` blocks auto mode (overrides the enable env var). |
| `forceLoginMethod` | Restrict login to `"claudeai"` or `"console"`. |
| `forceLoginOrgUUID` | Require login to a specific org UUID (a UUID string). |
| `strictPluginOnlyCustomization` | Block user/project skills/agents/hooks/MCP — `true` or `["skills","hooks"]`. |
| `allowManagedHooksOnly` | Only managed/SDK/force-enabled-plugin hooks load. |
| `allowManagedMcpServersOnly` | Only managed `allowedMcpServers` apply (`deniedMcpServers` still merges). |
| `allowAllClaudeAiMcps` | Load claude.ai connectors alongside a deployed `managed-mcp.json` (managed-only). |

Edit permissions interactively with **`/permissions`** (lists every rule + its source file, and a "Recently denied" tab where you press `r` to retry a denied action). See [Permissions docs](https://code.claude.com/docs/en/permissions). A `PreToolUse` hook (or a hook exiting code 2) can also block a call before rules are evaluated — but hook *approvals* never override a matching `deny`/`ask`.

---

## Hooks

`hooks` registers shell commands (or SDK/HTTP handlers) that fire at lifecycle events such as `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, and `SessionStart`. This is how "whenever X happens, run Y" automation is implemented — the harness runs hooks, not the model.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/validate-bash.sh" }
        ]
      }
    ],
    "Stop": [
      { "hooks": [ { "type": "command", "command": "say 'done'" } ] }
    ]
  }
}
```

Related hook-gating keys: `allowedHttpHookUrls`, `httpHookAllowedEnvVars`, `allowManagedHooksOnly`, `disableAllHooks`. Hook matchers and permission rules match a tool's **canonical** name (e.g. the row labeled `Stop Task` is canonically `TaskStop`). The complete event list, JSON I/O contract, exit-code semantics, and more examples are in **[hooks.md](./hooks.md)**.

---

## Sandbox settings

The `sandbox` object configures OS-level filesystem/network isolation for the Bash tool (macOS Seatbelt, Linux/WSL2 bubblewrap; native Windows unsupported — run inside WSL2). It complements permissions: permissions decide *whether* a tool runs; the sandbox enforces *what a running Bash command can touch*.

```json
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": false,
    "allowUnsandboxedCommands": true,
    "excludedCommands": ["docker *", "gh"],
    "filesystem": {
      "allowWrite": ["~/.kube", "/tmp/build"],
      "denyRead": ["~/"],
      "allowRead": ["."],
      "denyWrite": ["/etc"]
    },
    "network": {
      "allowedDomains": ["registry.npmjs.org", "github.com"],
      "deniedDomains": ["telemetry.example.com"],
      "httpProxyPort": 8080,
      "socksProxyPort": 8081
    },
    "credentials": {
      "files": [
        { "path": "~/.aws/credentials", "mode": "deny" },
        { "path": "~/.ssh", "mode": "deny" }
      ],
      "envVars": [ { "name": "GITHUB_TOKEN", "mode": "deny" } ]
    }
  }
}
```

| Key | Description |
| :--- | :--- |
| `sandbox.enabled` | Turn the sandbox on for the scope. |
| `sandbox.failIfUnavailable` | Refuse to start (instead of warning + running unsandboxed) if the sandbox can't init. |
| `sandbox.allowUnsandboxedCommands` | `false` disables the `dangerouslyDisableSandbox` escape hatch ("Strict sandbox mode"). |
| `sandbox.excludedCommands` | Commands that always run outside the sandbox (e.g. `docker *`, Go CLIs on macOS). No managed-only lockdown — keep narrow. |
| `sandbox.filesystem.allowWrite` / `denyWrite` / `denyRead` / `allowRead` | Path allow/deny. Paths use standard prefixes (`/`=absolute, `~/`=home, `./` or bare = project root for project settings / `~/.claude` for user settings) — **different** from Read/Edit rule anchors. Arrays merge across scopes. |
| `sandbox.filesystem.allowManagedReadPathsOnly` | (**managed-only**) Only managed `allowRead` paths honored. |
| `sandbox.network.allowedDomains` / `deniedDomains` | Domain allow/deny (no domains pre-allowed; first new domain prompts, then is allowed for the session as of `v2.1.191`). |
| `sandbox.network.allowManagedDomainsOnly` | (**managed-only**) Block non-allowed domains silently; only managed `allowedDomains` honored. |
| `sandbox.network.httpProxyPort` / `socksProxyPort` | Point at a custom (e.g. TLS-inspecting) proxy. |
| `sandbox.credentials` | (**managed-relevant**, `v2.1.187+`) Deny reads of credential files and unset secret env vars for sandboxed commands. `mode` is always `"deny"`; merges across scopes. |
| `autoAllowBashIfSandboxed` | Default `true`: sandboxed Bash runs without prompting even with a bare `Bash` ask rule (content-scoped asks like `Bash(git push *)`, explicit denies, and dangerous `rm` to `/`/home still prompt). |
| `allowAppleEvents` (macOS) | Allow Apple Events (`open`/`osascript`); user/managed/CLI only, **not** project. Weakens isolation. |
| `enableWeakerNetworkIsolation` / `enableWeakerNestedSandbox` / `enableWeakerNestedSandbox` | Compatibility escape hatches that weaken isolation; use only when an outer boundary already exists. |

Full reference and Linux/WSL2 setup (`bubblewrap`, `socat`, seccomp): [sandboxing docs](https://code.claude.com/docs/en/sandboxing).

---

## Status line

`statusLine` runs a shell command that receives session JSON on **stdin** and prints the bottom status bar (rendered above the built-in footer badges, not replacing them). Generate one conversationally with `/statusline show model and context %`.

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh",
    "padding": 2,
    "refreshInterval": 5,
    "hideVimModeIndicator": false
  }
}
```

An inline command works too:

```json
{
  "statusLine": {
    "type": "command",
    "command": "jq -r '\"[\\(.model.display_name)] \\(.context_window.used_percentage // 0)% context\"'"
  }
}
```

Fields: `type` (`"command"`), `command` (script path or inline shell), `padding` (extra horizontal chars, default `0`), `refreshInterval` (re-run every N seconds in addition to event-driven updates; min `1`), `hideVimModeIndicator` (suppress the built-in `-- INSERT --` when your script renders `vim.mode` itself). Updates fire after each assistant message, after `/compact`, on permission-mode change, and on vim-mode toggle, debounced at 300 ms. `disableAllHooks: true` also disables the status line. As of `v2.1.153+`, read `COLUMNS`/`LINES` env vars for terminal width (`tput cols` cannot see it). The command requires accepting the workspace-trust dialog.

The JSON delivered on stdin (abridged):

```json
{
  "cwd": "/current/working/directory",
  "session_id": "abc123...",
  "session_name": "my-session",
  "transcript_path": "/path/to/transcript.jsonl",
  "model": { "id": "claude-opus-4-8", "display_name": "Opus" },
  "workspace": {
    "current_dir": "/current/working/directory",
    "project_dir": "/original/project/directory",
    "added_dirs": [],
    "git_worktree": "feature-xyz",
    "repo": { "host": "github.com", "owner": "anthropics", "name": "claude-code" }
  },
  "version": "2.1.90",
  "output_style": { "name": "default" },
  "cost": { "total_cost_usd": 0.01234, "total_duration_ms": 45000, "total_api_duration_ms": 2300, "total_lines_added": 156, "total_lines_removed": 23 },
  "context_window": {
    "total_input_tokens": 15500,
    "total_output_tokens": 1200,
    "context_window_size": 200000,
    "used_percentage": 8,
    "remaining_percentage": 92,
    "current_usage": { "input_tokens": 8500, "output_tokens": 1200, "cache_creation_input_tokens": 5000, "cache_read_input_tokens": 2000 }
  },
  "exceeds_200k_tokens": false,
  "effort": { "level": "high" },
  "thinking": { "enabled": true },
  "rate_limits": {
    "five_hour": { "used_percentage": 23.5, "resets_at": 1738425600 },
    "seven_day": { "used_percentage": 41.2, "resets_at": 1738857600 }
  },
  "vim": { "mode": "NORMAL" },
  "agent": { "name": "security-reviewer" },
  "pr": { "number": 1234, "url": "https://github.com/anthropics/claude-code/pull/1234", "review_state": "pending" },
  "worktree": { "name": "my-feature", "path": "/path/to/.claude/worktrees/my-feature", "branch": "worktree-my-feature", "original_cwd": "/path/to/project", "original_branch": "main" }
}
```

Notes on key fields:
- `cwd` ≡ `workspace.current_dir` (prefer the latter); `workspace.project_dir` is the launch dir; `workspace.added_dirs` lists `--add-dir`/`/add-dir` paths.
- `context_window.context_window_size` is `200000` by default or `1000000` for extended-context models. `used_percentage`/`remaining_percentage` are **input tokens only** (`input + cache_creation + cache_read`, excluding output) and may be `null` early or after `/compact`. As of `v2.1.132`, `total_input_tokens`/`total_output_tokens` reflect *current* context (not cumulative session totals).
- `effort.level` may be `low`/`medium`/`high`/`xhigh`/`max` (Ultracode reports as `xhigh`); absent when the model lacks the effort param.
- `rate_limits` (both `five_hour` and `seven_day`) appears only for Claude.ai Pro/Max subscribers after the first API response; each window may be independently absent — use `// empty` in jq.
- Often-absent fields: `session_name`, `workspace.git_worktree`, `workspace.repo`, `effort`, `vim`, `agent`, `pr` (`pr.review_state` is `approved`/`pending`/`changes_requested`/`draft`), and `worktree.*` (present only in `--worktree` sessions).

A separate, script-free way to add clickable footer badges is `footerLinksRegexes`:

```json
{
  "footerLinksRegexes": [
    { "type": "regex", "pattern": "\\b(?<key>PROJ-\\d+)\\b",
      "url": "https://issues.example.com/browse/{key}", "label": "{key}" }
  ]
}
```

### Subagent status line

`subagentStatusLine` renders a custom row body for each subagent in the agent panel, replacing the default `name · description · token count`:

```json
{ "subagentStatusLine": { "type": "command", "command": "~/.claude/subagent-statusline.sh" } }
```

It runs once per refresh tick with all visible rows as one JSON object on stdin (base hook fields plus `columns` and a `tasks` array of `{id, name, type, status, description, label, startTime, tokenCount, tokenSamples, cwd}`). Write one `{"id": "...", "content": "..."}` line per row to override; omit a row's `id` to keep the default; emit empty `content` to hide it. The same trust and `disableAllHooks` gates apply; plugins can ship a default.

---

## Output styles

`outputStyle` selects how Claude formats responses (verbosity, comment density, register). It is part of the system prompt, so a change takes effect on the next turn and **requires a restart** (along with `model`). Built-ins include `"default"`, `"Explanatory"`, and `"Proactive"` (the last nudges autonomous behavior while keeping permission prompts). Set it via `/config` or directly:

```json
{ "outputStyle": "Explanatory" }
```

Custom styles live in `.claude/output-styles/*.md` (project) or `~/.claude/output-styles/*.md` (user).

---

## MCP server settings

These keys govern which Model Context Protocol servers from `.mcp.json` are trusted. Full transport/primitive details are in **[mcp.md](../capabilities/mcp.md)** and connector OAuth/admin in **[connectors.md](../capabilities/connectors.md)**.

| Key | Description | Example |
| :--- | :--- | :--- |
| `enableAllProjectMcpServers` | Auto-approve every server in project `.mcp.json` (default `false`). | `true` |
| `enabledMcpjsonServers` | Approve specific `.mcp.json` servers by name. | `["memory", "github"]` |
| `disabledMcpjsonServers` | Reject specific `.mcp.json` servers. | `["filesystem"]` |
| `allowedMcpServers` / `deniedMcpServers` | Allow/deny lists (`allowed` is **managed-only**; `denied` merges from all sources). | `[{ "serverName": "github" }]` |
| `allowManagedMcpServersOnly` | Only the managed allowlist applies (deny still merges). | `true` |
| `disableClaudeAiConnectors` | Disable claude.ai MCP connectors. | `true` |
| `allowAllClaudeAiMcps` | (**managed-only**) Load claude.ai connectors alongside `managed-mcp.json`. | `true` |
| `MCP_TIMEOUT` (env) | MCP request timeout in ms. | `"30000"` |

---

## Credential helpers (Bedrock / Vertex / custom auth)

| Key | Description | Example |
| :--- | :--- | :--- |
| `apiKeyHelper` | Script printing the auth value; refresh cadence via `CLAUDE_CODE_API_KEY_HELPER_TTL_MS`. | `"/bin/generate_temp_api_key.sh"` |
| `awsCredentialExport` | Script outputting JSON AWS credentials. | `"/bin/generate_aws_grant.sh"` |
| `awsAuthRefresh` | Script to refresh AWS creds (e.g. SSO). | `"aws sso login --profile myprofile"` |
| `gcpAuthRefresh` | Refresh GCP Application Default Credentials. | `"gcloud auth application-default login"` |
| `otelHeadersHelper` | Generate dynamic OpenTelemetry headers. | `"/bin/generate_otel_headers.sh"` |

---

## Environment variables

Set these in your shell or in the `env` block of any `settings.json`. (Values are strings in JSON.) Defaults are taken from the live settings reference; where a default is shown as "(default)" the docs do not pin an exact number.

### Authentication and provider routing

| Variable | Purpose | Example |
| :--- | :--- | :--- |
| `ANTHROPIC_API_KEY` | Anthropic API key. | `sk-ant-...` |
| `ANTHROPIC_AUTH_TOKEN` | OAuth bearer token (sent as `Authorization`). | (token) |
| `ANTHROPIC_BASE_URL` | Custom API endpoint / LLM gateway. | `https://gw.example.com` |
| `ANTHROPIC_MODEL` | Override default model. | `claude-opus-4-6` |
| `ANTHROPIC_SMALL_FAST_MODEL` | Small/fast model for background pilot actions. | `claude-haiku-4-5` |
| `CLAUDE_CODE_USE_BEDROCK` | Route through AWS Bedrock. | `1` |
| `CLAUDE_CODE_USE_VERTEX` | Route through Google Vertex AI. | `1` |
| `CLAUDE_CODE_USE_FOUNDRY` | Route through Microsoft Foundry. | `1` |
| `AWS_PROFILE` / `AWS_REGION` | Bedrock profile/region. | `myprofile` / `us-west-2` |
| `GOOGLE_APPLICATION_CREDENTIALS` / `GOOGLE_CLOUD_PROJECT` | Vertex creds/project. | `/path/creds.json` |

### Tool behavior and limits

| Variable | Purpose | Default |
| :--- | :--- | :--- |
| `BASH_DEFAULT_TIMEOUT_MS` | Default Bash timeout. | `30000` (30 s) |
| `BASH_MAX_TIMEOUT_MS` | Max allowed Bash timeout. | `600000` (10 min) |
| `MAX_THINKING_TOKENS` | Cap on extended-thinking tokens. | model default |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS` | Max output tokens per turn. | model max |
| `MCP_TIMEOUT` | MCP request timeout (ms). | (default) |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | Route `!` commands through the PowerShell tool. | `0` |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS` | `apiKeyHelper` refresh interval. | (default) |
| `CLAUDE_CODE_EFFORT_LEVEL` | Effort level (`low`/`medium`/`high`/`xhigh`). | (none) |
| `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB` | Strip Anthropic + cloud-provider creds from all subprocess envs. | (unset) |

### Telemetry, privacy, updates

| Variable | Purpose |
| :--- | :--- |
| `DISABLE_TELEMETRY` | `1` disables telemetry reporting. |
| `DISABLE_COST_WARNINGS` | `1` suppresses cost warnings. |
| `DISABLE_AUTOUPDATER` | `1` disables auto-updates entirely. |
| `CLAUDE_CODE_ENABLE_TELEMETRY` | `1` enables OpenTelemetry export (pair with `OTEL_METRICS_EXPORTER`). |
| `OTEL_METRICS_EXPORTER` | e.g. `otlp`. |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS` | Refresh interval for `otelHeadersHelper`. |
| `CLAUDE_CODE_SKIP_PROMPT_HISTORY` | `1` skips transcript writes. |

### Display, accessibility, feature toggles

| Variable | Purpose |
| :--- | :--- |
| `NO_COLOR` / `FORCE_COLOR` | Disable / force colored output. |
| `FORCE_HYPERLINK` | `1` forces OSC 8 hyperlink support detection. |
| `CLAUDE_AX_SCREEN_READER` | `1` enables screen-reader mode (default `0`). |
| `DISABLE_AUTO_COMPACT` | `1` disables auto-compaction (default `0`). |
| `CLAUDE_CODE_ENABLE_AUTO_MODE` | `1` enables auto mode on Bedrock/Vertex/Foundry (`v2.1.158+`). |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | `1` loads `CLAUDE.md` from `--add-dir` directories. |
| `CLAUDE_CODE_DISABLE_*` | Env equivalents for many `disable*` settings: `_AUTO_MEMORY`, `_ARTIFACT`, `_AGENT_VIEW`, `_BUNDLED_SKILLS`, `_FILE_CHECKPOINTING`, `_FEEDBACK_SURVEY`, `_WORKFLOWS` (each default `0`). |
| `CLAUDE_CODE_ENABLE_AWAY_SUMMARY` | `1` shows the away summary (default `1`). |

---

## Complete annotated `settings.json` example

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",

  "model": "claude-sonnet-4-6",
  "fallbackModel": ["claude-sonnet-4-6", "claude-haiku-4-5"],
  "outputStyle": "Explanatory",
  "editorMode": "vim",
  "effortLevel": "medium",
  "alwaysThinkingEnabled": false,

  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Bash(git commit *)",
      "Read(~/.zshrc)",
      "Edit(/src/**)",
      "WebFetch(domain:docs.claude.com)",
      "mcp__github__get_*"
    ],
    "ask": [
      "Bash(npm run deploy *)"
    ],
    "deny": [
      "Bash(curl *)",
      "Bash(git push *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ],
    "additionalDirectories": ["../shared-lib"]
  },

  "sandbox": {
    "enabled": true,
    "filesystem": { "allowWrite": ["/tmp/build"] },
    "credentials": { "files": [ { "path": "~/.ssh", "mode": "deny" } ] }
  },

  "env": {
    "BASH_DEFAULT_TIMEOUT_MS": "60000",
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp"
  },

  "attribution": { "commit": "🤖 Generated with Claude Code", "pr": "" },
  "includeCoAuthoredBy": false,

  "cleanupPeriodDays": 20,
  "autoUpdatesChannel": "stable",
  "spinnerTipsEnabled": false,
  "preferredNotifChannel": "terminal_bell",

  "enableAllProjectMcpServers": false,
  "enabledMcpjsonServers": ["memory", "github"],

  "statusLine": { "type": "command", "command": "~/.claude/statusline.sh", "padding": 2 },

  "apiKeyHelper": "/bin/generate_temp_api_key.sh",

  "hooks": {
    "PreToolUse": [
      { "matcher": "Bash",
        "hooks": [ { "type": "command", "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/guard.sh" } ] }
    ]
  }
}
```

---

## The `/config` and `/permissions` UIs

- **`/config`** opens an interactive settings panel: switch model, output style, theme, editor mode, and toggle common features without hand-editing JSON. Changes write to the appropriate `settings.json`.
- **`/permissions`** lists every active rule, the file it came from, and offers tabs to add `allow`/`ask`/`deny` rules. Its "Recently denied" tab lets you press `r` to retry a blocked action with a manual approval.
- **`/sandbox`** opens the sandbox panel (Mode / Overrides / Config tabs; selecting a mode writes to `.claude/settings.local.json`).
- Run **`claude doctor`** to validate settings; it reports invalid managed entries with their source. Managed settings parse tolerantly — invalid entries are stripped with a warning while valid policy stays enforced. `claude --debug` logs the status-line exit code and stderr.

See [slash-commands.md](./slash-commands.md) for `/config`, `/permissions`, `/statusline`, `/sandbox`, `/add-dir`, `/cd`, and the rest of the command catalog, and [cli-and-shortcuts.md](./cli-and-shortcuts.md) for the CLI flags (`--model`, `--permission-mode`, `--allowedTools`, `--add-dir`, `--dangerously-skip-permissions`) that override these settings per session.

---

## Related pages

- [hooks.md](./hooks.md) — every hook event, the JSON I/O contract, and examples
- [slash-commands.md](./slash-commands.md) — `/config`, `/permissions`, `/statusline`, `/sandbox`, custom commands
- [cli-and-shortcuts.md](./cli-and-shortcuts.md) — CLI flags, keyboard shortcuts, vim mode, `keybindings.json`
- [subagents.md](./subagents.md) — custom subagents and `Agent(...)` permission rules
- [dispatch-remote-routines.md](./dispatch-remote-routines.md) — Remote Control, Agent View, Routines, channels
- [mcp.md](../capabilities/mcp.md) — Model Context Protocol transports and `.mcp.json`
- [connectors.md](../capabilities/connectors.md) — connector catalog, OAuth, admin controls
- [plugins.md](../capabilities/plugins.md) — plugins, marketplaces, `enabledPlugins`
- [skills.md](../capabilities/skills.md) — Agent Skills and skill-related toggles
- [model-families.md](../models/model-families.md) — model IDs, pricing, context windows
- [capabilities-and-modes.md](../models/capabilities-and-modes.md) — thinking, effort, 1M context
- [glossary.md](../glossary.md)

## Open questions / to verify

- Precise **default values** for `MCP_TIMEOUT`, `CLAUDE_CODE_API_KEY_HELPER_TTL_MS`, and `cleanupPeriodDays` (the live settings table does not pin exact numbers; `BASH_DEFAULT_TIMEOUT_MS`=30000 and `BASH_MAX_TIMEOUT_MS`=600000 are confirmed).
- The full canonical set of `CLAUDE_CODE_DISABLE_*` env vars beyond the seven listed (the docs enumerate a subset).
- Exact version floors for newer keys (`sandbox.credentials` `v2.1.187+`, the session-scoped domain prompt `v2.1.191+`) — confirm against the changelog before quoting.
- Whether `disableClaudeAiConnectors` and `disableRemoteControl` are honored from user/project scope or treated as managed-only on every plan (docs describe `disableRemoteControl` as a managed device key).
- The exact `theme` value enum (only `"dark"` is shown as an example in the docs).

## Sources

- [Claude Code — Settings reference](https://code.claude.com/docs/en/settings)
- [Claude Code — Configure permissions](https://code.claude.com/docs/en/permissions)
- [Claude Code — Choose a permission mode](https://code.claude.com/docs/en/permission-modes)
- [Claude Code — Customize your status line](https://code.claude.com/docs/en/statusline)
- [Claude Code — Configure the sandboxed Bash tool](https://code.claude.com/docs/en/sandboxing)
