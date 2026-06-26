---
type: Command Reference
title: Claude Code — CLI Flags & Keyboard Shortcuts
description: Exhaustive reference to the `claude` binary's subcommands and flags (interactive, one-shot, and headless) plus every interactive-mode keyboard shortcut and vim-mode keybinding.
domain: claude-code
tags: [claude-code, cli, flags, headless, keyboard-shortcuts, vim, tui]
related: [claude-code-overview, slash-commands, settings, hooks, subagents, dispatch-remote, cli-shortcuts]
resource: https://code.claude.com/docs/en/cli-reference
timestamp: 2026-06-26T00:00:00Z
confidence: high
verified: 2026-06-26
okf_version: "0.1"
sources:
  - https://code.claude.com/docs/en/cli-reference
  - https://code.claude.com/docs/en/interactive-mode
  - https://code.claude.com/docs/en/headless
  - https://code.claude.com/docs/en/terminal-config
  - https://code.claude.com/docs/en/permission-modes
  - https://code.claude.com/docs/en/agent-view
  - https://code.claude.com/docs/en/checkpointing
  - https://code.claude.com/docs/en/worktrees
  - https://code.claude.com/docs/en/keybindings
---

# Claude Code — CLI Flags & Keyboard Shortcuts

The `claude` binary is the single entry point to Claude Code: run it with no arguments for an **interactive** terminal session, with a quoted prompt to seed that session, or with `-p`/`--print` for a **headless** one-shot you can pipe and script. This page is the exhaustive reference to that command line — every documented subcommand and flag in **Part A** — and to the in-session **keyboard shortcuts and vim mode** in **Part B**. Flags marked print-mode-only have no effect in an interactive session.

## At a glance

| | |
|---|---|
| **What it is** | The `claude` CLI (subcommands + flags) and the interactive TUI's keyboard map. |
| **Where you find it** | Any terminal where Claude Code is installed (`claude --version` to check). Flags go on the command line; shortcuts work inside an interactive session. |
| **Who can use it** | Anyone with Claude Code. Some flags need a plan/account (`--remote`, `--teleport`, `--channels` require claude.ai auth; `--betas` is API-key only). Shortcut availability varies by **terminal and platform**. |
| **Status** | Actively evolving; flags and shortcuts are added/removed across CLI releases. `claude --help` does **not** list every flag — absence from `--help` does not mean a flag is unavailable. Run `/release-notes` for the changelog. |

