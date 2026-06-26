---
type: Command Reference
title: Claude Code — Slash Commands
description: Exhaustive reference to every built-in slash command, bundled skill, and custom-command authoring mechanism in Claude Code.
domain: claude-code
tags: [claude-code, slash-commands, commands, skills, custom-commands, slashcommand-tool, cli, bundled-skills, mcp-prompts]
related: [skills, settings, hooks, plugins, subagents, cli-shortcuts, capabilities-and-modes]
resource: https://code.claude.com/docs/en/commands
timestamp: 2026-06-26T00:00:00Z
confidence: high
verified: 2026-06-26
okf_version: "0.1"
sources:
  - https://code.claude.com/docs/en/commands
  - https://code.claude.com/docs/en/slash-commands
  - https://code.claude.com/docs/en/skills
  - https://code.claude.com/docs/en/agent-sdk/slash-commands
  - https://code.claude.com/docs/en/sub-agents
  - https://code.claude.com/docs/en/settings
  - https://code.claude.com/docs/en/memory
  - https://code.claude.com/docs/en/output-styles
---

# Claude Code — Slash Commands

Slash commands control a Claude Code session from inside it: typed at the **start** of a message (`/help`, `/model sonnet`, `/clear`), they switch models, manage permissions, clear or compact context, run review workflows, and orchestrate background agents. Anything you type after the command name is passed to it as **arguments**. Claude Code today ships well over 90 commands; this page covers the built-ins plus the **custom command / skill** authoring system that lets you add your own.

> **Important 2026 change — custom commands merged into skills.** A file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md` both create `/deploy` and work the same way. Existing `.claude/commands/` files keep working; skills are the recommended, more capable successor. This page documents both. See [`../capabilities/skills.md`](../capabilities/skills.md) for the full Agent Skills model.

## At a glance

| | |
|---|---|
| **What it is** | In-session commands prefixed with `/`. Mix of *built-in* commands (hard-coded CLI logic), *bundled skills* (prompt-driven, Claude can auto-invoke), *bundled workflows* (fan out to subagents), and *custom* commands/skills you author. |
| **Where you find it** | Any interactive Claude Code session (terminal, IDE terminal, desktop, web). Type `/` to open the autocomplete menu; type `/` + letters to filter. Only recognized at the very start of a message. |
| **Who can use it by plan** | Most commands work on any plan. Some are gated: `/upgrade` (Pro/Max only), `/privacy-settings` (Pro/Max), `/desktop` (macOS/Windows + subscription), `/voice`/`/teleport` (claude.ai account), `/ultrareview` credits (3 free runs on Pro/Max). Availability also varies by **version, platform, and environment**. |
| **Status** | Actively evolving; commands are added, renamed, and removed across CLI releases. Version gates noted inline below. Run `/release-notes` for the changelog. |

**How a message is parsed.** A command must be the first token. `/model sonnet please` sets the model and passes `sonnet please` as arguments; `please /model` is treated as plain prose. To see the live, version-correct set for *your* install, type `/` or run `/help`.

---

## Part A — Built-in commands & bundled skills

The table is grouped by purpose for scannability. Legend: **Skill** = bundled prompt-based skill (Claude can auto-invoke; appears in `/skills`); **Workflow** = bundled dynamic workflow that fans out to subagents; everything else is a hard-coded built-in. `<arg>` is required, `[arg]` optional.

> Bundled skills and workflows (`/code-review`, `/batch`, `/debug`, `/loop`, `/claude-api`, `/deep-research`, …) can be removed entirely with `"disableBundledSkills": true` in [settings](./settings.md). When set, built-in slash commands like `/init` stay *typable* but are hidden from the model.

### Help, status & diagnostics

| Command | What it does | Example |
|---|---|---|
| `/help` | Show help and the list of available commands. | `/help` |
| `/status` | Open Settings on the **Status** tab (version, model, account, connectivity). Works while Claude is mid-response. | `/status` |
| `/doctor` | Diagnose and verify your install and settings; results show status icons. Press `f` to have Claude fix reported issues. | `/doctor` |
| `/debug [description]` | **Skill.** Enable debug logging from this point forward and analyze the session debug log. Optionally describe the issue. | `/debug streaming stalls after tool calls` |
| `/release-notes` | View the changelog in an interactive version picker. | `/release-notes` |
| `/feedback [report]` | Submit feedback / report a bug / share your conversation (session context attached). Aliases: `/bug`, `/share`. | `/bug rendering glitch in diff viewer` |
| `/insights` | Generate a report on your sessions: project areas, interaction patterns, friction points. | `/insights` |
| `/heapdump` | Write a JS heap snapshot + memory breakdown to `~/Desktop` (home dir on Linux) for diagnosing high memory. | `/heapdump` |
| `/powerup` | Discover features via quick interactive lessons with animated demos. | `/powerup` |

> The older `/cost`, `/stats`, and `/usage` triplet are now **aliases**: `/usage` is canonical (session cost, plan limits, activity stats; per-skill/subagent/plugin/MCP breakdown on Pro/Max/Team/Enterprise). `/cost` is a plain alias; `/stats` opens on the Stats tab.

### Context management

| Command | What it does | Example |
|---|---|---|
| `/clear [name]` | Start a fresh conversation with empty context; the old one stays in `/resume`. Pass a name to label it. Aliases: `/reset`, `/new`. | `/clear pre-refactor` |
| `/compact [instructions]` | Summarize the conversation so far to free context, optionally with focus instructions. Continues the *same* conversation (unlike `/clear`). | `/compact keep the API contract decisions` |
| `/context [all]` | Visualize context usage as a colored grid; flags context-heavy tools, memory bloat, capacity warnings. `all` expands the per-item breakdown in fullscreen. | `/context all` |
| `/rewind` | Roll conversation and/or code back to a checkpoint, or summarize from a selected message. Aliases: `/checkpoint`, `/undo`. As of recent builds it can restore context from before a `/clear`. | `/rewind` |
| `/btw <question>` | Ask a quick side question without adding it to the main conversation. | `/btw what does ETIMEDOUT mean here?` |
| `/copy [N]` | Copy the last assistant response to clipboard; `N` copies the Nth-latest. Code-block picker appears when present (press `w` to write to a file — handy over SSH). | `/copy 2` |
| `/export [filename]` | Export the conversation as plain text. With a filename, writes directly; without, opens a copy/save dialog. | `/export session.txt` |
| `/recap` | Generate a one-line summary of the current session on demand. | `/recap` |

### Model, effort & mode

| Command | What it does | Example |
|---|---|---|
| `/model [model]` | Switch model and save as default for new sessions; for capable models use ←/→ to adjust effort. No arg → picker (press `s` to switch for the current session only). | `/model opus` |
| `/effort [level\|auto]` | Set effort: `low`, `medium`, `high`, `xhigh`, `max`, `ultracode` (availability depends on model; `max`/`ultracode` are session-only). `auto` resets to default. No arg → slider. Takes effect immediately. | `/effort xhigh` |
| `/fast [on\|off]` | Toggle [fast mode](../models/capabilities-and-modes.md). | `/fast on` |
| `/plan [description]` | Enter plan mode directly; optional description starts plan mode on that task. | `/plan fix the auth bug` |
| `/goal [condition\|clear]` | Set a goal Claude keeps working toward across turns until met. `clear`/`stop`/`off`/`reset`/`none`/`cancel` removes it. | `/goal all tests pass and lint is clean` |
| `/advisor [model\|off]` | *(v2.1.98+)* Enable/disable the advisor tool that consults a second model at key moments. Accepts `opus`, `sonnet`, `fable` (v2.1.170+), or a model ID. | `/advisor opus` |

### Project setup & memory

| Command | What it does | Example |
|---|---|---|
| `/init` | Initialize the project with a starter `CLAUDE.md`. Set `CLAUDE_CODE_NEW_INIT=1` for an interactive flow that also covers skills, hooks, and personal memory. | `/init` |
| `/memory` | Edit `CLAUDE.md`/`CLAUDE.local.md`/rules files loaded this session, toggle [auto-memory](./settings.md), and open the auto-memory folder. Select a file to open it in your editor. | `/memory` |
| **Natural-language memory** | *Not a slash command.* The documented quick path to add a memory is to just **ask Claude**: "remember that the API tests need a local Redis," or "add this to CLAUDE.md." Claude routes preferences/learnings to auto-memory and explicit "add to CLAUDE.md" requests to the file. | `remember: always run pnpm, never npm` |
| **`#` shortcut** | *Not a slash command, and not in current docs.* WARN: verify — a `#`-prefixed quick-add memory line existed in older builds, but the current [memory docs](https://code.claude.com/docs/en/memory) document only `/memory` and natural-language requests. Don't rely on `#`; use `/memory` or ask Claude. | `# legacy quick-memory (may be inert)` |
| `/agents` | Manage [subagent](./subagents.md) configurations (the manager UI). | `/agents` |
| `/skills` | List available skills. Press `t` to sort by token count; `Space` to hide a skill from Claude or the `/` menu, `Enter` to save. | `/skills` |
| `/mcp [...]` | Manage [MCP](../capabilities/mcp.md) servers and OAuth. `reconnect <server>`, `enable`/`disable [<server>\|all]`, or no arg for the interactive list. | `/mcp reconnect github` |
| `/hooks` | View [hook](./hooks.md) configurations for tool events. | `/hooks` |
| `/permissions` | Manage allow/ask/deny rules in an interactive dialog; review recent auto-mode denials. Alias: `/allowed-tools`. | `/permissions` |
| `/add-dir <path>` | Add a working directory for file access this session. Note `.claude/skills/` *is* loaded from added dirs, but most other config is not. | `/add-dir ../shared-lib` |
| `/cd <path>` | *(v2.1.169+)* Move the session to a new working directory, preserving the prompt cache (new `CLAUDE.md` appended, not rebuilt). Restrict targets with `Cd` permission rules. | `/cd ../api` |
| `/plugin [subcommand]` | Manage [plugins](../capabilities/plugins.md): no arg opens the menu; `list`/`install`/`enable`/`disable` act directly. | `/plugin install skill-creator@claude-plugins-official` |
| `/reload-plugins [--force]` | Reload active plugins to apply pending changes without restarting. `--force` if the reload would invalidate the prompt cache. | `/reload-plugins` |
| `/reload-skills` | *(v2.1.152+)* Re-scan skill/command directories so on-disk additions become available without restarting. | `/reload-skills` |