> **Note on `--help`.** The official docs state plainly: *"`claude --help` does not list every flag, so a flag's absence from `--help` does not mean it is unavailable."* This page mirrors the full published [CLI reference](https://code.claude.com/docs/en/cli-reference), which is the authoritative superset.

---

# Part A — The `claude` CLI

## A.1 Three ways to run it

| Mode | Invocation | Behavior |
|---|---|---|
| **Interactive** (default) | `claude` | Opens the full TUI: streaming responses, slash commands, transcript viewer, keyboard shortcuts. Maintains session state for follow-ups. |
| **Interactive + seed prompt** | `claude "explain this project"` | Same interactive session, but starts with that prompt already submitted. You can keep chatting after. |
| **Headless / print** | `claude -p "explain this function"` | Non-interactive: runs the query, prints the result to stdout, exits. Pipe-friendly and scriptable. Most interactive features (the `/` menu, live shortcuts) are unavailable. |

```bash
# Interactive
claude

# Interactive with an initial prompt
claude "refactor the auth module to use async/await"

# Headless one-shot; result goes to stdout
claude -p "summarize what this repo does"

# Headless with piped stdin (stdin is capped at 10MB)
cat build-error.txt | claude -p "explain the root cause and suggest a fix" > fix.md
```

## A.2 Subcommands (the `claude` verbs)

These are documented in the official [CLI commands table](https://code.claude.com/docs/en/cli-reference). `<id>` is a background-session ID.

### Session entry points

| Command | What it does | Example |
|---|---|---|
| `claude` | Start an interactive session. | `claude` |
| `claude "query"` | Start interactive with an initial prompt. | `claude "explain this project"` |
| `claude -p "query"` | Query headless (print mode), then exit. | `claude -p "explain this function"` |
| `cat file \| claude -p "query"` | Process piped content headless. | `cat logs.txt \| claude -p "explain"` |
| `claude -c` | Continue the most recent conversation in the current directory. | `claude -c` |
| `claude -c -p "query"` | Continue headless. | `claude -c -p "check for type errors"` |
| `claude -r "<session>" "query"` | Resume a session by ID or name with a new prompt. | `claude -r "auth-refactor" "finish this PR"` |

### Install, update & auth

| Command | What it does | Example |
|---|---|---|
| `claude --version` / `-v` | Print the version number. | `claude -v` |
| `claude update` | Update to the latest version. (Typos like `claude udpate` print *"Did you mean claude update?"* and exit.) | `claude update` |
| `claude install [version]` | Install/reinstall the native binary; accepts `stable`, `latest`, or a version like `2.1.118`. | `claude install stable` |
| `claude auth login` | Sign in. Flags: `--email` (pre-fill), `--sso` (force SSO), `--console` (Anthropic Console / API billing). | `claude auth login --console` |
| `claude auth logout` | Sign out. | `claude auth logout` |
| `claude auth status` | Print auth status as JSON (`--text` for human-readable); exit 0 if logged in, 1 if not. | `claude auth status` |
| `claude setup-token` | Generate a long-lived OAuth token for CI/scripts; printed, not saved. Requires a Claude subscription. | `claude setup-token` |

### MCP & plugins

| Command | What it does | Example |
|---|---|---|
| `claude mcp` | Configure MCP servers (has its own subcommands — see [MCP](../capabilities/mcp.md)). | `claude mcp add ...` |
| `claude mcp login <name>` | *(v2.1.186+)* Run an MCP server's OAuth flow from the shell. `--no-browser` prints the URL for SSH. | `claude mcp login sentry` |
| `claude mcp logout <name>` | *(v2.1.186+)* Clear stored OAuth credentials for an MCP server. | `claude mcp logout sentry` |
| `claude plugin` (alias `plugins`) | Manage [plugins](../capabilities/plugins.md); has its own subcommands (`install`, …). | `claude plugin install code-review@claude-plugins-official` |

### Background sessions (agent view)

These manage detached background agents — see [Dispatch/Remote/Routines](./dispatch-remote-routines.md) and [Subagents](./subagents.md).

| Command | What it does | Example |
|---|---|---|
| `claude agents` | Open the agent view; `--json` prints active sessions for scripting (`--json --all` includes completed). Accepts `--cwd`, `--permission-mode`, `--model`, `--effort`, `--agent`, `--settings`, `--add-dir`, `--plugin-dir`, `--mcp-config`. | `claude agents --json` |
| `claude attach <id>` | Attach to a running background session in this terminal. | `claude attach 7c5dcf5d` |
| `claude logs <id>` | Print recent output from a background session. | `claude logs 7c5dcf5d` |
| `claude stop <id>` (alias `kill`) | Stop a background session. | `claude stop 7c5dcf5d` |
| `claude respawn <id>` | Restart a session (running or stopped) with its conversation intact; `--all` for every running session. | `claude respawn 7c5dcf5d` |
| `claude rm <id>` | Remove a session from the list (transcript stays, available via `claude --resume`). | `claude rm 7c5dcf5d` |
| `claude daemon status` | Print the supervisor's state/version/worker count; exit 1 if not running. | `claude daemon status` |
| `claude daemon stop --any` | Stop the supervisor and its sessions; `--keep-workers` leaves sessions running. | `claude daemon stop --any --keep-workers` |

### Remote, review & maintenance

| Command | What it does | Example |
|---|---|---|
| `claude remote-control` | Start a [Remote Control](./dispatch-remote-routines.md) server (no local interactive session); `--name` labels it. | `claude remote-control --name "My Project"` |
| `claude ultrareview [target]` | Run ultrareview non-interactively → stdout, exit 0/1. `--json` for raw payload, `--timeout <min>` (default 30). | `claude ultrareview 1234 --json` |
| `claude auto-mode defaults` | Print the built-in [auto-mode](https://code.claude.com/docs/en/permission-modes) classifier rules as JSON. `claude auto-mode config` shows your effective config. | `claude auto-mode defaults > rules.json` |
| `claude project purge [path]` | Delete all local Claude Code state for a project. Flags: `--dry-run`, `-y`/`--yes`, `-i`/`--interactive`, `--all`. | `claude project purge ~/work/repo --dry-run` |

> **`claude config` / `claude doctor`:** WARN: verify. These are **not** in the current official CLI-commands table. Settings are opened in-session with `/config` (or overridden via `--settings`), and diagnostics run via the `/doctor` slash command. A top-level `claude config` / `claude doctor` may exist in some builds but is not documented; prefer the slash commands. See [Slash commands](./slash-commands.md).

## A.3 Flags

Every flag below is from the official [CLI flags table](https://code.claude.com/docs/en/cli-reference). Flags grouped by purpose; **(print-only)** marks flags that require `-p`/`--print`.

### Session resume & identity

| Flag | What it does | Example |
|---|---|---|
| `--continue`, `-c` | Load the most recent conversation in the current directory. | `claude --continue` |
| `--resume`, `-r` | Resume a session by ID or name, or open the picker. *(v2.1.144+: background sessions appear marked `bg`.)* | `claude --resume auth-refactor` |
| `--fork-session` | When resuming, create a **new** session ID instead of reusing the original (use with `--resume`/`--continue`). | `claude --resume abc123 --fork-session` |
| `--session-id` | Use a specific session ID (must be a valid UUID). | `claude --session-id "550e8400-e29b-41d4-a716-446655440000"` |
| `--from-pr` | Resume sessions linked to a PR (accepts a number or a GitHub/GitLab/Bitbucket PR/MR URL). | `claude --from-pr 123` |
| `--name`, `-n` | Set a display name (shown in `/resume` and the terminal title). | `claude -n "my-feature-work"` |
| `--no-session-persistence` | **(print-only)** Don't save the session to disk; can't be resumed. (`CLAUDE_CODE_SKIP_PROMPT_HISTORY` does this in any mode.) | `claude -p --no-session-persistence "query"` |

### Model & effort

| Flag | What it does | Example |
|---|---|---|
| `--model` | Set the session model: alias (`sonnet`, `opus`, `haiku`, `fable`) or full name. Overrides the `model` setting and `ANTHROPIC_MODEL`. | `claude --model claude-sonnet-4-6` |
| `--effort` | Session [effort level](../models/capabilities-and-modes.md): `low`, `medium`, `high`, `xhigh`, `max` (levels depend on the model). Does not persist. | `claude --effort high` |
| `--fallback-model` | Comma-separated fallback model(s), tried in order when the primary is overloaded/retired. | `claude --fallback-model sonnet,haiku` |
| `--advisor <model>` | *(v2.1.98+)* Enable the server-side [advisor tool](../models/capabilities-and-modes.md): `opus`, `sonnet`, `fable` (v2.1.170+), or a full ID. | `claude --advisor opus` |
| `--betas` | Beta headers to include in API requests (**API-key users only**). | `claude --betas interleaved-thinking` |

> The slash command `/effort` additionally accepts `ultracode` as a session-only level; the documented **CLI** `--effort` table lists `low…max`. WARN: verify whether `--effort ultracode` is accepted on the command line.

### Permissions & tools

| Flag | What it does | Example |
|---|---|---|
| `--permission-mode` | Start in a [permission mode](https://code.claude.com/docs/en/permission-modes): `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, or `bypassPermissions`. Overrides `defaultMode`. | `claude --permission-mode plan` |
| `--allowedTools`, `--allowed-tools` | Tools that run **without** a permission prompt; supports [rule syntax](./settings.md). | `claude --allowedTools "Read" "Bash(git log *)"` |
| `--disallowedTools`, `--disallowed-tools` | Deny rules. A bare name removes the tool from context (`"Edit"`, `"*"`, `"mcp__*"`); a scoped rule (`Bash(rm *)`) denies only matching calls. | `claude --disallowedTools "Edit" "Bash(rm *)"` |
| `--tools` | Restrict built-in tools: `""` (none), `"default"` (all), or `"Bash,Edit,Read"`. (MCP tools unaffected — deny those via `--disallowedTools "mcp__*"`.) | `claude --tools "Bash,Edit,Read"` |
| `--dangerously-skip-permissions` | Skip all permission prompts (≡ `--permission-mode bypassPermissions`). | `claude --dangerously-skip-permissions` |
| `--allow-dangerously-skip-permissions` | Add `bypassPermissions` to the `Shift+Tab` cycle **without** starting in it. | `claude --permission-mode plan --allow-dangerously-skip-permissions` |
| `--permission-prompt-tool` | MCP tool that handles permission prompts in non-interactive mode. | `claude -p --permission-prompt-tool mcp_auth_tool "query"` |

> **Removed:** `--enable-auto-mode` was removed in **v2.1.111** — auto mode is in the `Shift+Tab` cycle by default; use `--permission-mode auto` to start in it.

### Files, directories & config sources

| Flag | What it does | Example |
|---|---|---|
| `--add-dir` | Add working directories for read/edit. Grants **file access only**; most `.claude/` config is not discovered from added dirs. | `claude --add-dir ../apps ../lib` |
| `--settings` | Path to a settings JSON file **or** an inline JSON string; overrides matching keys in `settings.json` for the session. | `claude --settings ./settings.json` |
| `--setting-sources` | Comma-separated sources to load: `user`, `project`, `local`. | `claude --setting-sources user,project` |
| `--mcp-config` | Load MCP servers from JSON files/strings (space-separated). | `claude --mcp-config ./mcp.json` |
| `--strict-mcp-config` | Use **only** servers from `--mcp-config`, ignoring all others. | `claude --strict-mcp-config --mcp-config ./mcp.json` |
| `--agents` | Define [subagents](./subagents.md) inline via JSON (subagent frontmatter field names + a `prompt`). | `claude --agents '{"reviewer":{"description":"Reviews code","prompt":"You are a code reviewer"}}'` |
| `--agent` | Use a named agent for the session (overrides the `agent` setting). | `claude --agent my-custom-agent` |
| `--plugin-dir` | Load a plugin from a directory or `.zip` for this session; repeat for multiple. | `claude --plugin-dir ./my-plugin` |
| `--plugin-url` | Fetch a plugin `.zip` from a URL for this session; repeat or space-separate. | `claude --plugin-url https://example.com/plugin.zip` |
| `--bare` | Minimal mode: skip auto-discovery of hooks/skills/plugins/MCP/auto-memory/CLAUDE.md; only Bash, Read, Edit. Sets `CLAUDE_CODE_SIMPLE`. | `claude --bare -p "query"` |
| `--safe-mode` | *(v2.1.169+)* Start with **all** customizations disabled (CLAUDE.md, skills, plugins, hooks, MCP, themes, keybindings…) — but auth, model, built-in tools, and permissions still work. For troubleshooting. | `claude --safe-mode` |

### System prompt

All four work in both interactive and print mode. `--system-prompt` and `--system-prompt-file` are **mutually exclusive**; the append flags combine with either.

| Flag | What it does | Example |
|---|---|---|
| `--system-prompt` | Replace the entire system prompt with custom text. | `claude --system-prompt "You are a Python expert"` |
| `--system-prompt-file` | Replace the system prompt with a file's contents. | `claude --system-prompt-file ./prompts/review.txt` |
| `--append-system-prompt` | Append custom text to the default prompt. | `claude --append-system-prompt "Always use TypeScript"` |
| `--append-system-prompt-file` | Append a file's contents to the default prompt. | `claude --append-system-prompt-file ./style-rules.txt` |
| `--exclude-dynamic-system-prompt-sections` | Move per-machine sections (cwd, env, memory paths, git flag) into the first user message to improve prompt-cache reuse. Default prompt only; pair with `-p`. | `claude -p --exclude-dynamic-system-prompt-sections "query"` |

> **Rule of thumb:** append when Claude should stay a coding assistant that also follows your extra rules (you keep the default tool guidance + safety instructions); replace only when the identity/permission model genuinely differs (you then own all safety/tool guidance). For persistent personas use [output styles](./settings.md); for project conventions use [CLAUDE.md](./settings.md).

### Headless / print-mode output & input

| Flag | What it does | Example |
|---|---|---|
| `--print`, `-p` | Print the response and exit (non-interactive). | `claude -p "query"` |
| `--output-format` | **(print)** `text` (default), `json`, or `stream-json`. | `claude -p "query" --output-format json` |
| `--input-format` | **(print)** `text` (default) or `stream-json`. | `claude -p --output-format json --input-format stream-json` |
| `--json-schema` | **(print)** Validate the final output against a JSON Schema (structured outputs). | `claude -p --json-schema '{"type":"object","properties":{...}}' "query"` |
| `--include-partial-messages` | Include partial streaming events. Requires `--print` **and** `--output-format stream-json`. | `claude -p --output-format stream-json --verbose --include-partial-messages "query"` |
| `--include-hook-events` | Include hook lifecycle events in the stream. Requires `--output-format stream-json`. | `claude -p --output-format stream-json --verbose --include-hook-events "query"` |
| `--prompt-suggestions` | Emit a `prompt_suggestion` message after each turn. Requires `--print` + `stream-json` + `--verbose`. | `claude -p --prompt-suggestions --output-format stream-json --verbose "query"` |
| `--replay-user-messages` | Re-emit stdin user messages on stdout for acknowledgment. Requires `--input-format stream-json` + `--output-format stream-json`. | `claude -p --input-format stream-json --output-format stream-json --verbose --replay-user-messages` |
| `--verbose` | Full turn-by-turn output. Overrides the `viewMode` setting. | `claude --verbose` |
| `--max-turns` | **(print)** Limit agentic turns; errors when the limit is hit. No limit by default. | `claude -p --max-turns 3 "query"` |
| `--max-budget-usd` | **(print)** Max dollar spend on API calls before stopping. | `claude -p --max-budget-usd 5.00 "query"` |
| `--init` | **(print)** Run Setup hooks with the `init` matcher before the session. | `claude -p --init "query"` |
| `--maintenance` | **(print)** Run Setup hooks with the `maintenance` matcher before the session. | `claude -p --maintenance "query"` |
| `--init-only` | Run Setup + `SessionStart` hooks, then exit without a conversation. | `claude --init-only` |

### Worktrees, IDE, remote & teams

| Flag | What it does | Example |
|---|---|---|
| `--worktree`, `-w` | Start in an isolated git [worktree](https://code.claude.com/docs/en/worktrees) at `<repo>/.claude/worktrees/<name>`; auto-names if omitted; accepts `#<pr>` or a PR URL. | `claude -w feature-auth` |
| `--tmux` | Create a tmux session for the worktree (requires `--worktree`; `--tmux=classic` for traditional tmux). | `claude -w feature-auth --tmux` |
| `--ide` | Auto-connect to an IDE on startup if exactly one is available. | `claude --ide` |
| `--remote` | Create a new web session on claude.ai with a task description. | `claude --remote "Fix the login bug"` |
| `--remote-control`, `--rc` | Start interactive with Remote Control enabled; optional session name. | `claude --remote-control "My Project"` |
| `--teleport` | Pull a Claude Code web session into this terminal. | `claude --teleport` |
| `--bg`, `--background` | Start as a background agent and return immediately (prints session ID + management commands). Combine with `--exec` or `--agent`. | `claude --bg "investigate the flaky test"` |
| `--exec` | Run a shell command as a PTY-backed background job instead of a Claude session (use with `--bg`). | `claude --bg --exec 'pytest -x'` |
| `--teammate-mode` | [Agent-team](https://code.claude.com/docs/en/agent-teams) display: `in-process` (default), `auto`, `tmux`, `iterm2` (v2.1.186+). | `claude --teammate-mode auto` |

### Display, debug & misc

| Flag | What it does | Example |
|---|---|---|
| `--debug` | Debug mode with optional category filter, e.g. `"api,hooks"` or `"!statsig,!file"`. | `claude --debug "api,mcp"` |
| `--debug-file <path>` | Write debug logs to a file (implicitly enables debug). | `claude --debug-file /tmp/claude-debug.log` |
| `--ax-screen-reader` | *(v2.1.181+)* Screen-reader-friendly flat text; forces the classic renderer. Overrides `CLAUDE_AX_SCREEN_READER` / `axScreenReader`. | `claude --ax-screen-reader` |
| `--disable-slash-commands` | Disable all skills/commands for the session. | `claude --disable-slash-commands` |
| `--chrome` / `--no-chrome` | Enable/disable [Chrome browser integration](https://code.claude.com/docs/en/chrome). | `claude --chrome` |
| `--channels` | *(research preview)* MCP servers whose channel notifications to listen for: `plugin:<name>@<marketplace>`. Requires claude.ai auth. | `claude --channels plugin:my-notifier@my-marketplace` |
| `--dangerously-load-development-channels` | Enable non-allowlisted channels for local dev; prompts to confirm. | `claude --dangerously-load-development-channels server:webhook` |

## A.4 Headless output formats

What each `--output-format` emits in print mode:

- **`text`** (default) — the raw text response on stdout. Best for `| grep`, redirecting to a file, etc.
- **`json`** — a single JSON object with the result plus metadata (`result`, `session_id`, token `usage`, `total_cost_usd`, `model`, `stop_reason`; and `structured_output` when `--json-schema` is used). WARN: verify the exact field names against the [Agent SDK reference](https://code.claude.com/docs/en/agent-sdk/overview) for your version.
- **`stream-json`** — newline-delimited JSON, one event per line, as the run progresses. With `--verbose` you also get `system` events (e.g. `init` with model/tools/MCP servers, and `api_retry`). Add `--include-partial-messages` for token-level `text_delta` deltas.

```bash
# Pull just the answer text out of JSON
claude -p "summarize this project" --output-format json | jq -r '.result'

# Track cost of a headless run
claude -p "review src/auth.ts for security issues" --output-format json | jq '.total_cost_usd'

# Stream tokens as they arrive
claude -p "write a haiku about caching" \
  --output-format stream-json --verbose --include-partial-messages \
| jq -rj 'select(.event.delta.type? == "text_delta") | .event.delta.text'
```

## A.5 Headless & CI recipes

```bash
# Capture a session ID, then continue it later in a script
session_id=$(claude -p "analyze the database schema" --output-format json | jq -r '.session_id')
claude -p "now propose indexes for the slow queries" --resume "$session_id"

# CI: no prompts, narrow tool surface, bounded turns
claude -p "run the test suite and summarize failures" \
  --permission-mode dontAsk \
  --allowedTools "Bash(npm run *)" "Read" \
  --max-turns 5

# Fast, deterministic scripted call (skip discovery of skills/plugins/MCP/CLAUDE.md)
claude --bare -p "list the exported symbols in src/index.ts"

# A git-diff "typo linter" as an npm script
#   "lint:claude": "git diff main | claude -p 'report any typos as file:line — nothing else'"
git diff main | claude -p "report any typos as file:line, one per line — nothing else"
```

> **stdin cap:** piped stdin is capped at **10MB**; exceeding it errors with a non-zero exit.

## A.6 Git worktrees (canonical)

This is the authoritative reference for worktree isolation in Claude Code; other pages link here. A [git worktree](https://git-scm.com/docs/git-worktree) is a separate working directory with its own files and branch that shares the repo's history and remote with your main checkout. Running a session (or a subagent) in its own worktree means edits in one never touch files in another — so Claude can build a feature in one terminal while fixing a bug in a second, with no collisions. Worktrees isolate **file edits**; [subagents](./subagents.md) and [agent teams](https://code.claude.com/docs/en/agent-teams) coordinate the *work itself*. Everything here assumes a git repo (see [Non-git VCS](#non-git-vcs) for others).

### Where they live & how they're named

| Aspect | Default |
|---|---|
| **Location** | `<repo-root>/.claude/worktrees/<value>/`. Add `.claude/worktrees/` to `.gitignore` so worktree contents don't show as untracked in your main checkout. |
| **Branch name** | `worktree-<value>` on a **new** branch. |
| **Auto-generated name** | If you omit the name, Claude generates one like `bright-running-fox` (branch `worktree-bright-running-fox`). |
| **PR worktrees** | `--worktree "#1234"` (or a full GitHub PR URL) fetches `pull/<number>/head` from `origin` and checks out at `.claude/worktrees/pr-<number>`. |
| **Relocating** | To put worktrees elsewhere, configure a [`WorktreeCreate` hook](https://code.claude.com/docs/en/hooks#worktreecreate), which replaces the default `git worktree` logic entirely. |

### Starting a session in a worktree

```bash
# New isolated worktree at .claude/worktrees/feature-auth on branch worktree-feature-auth
claude --worktree feature-auth          # or -w

# A second isolated session in another terminal — never collides with the first
claude --worktree bugfix-123

# Auto-named worktree (e.g. bright-running-fox)
claude --worktree

# From a PR → .claude/worktrees/pr-1234
claude --worktree "#1234"

# Pair with a tmux session for the worktree
claude -w feature-auth --tmux           # --tmux=classic for traditional tmux
```

- **Trust gate:** before using `--worktree` interactively in a directory for the first time, run plain `claude` once there to accept the [workspace-trust](https://code.claude.com/docs/en/security) dialog; otherwise `--worktree` exits with an error. Non-interactive `claude -p --worktree` **skips** the trust check.
- **In-session:** you can ask Claude to "work in a worktree" mid-session and it creates one via the [`EnterWorktree`](https://code.claude.com/docs/en/tools-reference) tool; calling `EnterWorktree` again with a target path under `.claude/worktrees/` switches between them, leaving the previous one untouched on disk.
- **Desktop app:** creates a worktree for **every** new session automatically.

### Base branch (`worktree.baseRef`)

Worktrees branch from the repo's default branch (`origin/HEAD`) by default, so they start from a clean tree matching the remote. If no remote is configured or the fetch fails, they fall back to the local `HEAD`. To always branch from local `HEAD` (carrying your unpushed commits / feature-branch state — useful when isolating subagents that must operate on in-progress work), set [`worktree.baseRef`](https://code.claude.com/docs/en/settings) to `"head"`. The setting accepts **only** `"fresh"` (default) or `"head"`, not arbitrary git refs:

```json
{ "worktree": { "baseRef": "head" } }
```

### Copying gitignored files (`.worktreeinclude`)

A worktree is a fresh checkout, so untracked files like `.env` / `.env.local` are **not** present. Add a `.worktreeinclude` file (`.gitignore` syntax) to your project root to copy them in automatically. Only files that both match a pattern **and** are gitignored are copied, so tracked files are never duplicated. Applies to `--worktree`, subagent worktrees, and desktop parallel sessions:

```text
# .worktreeinclude
.env
.env.local
config/secrets.json
```

### Isolating subagents (`isolation: worktree`)

Subagents can each run in their own worktree so parallel edits don't conflict — this is the main reason to reach for worktrees: **parallel agents that mutate files.** Two ways:

- **Ad hoc:** ask Claude to "use worktrees for your agents".
- **Permanent:** add `isolation: worktree` to a [custom subagent's frontmatter](./subagents.md). Each subagent gets a **temporary** worktree, removed automatically when it finishes **without changes**.

Subagent worktrees use the same base branch as `--worktree` (default branch unless `worktree.baseRef: "head"`).

### Lifecycle & auto-cleanup

What happens on exit depends on whether the worktree changed:

| State on exit | What happens |
|---|---|
| **No** uncommitted changes, untracked files, or new commits | Worktree **and its branch are removed automatically**. If the session has a [name](https://code.claude.com/docs/en/sessions), Claude prompts instead so you can keep it. |
| Uncommitted changes, untracked files, **or** new commits exist | Claude prompts to keep or remove. Keeping preserves the directory + branch; removing deletes them, discarding the changes/commits. |
| **Non-interactive** (`-p --worktree`) | **Not** cleaned up automatically (no exit prompt). Remove with `git worktree remove`. |

Beyond per-session cleanup, a startup **sweep** removes worktrees Claude created for **subagents and [background sessions](https://code.claude.com/docs/en/agent-view)** once they're older than your [`cleanupPeriodDays`](./settings.md) setting (default 30) — but only if they have no uncommitted changes, untracked files, or unpushed commits. Worktrees you made with `--worktree` are **never** removed by this sweep. While an agent runs, Claude `git worktree lock`s its worktree so concurrent cleanup can't remove it; the lock releases when the agent finishes. To force-remove one the sweep keeps: `git worktree remove [--force] <path>`.

### Manual worktrees & non-git VCS {#non-git-vcs}

For full control over location/branch, manage worktrees with git directly, then `cd` in and run `claude`:

```bash
git worktree add ../project-feature-a -b feature-a   # new branch
git worktree add ../project-bugfix bugfix-123        # existing branch
git worktree list
git worktree remove ../project-feature-a
```

Remember to re-initialize your dev environment in each new worktree (install deps, venvs, etc.). For **SVN/Perforce/Mercurial/other** VCS, configure [`WorktreeCreate` and `WorktreeRemove` hooks](https://code.claude.com/docs/en/hooks#worktreecreate) for custom create/cleanup logic. Because the hook replaces the default git behavior, `.worktreeinclude` is **not** processed — copy local config files inside the hook script instead.

---

# Part B — Interactive keyboard shortcuts

> Shortcuts vary by platform and terminal. In [fullscreen rendering](https://code.claude.com/docs/en/fullscreen), press `?` in the transcript viewer for the live shortcut panel. On **macOS**, the `Alt+…` shortcuts (`Alt+B/F/Y/M/P`) require configuring **Option as Meta** (see [B.5](#b5-terminal-setup--notifications)).

## B.0 Chord & modifier reference (consolidated)

The single table to skim. Each row's **rebind ID** is the keybindings action you'd remap (see [B.7](#b7-custom-keybindings)); chords (two keystrokes pressed in sequence) are shown with a space. The remaining sections (B.1–B.5) repeat these with fuller context.

| Chord | Action | Rebind ID (context) |
|---|---|---|
| **Shift+Tab** (`Alt+M` / `Meta+M` in some configs) | Cycle permission modes: `default` → `acceptEdits` → `plan` → any enabled (`auto`, `bypassPermissions`). | `chat:cycleMode` (Chat) |
| **Ctrl+B** (or `Ctrl+X Ctrl+B`) | Background the running bash task/agent. **tmux:** press `Ctrl+B` twice (tmux prefix); the `Ctrl+X Ctrl+B` chord (v2.1.169+) avoids that conflict. | `task:background` (Task) |
| **Ctrl+O** | Toggle the transcript viewer (expands tool usage; unfolds collapsed MCP calls). | `app:toggleTranscript` (Global) |
| **Ctrl+R** | Reverse-search command history. | `history:search` (Global) |
| **Ctrl+S** | In history search: cycle scope (session → project → everywhere). In the prompt: **stash** the current prompt. | `historySearch:cycleScope` / `chat:stash` |
| **Ctrl+G** or **Ctrl+X Ctrl+E** | Open the prompt in `$EDITOR`. | `chat:externalEditor` (Chat) |
| **Ctrl+X Ctrl+K** | Stop all background subagents (press twice within 3 s). | `chat:killAgents` (Chat) |
| **Ctrl+T** | Toggle the task list (in `/theme`: toggle syntax highlighting). | `app:toggleTodos` / `theme:toggleSyntaxHighlighting` |
| **Ctrl+L** | Redraw screen (in fullscreen, press twice in 2 s to `/clear`). | `chat:clearInput` (Chat) |
| **Ctrl+J** | Insert a newline without submitting. | `chat:newline` (Chat) |
| **Esc** | Interrupt Claude mid-turn (work so far kept). | `chat:cancel` (Chat) |
| **Esc Esc** | Clear input + save draft; if input empty, open the rewind menu. | — |
| **Ctrl+C** / **Ctrl+D** | Interrupt-or-exit / EOF exit. **Reserved — cannot be rebound.** | `app:interrupt` / `app:exit` |
| **Alt+P / Option+P** | Switch model without clearing the prompt. | `chat:modelPicker` (Chat) |
| **Alt+T / Option+T** | Toggle extended thinking. | `chat:thinkingToggle` (Chat) |
| **Alt+O / Option+O** | Toggle fast mode. | `chat:fastMode` (Chat) |
| **Ctrl+E** (transcript) | Toggle show-all-content in the transcript viewer. | `transcript:toggleShowAll` (Transcript) |
| **q / Ctrl+C / Esc** (transcript) | Exit the transcript viewer. | `transcript:exit` (Transcript) |

> **`Ctrl+]` (reopen artifact/transcript):** WARN: verify. `Ctrl+]` is **not** in the official interactive-mode or keybindings tables, and no `transcript`/artifact action binds it by default. The documented way to (re)open the transcript viewer is **Ctrl+O** (`app:toggleTranscript`); inside fullscreen the viewer is reviewed with `[`, `{`/`}`, `v`, and `?`. Treat `Ctrl+]` as unconfirmed unless you have rebound it yourself in `keybindings.json`.

## B.1 General controls

| Shortcut | Action |
|---|---|
| **Enter** | Send the message. |
| **Ctrl+C** | Interrupt a running operation. If nothing is running: first press clears the prompt, a second press **exits** Claude Code. |
| **Ctrl+D** | Exit the session (EOF). |
| **Ctrl+L** | Redraw the screen (recover from a garbled display); input and history are kept. |
| **Ctrl+R** | Reverse-search command history (see [B.4](#b4-history--reverse-search)). |
| **Ctrl+O** | Toggle the transcript viewer (expands tool usage; e.g. unfolds a collapsed "Called slack 3 times"). |
| **Ctrl+G** or **Ctrl+X Ctrl+E** | Open the prompt in your `$EDITOR`. (`Ctrl+X Ctrl+E` is the readline-native binding.) |
| **Ctrl+X Ctrl+K** | Stop all background subagents in this session (press twice within 3 s to confirm). |
| **Ctrl+B** | Background the running bash task/agent. (tmux users press twice.) |
| **Ctrl+T** | Toggle the task list in the status area (shows up to 5). |
| **Esc** | Interrupt Claude mid-turn so you can redirect; work done so far is kept. |
| **Esc Esc** (double) | If the input has text: clear it and save the draft to history (`Up` recalls it). If the input is **empty**: open the [rewind menu](https://code.claude.com/docs/en/checkpointing) to restore/summarize code & conversation from an earlier point. |
| **Up/Down** or **Ctrl+P/Ctrl+N** | Move the cursor within a multi-row prompt; once on the first/last visual row, navigate command history. *(v2.1.169+: wrapped single-line input behaves like multiline.)* |
| **Left/Right** | Cycle tabs in permission dialogs and menus. |
| **Shift+Tab** or **Alt+M** | Cycle permission modes: `default` → `acceptEdits` → `plan` → any you've enabled (`auto`, `bypassPermissions`). See [permission modes](https://code.claude.com/docs/en/permission-modes). |
| **Option+P** / **Alt+P** | Switch model without clearing the prompt. |
| **Option+T** / **Alt+T** | Toggle [extended thinking](../models/capabilities-and-modes.md). No effect on Fable 5 (always thinks). *(v2.1.132+: works on macOS without Option-as-Meta.)* |
| **Option+O** / **Alt+O** | Toggle [fast mode](../models/capabilities-and-modes.md). |

> **Tab does *not* toggle extended thinking.** That is **Alt+T / Option+T**. `Tab` accepts a prompt suggestion or autocompletes a `!` shell command (see [B.3](#b3-prefixes---and-images)).

## B.2 Text editing (readline-style)

| Shortcut | Action |
|---|---|
| **Ctrl+A** / **Ctrl+E** | Move to start / end of the current logical line. |
| **Ctrl+K** | Delete to end of line (stored for paste). |
| **Ctrl+U** | Delete to line start (stored; repeat clears across multiline lines). On macOS, `Cmd+Backspace` maps here. |
| **Ctrl+W** | Delete the previous word (stored). On Windows, `Ctrl+Backspace` also does this. |
| **Ctrl+Y** | Paste the last text deleted with `Ctrl+K/U/W`. |
| **Alt+Y** (after `Ctrl+Y`) | Cycle through paste history. *(macOS: needs Option as Meta.)* |
| **Alt+B** / **Alt+F** | Move back / forward one word. *(macOS: needs Option as Meta.)* |

## B.3 Prefixes (`/`, `!`, `@`) and images

| Key | Action |
|---|---|
| **`/`** at start | Open the command/skill menu; type letters to filter. See [Slash commands](./slash-commands.md). |
| **`!`** at start | **Shell mode** — run a command directly, add its output to context. *(v2.1.186+: Claude then responds automatically; disable with `respondToBashCommands: false`.)* `Tab` autocompletes from previous `!` commands in the project. Exit with `Esc`, `Backspace`, or `Ctrl+U` on an empty prompt. |
| **`@`** | File-path mention/autocomplete — inline a file or directory by path. |
| **Tab** / **Right** | Accept the grayed-out **prompt suggestion**, then `Enter` to submit. (Suggestions are skipped after the first turn and in plan mode.) |
| **Paste image** — `Ctrl+V` (Win/Linux), `Cmd+V` (macOS/iTerm2), `Alt+V` (Windows/WSL) | Paste an image from the clipboard; inserts an `[Image #N]` chip you can reference positionally. On WSL both `Ctrl+V` and `Alt+V` are bound — use `Alt+V` if the terminal eats `Ctrl+V`. |
| **Drag-and-drop** an image file | Drop into the terminal to attach it. WARN: verify — drag-drop behavior depends on the terminal emulator; in some you must drag while holding a modifier or paste the path. |

> **`#` to add memory:** WARN: verify. The current [interactive-mode docs](https://code.claude.com/docs/en/interactive-mode) do **not** list `#` as a memory-add prefix. Add memory via `/memory`, by asking Claude ("remember that…"), or by editing `CLAUDE.md`. Don't rely on a `#` quick-add shortcut. (Separately, `#` is used to *comment out* the prepended previous-response block in the `Ctrl+G` external editor.)

### Multiline input

| Method | How | Where it works |
|---|---|---|
| Backslash escape | `\` then **Enter** | **All** terminals, no setup. |
| Control sequence | **Ctrl+J** | Any terminal, no setup. |
| Shift+Enter | **Shift+Enter** | Native in iTerm2, WezTerm, Ghostty, Kitty, Warp, Apple Terminal, Windows Terminal. For **VS Code, Cursor, Devin Desktop, Alacritty, Zed** run `/terminal-setup` first. Not in gnome-terminal / JetBrains IDEs — use `Ctrl+J` or `\`. |
| Option+Enter | **Option+Enter** | macOS, after enabling Option as Meta. |
| Paste mode | paste directly | For code blocks and logs. |

## B.4 History & reverse search

- Input history is stored **per working directory**; `/clear` resets it (the prior conversation is preserved and resumable). Submitting the same prompt twice records one entry.
- **`Ctrl+R`** opens reverse search: type to filter (matches highlighted), `Ctrl+R` again cycles older matches, **`Ctrl+S`** cycles scope (this session → this project → all projects), `Tab`/`Esc` accept-and-edit, `Enter` accept-and-run, `Ctrl+C` cancels. It loads the 100 most recent unique prompts in the chosen scope.
- Shell history expansion (`!`) is **disabled by default** so a leading `!` means shell mode, not history expansion.

## B.5 Terminal setup & notifications

**`/terminal-setup`** installs the `Shift+Enter` (and related) keybindings for terminals that need it — **VS Code, Cursor, Devin Desktop, Alacritty, Zed**. Run it in the **host** terminal, not inside tmux/screen. It also adjusts GPU acceleration / mouse-wheel settings in integrated-terminal configs.

**Option as Meta (macOS)** — required for the `Alt+…` shortcuts:

- **iTerm2:** Settings → Profiles → Keys → General → Left/Right Option key → "Esc+".
- **Apple Terminal:** Settings → Profiles → Keyboard → "Use Option as Meta Key".
- **VS Code:** set `"terminal.integrated.macOptionIsMeta": true`.

**tmux** — add to `~/.tmux.conf` (then `tmux source-file ~/.tmux.conf`) so `Shift+Enter` and notifications pass through:

```bash
set -g allow-passthrough on
set -s extended-keys on
set -as terminal-features 'xterm*:extkeys'
```

**Notifications** — desktop notifications work out of the box in Ghostty, Kitty, and iTerm2. Elsewhere, set `preferredNotifChannel` to `"terminal_bell"` in [settings](./settings.md), or wire a `Notification` hook (see [Hooks](./hooks.md)):

```json
{
  "hooks": {
    "Notification": [
      { "hooks": [{ "type": "command", "command": "afplay /System/Library/Sounds/Glass.aiff" }] }
    ]
  }
}
```

## B.6 Vim editor mode

Enable with **`/config` → Editor mode** (or set `editorMode: "vim"` in `~/.claude/settings.json`). The standalone `/vim` command was **removed in v2.1.92** — use `/config`. In INSERT mode, **Enter still submits** (unlike real vim); insert a newline with `o`/`O` in NORMAL mode or `Ctrl+J`.

### Mode switching (from NORMAL unless noted)

| Key | Action |
|---|---|
| `Esc` | Enter NORMAL mode (from INSERT/VISUAL). |
| `i` / `I` | Insert before cursor / at beginning of line. |
| `a` / `A` | Insert after cursor / at end of line. |
| `o` / `O` | Open a line below / above. |
| `v` / `V` | Start character-wise / line-wise VISUAL selection. |

### Navigation (NORMAL)

| Key | Action |
|---|---|
| `h` `j` `k` `l` | Left / down / up / right (arrows also work). |
| `Space` | Move right. |
| `w` / `e` / `b` | Next word / end of word / previous word. |
| `0` / `$` / `^` | Line start / line end / first non-blank. |
| `gg` / `G` | Beginning / end of input. |
| `f{c}` / `F{c}` | Jump to next / previous occurrence of char `c`. |
| `t{c}` / `T{c}` | Jump to just before next / after previous `c`. |
| `;` / `,` | Repeat last `f/F/t/T` forward / reverse. |
| `/` | Open reverse history search (same as `Ctrl+R`). *(v2.1.191+: an empty search hints `Esc`, `i`, `/` to open the command menu.)* |

> If the cursor is at the start/end of input and can't move further, `j`/`k` and the arrow keys navigate **command history** instead.

### Editing (NORMAL)

| Key | Action |
|---|---|
| `x` | Delete a character. |
| `dd` / `D` | Delete line / to end of line. |
| `dw` `de` `db` | Delete word / to end / back. |
| `cc` / `C` | Change line / to end of line. |
| `cw` `ce` `cb` | Change word / to end / back. |
| `yy` `Y` | Yank (copy) line. |
| `yw` `ye` `yb` | Yank word / to end / back. |
| `p` / `P` | Paste after / before cursor. |
| `>>` / `<<` | Indent / dedent line. |
| `J` | Join lines. |
| `u` / `.` | Undo / repeat last change. |

### Text objects (with `d`, `c`, `y`)

| Key | Action |
|---|---|
| `iw` / `aw` | Inner / around word. |
| `iW` / `aW` | Inner / around WORD (whitespace-delimited). |
| `i"` `a"` / `i'` `a'` | Inner / around double / single quotes. |
| `i(` `a(` / `i[` `a[` / `i{` `a{` | Inner / around parens / brackets / braces. |

### Visual mode

Press `v` (character-wise) or `V` (line-wise); motions extend the selection, operators act on it: `d`/`x` delete, `y` yank, `c`/`s` change, `p` replace with register, `r{c}` replace each selected char, `~`/`u`/`U` toggle/lower/upper case, `>`/`<` indent/dedent, `J` join, `o` swap cursor/anchor, `iw`/`aw`/`i"`/… select a text object, `v`/`V` toggle or exit. **Block-wise visual mode (`Ctrl+V`) is not supported.**

## B.7 Custom keybindings

Claude Code's shortcuts are remappable from **v2.1.18+** (check with `claude --version`). Run **`/keybindings`** to create or open the config file at **`~/.claude/keybindings.json`**. Changes are auto-detected and applied **without restarting** Claude Code. Validation runs on load — `/doctor` surfaces any warnings (parse errors, invalid context names, reserved/multiplexer conflicts, duplicate bindings).

### File schema

A top-level object with a `bindings` **array**; each block scopes a map of keystrokes → actions to one **context**.

| Field | Description |
|---|---|
| `$schema` | Optional JSON Schema URL for editor autocompletion (`https://www.schemastore.org/claude-code-keybindings.json`). |
| `$docs` | Optional docs URL. |
| `bindings` | Array of binding blocks, each `{ "context": …, "bindings": { "<keystroke>": "<action>" } }`. |

```json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "$docs": "https://code.claude.com/docs/en/keybindings",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+u": null
      }
    }
  ]
}
```

### Contexts & actions

Bindings are scoped to a **context** — where they apply. Documented contexts: `Global`, `Chat`, `Autocomplete`, `Settings`, `Confirmation`, `Tabs`, `Help`, `Transcript`, `HistorySearch`, `Task`, `ThemePicker`, `Attachments`, `Footer`, `MessageSelector`, `DiffDialog`, `ModelPicker`, `Select`, `Plugin`, `Scroll`, `Doctor`.

Actions use a `namespace:action` format. A representative slice (full list on the [keybindings reference](https://code.claude.com/docs/en/keybindings)):

| Action | Default | Context |
|---|---|---|
| `app:toggleTranscript` | Ctrl+O | Global |
| `app:toggleTodos` | Ctrl+T | Global |
| `app:redraw` | (unbound) | Global |
| `history:search` | Ctrl+R | Global |
| `chat:cycleMode` | Shift+Tab\* | Chat |
| `chat:submit` | Enter | Chat |
| `chat:newline` | Ctrl+J | Chat |
| `chat:externalEditor` | Ctrl+G, Ctrl+X Ctrl+E | Chat |
| `chat:stash` | Ctrl+S | Chat |
| `chat:killAgents` | Ctrl+X Ctrl+K | Chat |
| `task:background` | Ctrl+B, Ctrl+X Ctrl+B | Task |
| `transcript:toggleShowAll` | Ctrl+E | Transcript |
| `transcript:exit` | q, Ctrl+C, Escape | Transcript |
| `historySearch:cycleScope` | Ctrl+S | HistorySearch |

> \* `chat:cycleMode` defaults to **`Meta+M`** on Windows without VT mode (Node <24.2.0 / <22.17.0, Bun <1.2.23).

### Keystroke syntax

- **Modifiers** join with `+`: `ctrl`/`control`, `shift`, `alt`/`opt`/`option`/`meta` (Alt on Win/Linux, Option on macOS), `cmd`/`command`/`super`/`win`. The `cmd` group is only detected in terminals that report Super (Kitty protocol, xterm `modifyOtherKeys`) — most terminals don't send it, so prefer `ctrl`/`meta` for portable bindings.
- **Uppercase letters imply Shift** (`K` ≡ `shift+k`) — but uppercase **with a modifier** does not (`ctrl+K` ≡ `ctrl+k`).
- **Chords** are space-separated sequences: `"ctrl+k ctrl+s"` = press `Ctrl+K`, release, then `Ctrl+S`.
- **Special keys:** `escape`/`esc`, `enter`/`return`, `tab`, `space`, `up`/`down`/`left`/`right`, `backspace`, `delete`.

### Unbinding & reserved keys

Set an action to **`null`** to unbind a default (works for chords too). Unbinding *every* chord sharing a prefix frees that prefix for a single-key binding; unbinding *some* leaves the prefix in chord-wait mode:

```json
{ "bindings": [ { "context": "Chat", "bindings": {
  "ctrl+x ctrl+k": null,
  "ctrl+x ctrl+e": null,
  "ctrl+x": "chat:newline"
} } ] }
```

**Reserved (cannot be rebound):** `Ctrl+C` (interrupt), `Ctrl+D` (exit), `Ctrl+M` (= Enter in terminals), Caps Lock (not delivered). **Multiplexer conflicts:** `Ctrl+B` (tmux prefix — press twice), `Ctrl+A` (GNU screen prefix), `Ctrl+Z` (Unix SIGTSTP).

> **Vim mode + keybindings are independent layers.** Vim handles text-input editing (cursor/modes/motions); keybindings handle component-level actions (toggle todos, submit, …). `Esc` in vim switches INSERT→NORMAL and does **not** fire `chat:cancel`; most `Ctrl+`key shortcuts pass through vim to the keybinding system. See [B.6](#b6-vim-editor-mode).

---

## Related pages

- [Slash commands](./slash-commands.md) — in-session `/` commands; `/config`, `/terminal-setup`, `/keybindings`, `/vim` status, `/statusline`.
- [Settings](./settings.md) — `settings.json` keys referenced here (`respondToBashCommands`, `preferredNotifChannel`, `viewMode`, `editorMode`, `model`, output styles, CLAUDE.md).
- [Hooks](./hooks.md) — `Notification`/`Setup`/`SessionStart` hooks invoked by `--init`, `--maintenance`, `--init-only`, and notification config.
- [Subagents](./subagents.md) — `--agents`, `--agent`, background sessions, agent view.
- [Dispatch / Remote / Routines](./dispatch-remote-routines.md) — `--bg`, `claude agents`, `--remote`, `--remote-control`, `--teleport`.
- [Claude Code overview](../platform/claude-code.md) — the product surface this CLI drives.
- [Capabilities & modes](../models/capabilities-and-modes.md) — effort levels, extended thinking, fast mode behind `--effort`, `Alt+T`, `Alt+O`.
- [MCP](../capabilities/mcp.md) — `claude mcp`, `--mcp-config`, `--strict-mcp-config`.

## Open questions / to verify

- **`claude config` / `claude doctor` top-level subcommands:** not in the current official CLI-commands table; settings/diagnostics are surfaced via `/config` and `/doctor`. Re-verify whether bare `claude config`/`claude doctor` exist in any build.
- **`--effort ultracode` on the CLI:** the documented `--effort` table lists `low…max`; `ultracode` appears only for the `/effort` slash command. Confirm whether the flag accepts it.
- **`--output-format json` field names:** the exact JSON shape (`result`, `session_id`, `usage`, `total_cost_usd`, `model`, `stop_reason`, `structured_output`) should be re-checked against the Agent SDK reference for the target version.
- **Image drag-and-drop:** terminal-dependent; paste (`Ctrl/Cmd/Alt+V`) is the documented, reliable path. Confirm drag-drop per emulator.
- **`#` memory prefix:** not documented in interactive-mode shortcuts. Treat as unsupported; use `/memory` or natural-language requests.
- **`Ctrl+]` (reopen artifact/transcript):** not in the official interactive-mode or keybindings tables, and unbound by default. Use `Ctrl+O` (`app:toggleTranscript`) to (re)open the transcript. Re-verify whether any build binds `Ctrl+]`.
- **Worktree sweep vs. `--worktree`:** the startup `cleanupPeriodDays` sweep removes only **subagent/background-session** worktrees with no pending changes; `--worktree`-created ones are never swept. Re-verify this asymmetry against the worktrees page per release.
- **`chat:cycleMode` default key:** `Shift+Tab` normally, but `Meta+M` on Windows without VT mode (older Node/Bun). Re-verify the version thresholds against the keybindings page for your install.
- **Version gates** (e.g. `mcp login/logout` v2.1.186+, `--safe-mode` v2.1.169+, `--ax-screen-reader` v2.1.181+, `--advisor` v2.1.98+, `--enable-auto-mode` removed v2.1.111) are point-in-time — re-verify against `/release-notes` for the install you target.

## Sources

- [CLI reference — code.claude.com/docs/en/cli-reference](https://code.claude.com/docs/en/cli-reference) (subcommands + full flag table, verbatim)
- [Interactive mode — code.claude.com/docs/en/interactive-mode](https://code.claude.com/docs/en/interactive-mode) (keyboard shortcuts, multiline, shell mode, vim mode, verbatim)
- [Terminal configuration — code.claude.com/docs/en/terminal-config](https://code.claude.com/docs/en/terminal-config) (Shift+Enter, Option-as-Meta, tmux, notifications)
- [Headless mode — code.claude.com/docs/en/headless](https://code.claude.com/docs/en/headless) (`-p`, output formats, bare mode, scripting)
- [Permission modes — code.claude.com/docs/en/permission-modes](https://code.claude.com/docs/en/permission-modes) (`--permission-mode`, auto/bypass)
- [Agent view — code.claude.com/docs/en/agent-view](https://code.claude.com/docs/en/agent-view) (`claude agents`, background-session commands)
- [Checkpointing — code.claude.com/docs/en/checkpointing](https://code.claude.com/docs/en/checkpointing) (double-Esc rewind menu)
- [Worktrees — code.claude.com/docs/en/worktrees](https://code.claude.com/docs/en/worktrees) (`--worktree`, `.claude/worktrees/`, branch naming, `isolation: worktree`, lifecycle/cleanup, `.worktreeinclude`, `worktree.baseRef`)
- [Keybindings — code.claude.com/docs/en/keybindings](https://code.claude.com/docs/en/keybindings) (`~/.claude/keybindings.json` schema, `/keybindings`, contexts, action IDs, chord syntax, reserved keys)

*Verified 2026-06-26 against the live `code.claude.com` docs. Part A's flag/subcommand tables and Part B's shortcut/vim tables were cross-checked against the official CLI-reference and interactive-mode pages (both fetched verbatim); items not confirmable against those pages are marked **WARN: verify** inline and listed under Open questions.*