### Review, ship & quality (mostly skills/workflows)

| Command | What it does | Example |
|---|---|---|
| `/diff` | Interactive diff viewer: uncommitted changes and per-turn diffs (←/→ to switch turns, ↑/↓ to browse files). | `/diff` |
| `/code-review [low\|medium\|high\|xhigh\|max\|ultra] [--fix] [--comment] [target]` | **Skill.** Review the current diff for correctness bugs + cleanups. `--fix` applies findings, `--comment` posts inline GitHub PR comments, `ultra` runs a deep cloud review. | `/code-review high --fix` |
| `/review [PR]` | Review a GitHub PR by number using the same engine as `/code-review`; no arg lists open PRs. | `/review 482` |
| `/simplify [target]` | *(v2.1.154+)* **Skill.** Cleanup-only review (reuse, simplification, efficiency, abstraction level) that applies fixes — does **not** hunt for bugs. | `/simplify src/parser` |
| `/security-review` | Analyze pending branch changes for security vulnerabilities (injection, auth, data exposure). | `/security-review` |
| `/ultrareview [PR]` | Deep multi-agent cloud review. Preferred form is now `/code-review ultra`; this remains an alias. 3 free runs on Pro/Max, then credits. | `/ultrareview` |
| `/batch <instruction>` | **Skill.** Decompose a large change into 5–30 units, plan, then run one background subagent per unit in its own git worktree, each opening a PR. | `/batch migrate src/ from Solid to React` |
| `/deep-research <question>` | **Workflow.** Fan out web searches, fetch/cross-check sources, synthesize a cited report. | `/deep-research current Postgres vs SQLite tradeoffs for edge` |
| `/run` | *(v2.1.145+)* **Skill.** Launch and drive your app to see a change working in the running app. | `/run` |
| `/verify` | *(v2.1.145+)* **Skill.** Build + run the app and observe the result to confirm a change works (not just tests). | `/verify` |
| `/run-skill-generator` | *(v2.1.145+)* **Skill.** Record a per-project recipe (`.claude/skills/run-<name>/`) so `/run` and `/verify` know how to build/launch. | `/run-skill-generator` |
| `/fewer-permission-prompts` | **Skill.** Scan transcripts for common read-only Bash/MCP calls and add an allowlist to project `.claude/settings.json`. | `/fewer-permission-prompts` |
| `/claude-api [migrate\|managed-agents-onboard]` | **Skill.** Load Claude API reference for your language; `migrate` upgrades existing API code to a newer model; `managed-agents-onboard` walks through a new Managed Agent. | `/claude-api migrate` |

### Sessions, branching & background work

| Command | What it does | Example |
|---|---|---|
| `/resume [session]` | Resume a conversation by ID/name, or open the picker. Background sessions appear marked `bg`. Alias: `/continue`. | `/resume` |
| `/branch [name]` | Branch the conversation at this point and switch into the copy; return to the original with `/resume`. | `/branch try-redis` |
| `/fork <directive>` | *(v2.1.161+)* Spawn a forked background subagent that inherits the full conversation, works on the directive, and returns its result. Before 2.1.161, `/fork` aliases `/branch`. | `/fork write migration tests for the new schema` |
| `/background [prompt]` | Detach the session to run as a background agent and free the terminal. Alias: `/bg`. Monitor with `claude agents`. | `/background keep iterating on the failing test` |
| `/tasks` | View/manage everything running in the background. Also `/bashes`. | `/tasks` |
| `/stop` | Stop the current background session (only while attached); transcript and worktree kept. | `/stop` |
| `/workflows` | Open the workflow progress view to watch/pause/resume/save workflows. | `/workflows` |
| `/exit` | Exit the CLI. In an attached background session this detaches and the session keeps running. Alias: `/quit`. | `/exit` |
| `/rename [name]` | Rename the session (shown on the prompt bar). No name → auto-generated. | `/rename auth-refactor` |

### Cloud, remote & cross-device

| Command | What it does | Example |
|---|---|---|
| `/teleport` | Pull a Claude Code on the web session into this terminal (picker → fetch branch + conversation). Also `/tp`. Requires a claude.ai subscription. | `/teleport` |
| `/remote-control` | Make this session controllable from claude.ai. Alias: `/rc`. | `/remote-control` |
| `/desktop` | Continue the session in the Claude Code Desktop app (macOS/Windows + subscription). Alias: `/app`. | `/desktop` |
| `/mobile` | Show a QR code to download the mobile app. Aliases: `/ios`, `/android`. | `/mobile` |
| `/schedule [description]` | Create/update/list/run [routines](./dispatch-remote-routines.md) on Anthropic-managed cloud. Alias: `/routines`. | `/schedule run the nightly dependency audit at 2am` |
| `/loop [interval] [prompt]` | **Skill.** Run a prompt repeatedly while the session stays open; omit the interval and Claude self-paces between iterations. Omit the prompt too and (where available) Claude runs an autonomous maintenance check, or the prompt stored in `.claude/loop.md`. Alias: `/proactive`. | `/loop 5m check if the deploy finished` |
| `/autofix-pr [prompt]` | Spawn a web session that watches the current branch's PR and pushes fixes on CI failure / review comments. Requires `gh`. | `/autofix-pr only fix lint and type errors` |
| `/ultraplan <prompt>` | Draft a plan in an ultraplan session, review in browser, then execute remotely or send back to the terminal. | `/ultraplan add multi-tenant support` |
| `/web-setup` | Connect GitHub to Claude Code on the web using your local `gh` credentials. | `/web-setup` |
| `/remote-env` | Choose the default environment for cloud agents. | `/remote-env` |

### Accounts, install & integrations

| Command | What it does | Example |
|---|---|---|
| `/login` | Sign in to your Anthropic account. | `/login` |
| `/logout` | Sign out. | `/logout` |
| `/upgrade` | Open the upgrade page (Pro/Max only). | `/upgrade` |
| `/usage-credits` | Configure usage credits to keep working past a limit. Previously `/extra-usage`. | `/usage-credits` |
| `/privacy-settings` | View/update privacy settings (Pro/Max only). | `/privacy-settings` |
| `/install-github-app` | Install the Claude GitHub App for a repo; optional GitHub Actions setup. | `/install-github-app` |
| `/install-slack-app` | Install the Claude Slack app via OAuth. | `/install-slack-app` |
| `/ide` | Manage IDE integrations and show status. | `/ide` |
| `/chrome` | Configure Claude in Chrome settings. | `/chrome` |
| `/setup-bedrock` | Configure Amazon Bedrock (only when `CLAUDE_CODE_USE_BEDROCK=1`). | `/setup-bedrock` |
| `/setup-vertex` | Configure Google Vertex AI (only when `CLAUDE_CODE_USE_VERTEX=1`). | `/setup-vertex` |
| `/team-onboarding` | Generate a team onboarding guide from your last 30 days of usage. | `/team-onboarding` |
| `/passes` | Share a free week of Claude Code (only if eligible). | `/passes` |
| `/stickers` | Order Claude Code stickers. | `/stickers` |

### Appearance, terminal & input

| Command | What it does | Example |
|---|---|---|
| `/config [key=value ...]` | Open Settings, or (v2.1.181+) set keys directly: `/config thinking=false`, `/config theme=dark`, `/config model=sonnet`. Works in `-p` and Remote Control. `--help` lists every key. Alias: `/settings`. | `/config theme=dark model=sonnet` |
| `/theme` | Change color theme: `auto`, light/dark variants, daltonized, ANSI, and custom themes from `~/.claude/themes/` or plugins. | `/theme` |
| `/color [color\|default]` | Set the prompt-bar color for this session (`red`/`blue`/`green`/`yellow`/`purple`/`orange`/`pink`/`cyan`; `default` to reset; no arg → random). | `/color cyan` |
| `/statusline` | Configure the [status line](./cli-and-shortcuts.md): describe what you want, or run with no args to auto-configure from your shell prompt. | `/statusline show git branch and model` |
| `/output-style` | **Removed.** Deprecated in v2.1.73, removed in v2.1.91. Set the [output style](./settings.md) via `/config` → **Output style**, or the `outputStyle` setting (`Default`, `Proactive`, `Explanatory`, `Learning`, or a custom style). Takes effect after `/clear` or a new session. | `/config` → Output style |
| `/keybindings` | Open your keyboard-shortcuts file (`~/.claude/keybindings.json`). | `/keybindings` |
| `/terminal-setup` | Configure terminal keybindings (Shift+Enter etc.). Only shows in terminals that need it (VS Code, Cursor, Zed, Alacritty…). | `/terminal-setup` |
| `/tui [default\|fullscreen]` | Set the TUI renderer and relaunch into it with the conversation intact. | `/tui fullscreen` |
| `/focus` | Toggle focus view (last prompt + one-line tool summaries + final response). Fullscreen only. | `/focus` |
| `/scroll-speed` | Adjust mouse-wheel scroll speed (fullscreen only; not JetBrains terminal). | `/scroll-speed` |
| `/voice [hold\|tap\|off]` | Toggle voice dictation or enable a mode. Requires a Claude.ai account. | `/voice tap` |
| `/sandbox` | Toggle sandbox mode (supported platforms only). | `/sandbox` |
| `/radio` | Open Claude FM lo-fi radio (not on Bedrock/Vertex/Foundry). | `/radio` |
| `/vim` | **Removed in v2.1.92.** Toggle Vim/Normal editing via `/config` → Editor mode instead. | `/config` → Editor mode |

### Removed / renamed (don't rely on these)

| Command | Status |
|---|---|
| `/vim` | Removed v2.1.92 — use `/config` → Editor mode. |
| `/pr-comments [PR]` | Removed in v2.1.91 — ask Claude directly to view PR comments. |
| `/output-style` | Deprecated v2.1.73, removed v2.1.91 — use `/config` → Output style or the `outputStyle` setting. |
| `/extra-usage` | Renamed → `/usage-credits`. |
| `/cost`, `/stats` | Now aliases for `/usage`. |
| `/proactive` | Alias for `/loop`. |
| `/settings`, `/allowed-tools` | Aliases for `/config` and `/permissions`. |

### MCP prompts as commands

MCP servers can expose **prompts** that appear as commands in the form `/mcp__<server>__<prompt>`, dynamically discovered from connected servers. Arguments after the name are passed through to the prompt. Example: a server named `linear` exposing a `create_issue` prompt becomes `/mcp__linear__create_issue`. See [`../capabilities/mcp.md`](../capabilities/mcp.md).

```text
/mcp__github__open_pr title="Fix race condition" base=main
```

---

## Part B — Custom slash commands (skills)

Custom commands are now **skills**. The directory or file name you create becomes the command you type. Both layouts produce the same `/name`:

- **Skill (recommended):** `.claude/skills/<name>/SKILL.md` → `/<name>` (supports supporting files, subagent execution, dynamic context).
- **Legacy command:** `.claude/commands/<name>.md` → `/<name>` (still works; same frontmatter, no supporting-file directory).

If a skill and a command share a name, the **skill wins**.

### Where command/skill files live

| Location | Path | Applies to | Command name source |
|---|---|---|---|
| Personal | `~/.claude/skills/<name>/SKILL.md` | all your projects | directory name |
| Project | `.claude/skills/<name>/SKILL.md` | this project (commit to share) | directory name |
| Legacy (personal) | `~/.claude/commands/<name>.md` | all your projects | file name (no ext) |
| Legacy (project) | `.claude/commands/<name>.md` | this project | file name (no ext) |
| Plugin | `<plugin>/skills/<name>/SKILL.md` | where the plugin is enabled | `plugin:name` namespace |
| Enterprise | managed settings dir | whole org | directory name |

**Precedence:** enterprise > personal > project, and any of these overrides a bundled skill of the same name (e.g. a project `code-review` skill replaces the bundled `/code-review`). Plugin skills use a `plugin-name:skill-name` namespace so they never collide.

**Namespacing via subfolders.** Project skills load from `.claude/skills/` in your starting directory and **every parent up to the repo root**; nested directories below the cwd load on demand when Claude touches a file there. With a `deploy` skill at the repo root and another at `apps/web/.claude/skills/deploy/`, `/deploy` runs the root one and the nested one is reachable as `/apps/web:deploy`.

**Live change detection.** Adding/editing/removing a `SKILL.md` under `~/.claude/skills/`, the project `.claude/skills/`, or an `--add-dir` directory's `.claude/skills/` takes effect *within the session* — no restart. Creating a top-level skills directory that didn't exist at launch requires a restart (so it can be watched); or run `/reload-skills` (v2.1.152+) to force a re-scan.

**Skill folder as a plugin.** Drop a `.claude-plugin/plugin.json` into a skill folder and it loads as a plugin named `<name>@skills-dir`, letting it bundle agents, hooks, and MCP servers. In a project's `.claude/skills/` this requires accepting the workspace-trust dialog. Changes to its `hooks/`, `.mcp.json`, `agents/`, `output-styles/` need `/reload-plugins` (live detection only covers `SKILL.md` text).

### Frontmatter keys

Frontmatter sits between `---` markers at the top of `SKILL.md` (or the legacy `.md` file). All fields are optional; `description` is recommended.

| Key | Purpose |
|---|---|
| `name` | Display label in listings (defaults to directory name). For non-plugin-root skills it does **not** change what you type. |
| `description` | What the skill does and when to use it — Claude matches on this to auto-invoke. Combined with `when_to_use`, capped at 1,536 chars in the listing. |
| `when_to_use` | Extra trigger phrases / example requests, appended to `description`. |
| `argument-hint` | Autocomplete hint, e.g. `[issue-number]` or `[filename] [format]`. |
| `arguments` | Named positional args for `$name` substitution; space-separated string or YAML list, mapped in order. |
| `allowed-tools` | Tools pre-approved (no permission prompt) while the skill is active. Space/comma string or YAML list. |
| `disallowed-tools` | Tools removed from the pool while active (clears on your next message). |
| `disable-model-invocation` | `true` = only you can invoke it (Claude won't auto-trigger; also not preloaded into subagents). Use for `/deploy`, `/commit`, etc. |
| `user-invocable` | `false` = hidden from the `/` menu; only Claude can invoke. For background knowledge. |
| `model` | Model override for the turn (same values as `/model`, or `inherit`). |
| `effort` | Effort override (`low`…`max`). |
| `context` | `fork` runs the skill in an isolated subagent context: the SKILL.md body becomes the subagent's task prompt and it gets **no** conversation history. Only meaningful for skills with explicit instructions (pure reference content yields no actionable prompt). |
| `agent` | Which subagent type runs it when `context: fork` — `Explore`, `Plan`, `general-purpose`, or any custom agent from `.claude/agents/`. **Defaults to `general-purpose`** if omitted. `Explore`/`Plan` skip CLAUDE.md + git status to stay small. |
| `hooks` | Hooks scoped to this skill's lifecycle — see [`./hooks.md`](./hooks.md). |
| `paths` | Glob patterns limiting auto-activation to matching files. |
| `shell` | `bash` (default) or `powershell` for inline `!` commands (PowerShell requires `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`). |

### Arguments and string substitution

| Token | Expands to |
|---|---|
| `$ARGUMENTS` | The full argument string as typed. If absent from the body, args are appended as `ARGUMENTS: <value>`. |
| `$ARGUMENTS[N]` | Argument at 0-based index `N` (shell-style quoting; quote multi-word values). |
| `$N` | Shorthand for `$ARGUMENTS[N]` — `$0` is the first arg, `$1` the second. |
| `$name` | Named arg declared in `arguments:` frontmatter, mapped by position. |
| `${CLAUDE_SESSION_ID}` | Current session ID. |
| `${CLAUDE_EFFORT}` | Current effort level (`low`…`max`; ultracode reports `xhigh`). |
| `${CLAUDE_SKILL_DIR}` | The directory containing this `SKILL.md` — use it to reference bundled scripts. |

Escape a literal `$` before a digit/`ARGUMENTS`/name with a single backslash: `\$1.00` stays literal.

### Dynamic bash injection (`!`) and file references (`@`)

Two preprocessing features run **before** Claude sees the content:

- **Inline bash:** `` !`<command>` `` runs the shell command and replaces the placeholder with its output. Only recognized when `!` is at the start of a line or right after whitespace (so `` KEY=!`cmd` `` stays literal). Substitution runs once; output is not re-scanned.
- **Multi-line bash:** open a fenced block with ` ```! ` for several commands.
- **File references:** `@path/to/file` inlines a file's contents into the prompt, e.g. `@src/config.ts` or `@README.md`.

Disable shell injection org-wide with `"disableSkillShellExecution": true` in [settings](./settings.md) (each command becomes `[shell command execution disabled by policy]`; bundled/managed skills are exempt).

### The Skill tool (model-invoked)

Skills without `disable-model-invocation: true` are exposed to Claude through the **Skill tool** (the model-invocation mechanism; "SlashCommand" is the older/SDK name for the same idea), so Claude can run them itself when relevant — e.g. you ask "what changed?" and Claude fires your `/summarize-changes` skill. Controls, all expressed against the `Skill` tool in permission rules:

- **Disable all:** add `Skill` to deny rules in `/permissions`.
- **Allow specific:** `Skill(commit)` (exact), `Skill(review-pr *)` (prefix-with-any-args). Syntax: `Skill(name)` = exact, `Skill(name *)` = prefix.
- **Deny specific:** `Skill(deploy *)` in deny rules.
- **Block one skill from the model only:** set `disable-model-invocation: true` in its frontmatter (this also removes its description from Claude's context entirely). Note `user-invocable: false` only hides the `/` menu entry — it does **not** block Skill-tool access.
- A few built-ins are also reachable via the tool (`/init`, `/review`, `/security-review`); most (like `/compact`) are not.

**Listing budget.** All skill *names* are always listed; *descriptions* share a budget that scales at **1% of the model's context window**. On overflow, the least-invoked skills' descriptions are dropped first. Raise it with the `skillListingBudgetFraction` setting (e.g. `0.02` = 2%) or pin a fixed character count via the `SLASH_COMMAND_TOOL_CHAR_BUDGET` environment variable. Each entry's combined `description` + `when_to_use` is independently capped at **1,536 chars** (`maxSkillDescriptionChars`). Run `/doctor` to see how many descriptions are being shortened or dropped.

**Lifecycle.** An invoked skill's rendered body enters the conversation as a single message and stays in context for the rest of the session; Claude does **not** re-read the file on later turns — write standing instructions, not one-time steps. Across `/compact`, Claude re-attaches the most recent invocation of each skill (first 5,000 tokens each, sharing a combined ~25,000-token budget, newest-first), so older skills can be dropped — re-invoke a large skill after compaction to restore its full content.

### Overriding skill visibility from settings (`skillOverrides`)

Beyond per-skill frontmatter, the `skillOverrides` setting controls visibility without editing the `SKILL.md` (useful for shared-repo or MCP-provided skills). The `/skills` menu writes it for you (`Space` cycles state, `Enter` saves to `.claude/settings.local.json`). Each key is a skill name; values:

| Value | Listed to Claude | In `/` menu |
|---|---|---|
| `"on"` (default if absent) | name + description | yes |
| `"name-only"` | name only (frees description budget) | yes |
| `"user-invocable-only"` | hidden from Claude | yes |
| `"off"` | hidden | hidden |

Plugin skills are not affected by `skillOverrides` — manage those via `/plugin`.

### Worked example: authoring `/open-pr`

Goal: a command that opens a pull request, pre-filled with the live diff and a generated title, that **only you** can trigger.

```bash
mkdir -p ~/.claude/skills/open-pr
```

`~/.claude/skills/open-pr/SKILL.md`:

````markdown
---
name: open-pr
description: Open a GitHub pull request for the current branch with a generated title and body. Use when the user asks to open or create a PR.
argument-hint: "[base-branch]"
arguments: [base]
disable-model-invocation: true
allowed-tools: Bash(git *) Bash(gh *)
model: sonnet
---

## Current state
- Branch: !`git rev-parse --abbrev-ref HEAD`
- Base: $base
- Diff vs base: !`git diff $base...HEAD`
- Recent commits: !`git log --oneline -10`

## Project conventions
@.github/PULL_REQUEST_TEMPLATE.md

## Your task
1. Summarize the diff above into a concise PR title and a body that follows the template.
2. Push the branch if it has no upstream.
3. Run: `gh pr create --base $base --title "<title>" --body "<body>"`.
4. Print the PR URL.
````

Invoke it:

```text
/open-pr main
```

What happens: `$base` → `main`; the `!`git ...`` placeholders execute and inline real branch/diff/commit data; `@.github/PULL_REQUEST_TEMPLATE.md` is inlined; `allowed-tools` lets Claude run `git`/`gh` without per-call prompts; `model: sonnet` runs the turn on Sonnet; and `disable-model-invocation: true` ensures Claude never opens a PR on its own — only when you type `/open-pr`.

### Legacy equivalent

The same as a legacy command — `~/.claude/commands/open-pr.md` — with identical frontmatter and body. It works, but you lose the skill folder's supporting-file and subagent features, so prefer the skill form.

---

## Related pages

- [`../capabilities/skills.md`](../capabilities/skills.md) — Agent Skills: `SKILL.md`, progressive disclosure, install, full frontmatter model (custom commands are skills).
- [`./settings.md`](./settings.md) — `settings.json`, `disableSkillShellExecution`, `skillListingBudgetFraction`, output styles, permissions.
- [`./hooks.md`](./hooks.md) — hook events you can scope to a skill via the `hooks` frontmatter key.
- [`../capabilities/plugins.md`](../capabilities/plugins.md) — plugins bundle skills/commands under a `plugin:name` namespace; `/plugin`, `/reload-plugins`.
- [`./subagents.md`](./subagents.md) — `context: fork`, the `agent` field, and the Task tool that `/batch` and `/fork` use.
- [`./cli-and-shortcuts.md`](./cli-and-shortcuts.md) — CLI flags, keyboard shortcuts, `/statusline`, `/keybindings`, vim mode.
- [`../capabilities/mcp.md`](../capabilities/mcp.md) — MCP prompts surfaced as `/mcp__<server>__<prompt>` commands.
- [`../models/capabilities-and-modes.md`](../models/capabilities-and-modes.md) — extended thinking, fast mode, effort levels behind `/effort` and `/fast`.

## Open questions / to verify

- **`/output-style`:** RESOLVED — confirmed deprecated v2.1.73, removed v2.1.91. Output styles are now set via `/config` → Output style or the `outputStyle` setting. Documented as removed above.
- **`#` memory shortcut:** RESOLVED against docs — the current [memory docs](https://code.claude.com/docs/en/memory) document **no** `#` quick-add shortcut; the supported paths are `/memory` and natural-language ("remember…", "add this to CLAUDE.md"). The `#` form (if present in any build) is undocumented and should not be relied on.
- **SlashCommand vs Skill tool naming:** RESOLVED for Claude Code — the in-product tool and permission rules use **`Skill`** (`Skill(name)`, `Skill(name *)`). "SlashCommand" persists as legacy/SDK naming; re-verify the SDK page if targeting the Agent SDK specifically.
- **Exact total count of commands:** the live "All commands" table currently lists ~95 entries (a couple marked removed). It is version-dependent — type `/` in a live session for the authoritative set for your install.
- **`/todos`:** not present as a distinct command in the current commands reference; background/turn task tracking surfaces via `/tasks` (alias `/bashes`) and the TodoWrite tool. No `/todos` command found in current docs.
- Several commands carry version gates (e.g. `/cd` v2.1.169+, `/fork` v2.1.161+, `/advisor` v2.1.98+ with `fable` from v2.1.170+, `/simplify` v2.1.154+, `/reload-skills` v2.1.152+, `/run` `/verify` `/run-skill-generator` v2.1.145+, `/config key=value` v2.1.181+ and named shorthand v2.1.182+); re-verify minimum versions against `/release-notes` for the install you target.

## Sources

- [Commands reference — code.claude.com/docs/en/commands](https://code.claude.com/docs/en/commands)
- [Slash commands / Extend Claude with skills — code.claude.com/docs/en/slash-commands](https://code.claude.com/docs/en/slash-commands)
- [Skills (bundled skills, frontmatter, substitutions, injection) — code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)
- [Slash Commands in the SDK — code.claude.com/docs/en/agent-sdk/slash-commands](https://code.claude.com/docs/en/agent-sdk/slash-commands)
- [Subagents — code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents)
- [Settings reference — code.claude.com/docs/en/settings](https://code.claude.com/docs/en/settings)
- [Memory / CLAUDE.md & auto-memory — code.claude.com/docs/en/memory](https://code.claude.com/docs/en/memory) (verified: no `#` quick-memory shortcut)
- [Output styles — code.claude.com/docs/en/output-styles](https://code.claude.com/docs/en/output-styles) (verified: `/output-style` removed v2.1.91)

*Verified 2026-06-26 against the live `code.claude.com` docs. The Part A command table and Part B skills system were cross-checked against the official "All commands" table and "Extend Claude with skills" page; corrections are listed in this page's review metadata.*
