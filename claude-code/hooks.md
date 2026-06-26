---
type: Configuration Reference
title: Claude Code — Hooks
description: Complete reference to Claude Code hooks — every lifecycle event, matcher syntax, the settings.json config shape, per-event stdin input, exit codes, and structured JSON output.
domain: claude-code
tags: [hooks, claude-code, settings, automation, lifecycle, PreToolUse, PostToolUse, guardrails, mcp]
related: [settings, slash-commands, subagents, plugins, mcp, cli-shortcuts]
resource: https://code.claude.com/docs/en/hooks
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://code.claude.com/docs/en/hooks
  - https://code.claude.com/docs/en/hooks-guide
  - https://code.claude.com/docs/en/settings
  - https://code.claude.com/docs/en/permissions
  - https://code.claude.com/docs/en/env-vars
  - https://code.claude.com/docs/en/security-guidance
---

# Claude Code — Hooks

Hooks are user-defined shell commands (or HTTP/MCP/prompt/agent handlers) that Claude Code runs automatically at specific points in its lifecycle. They give you **deterministic** control over the agent — formatting code after every edit, blocking dangerous Bash before it runs, injecting context at session start — instead of hoping the model chooses to do those things. Because hooks execute with your full user credentials, they are powerful and must be treated as security-sensitive code.

> at-a-glance
> - **What it is** — A configuration block (`"hooks"`) in `settings.json` that maps lifecycle *events* to shell commands the harness executes. Hooks communicate over stdin (JSON in) / stdout / stderr / exit codes (decisions out).
> - **Where you find it** — `~/.claude/settings.json` (global), `.claude/settings.json` (project, shareable), `.claude/settings.local.json` (project, gitignored), managed/policy settings (org), plugin `hooks/hooks.json`, and skill/agent frontmatter. Browse with the `/hooks` slash command.
> - **Who can use it** — Anyone running Claude Code (CLI, IDE, desktop, web). Same feature across all plans; managed/policy hooks are admin-controlled for Team/Enterprise.
> - **Status** — Stable and actively expanding. The event catalog grew substantially through 2026 (now 30 events, verified against the current reference). `agent`-type hooks are explicitly experimental and may change. WARN: older installs expose fewer events — verify your installed version with `/hooks`.

---

## How hooks work

When a lifecycle event fires, Claude Code finds every configured hook whose **matcher** matches, then runs all matching hooks **in parallel**. Identical hook commands are automatically deduplicated. Each hook receives event-specific JSON on **stdin**, does its work, and signals back through its **exit code** and/or a **JSON object printed to stdout**.

The core loop:

1. Event fires (e.g. `PreToolUse` before Claude runs `Bash`).
2. Claude Code serializes event data to JSON and pipes it to each matching hook's stdin.
3. Hook runs. For shell-form `command` hooks (no `args`), the default shell is `sh -c` on macOS/Linux and Git Bash on Windows; set `"shell": "powershell"` or switch to exec form with `args` to override.
4. Hook exits: `0` = no objection, `2` = block (stderr fed to Claude), other = non-blocking error.
5. Optionally the hook prints a JSON object on stdout for structured control (`permissionDecision`, `decision`, `additionalContext`, `continue`, etc.).
6. Claude Code merges results from all hooks (most-restrictive-wins for permission decisions) and proceeds.

Hooks are **hot-reloaded**: if you edit a settings file while Claude Code is running, the file watcher normally picks up hook changes automatically within a few seconds. If a change doesn't appear (the watcher missed it), restart the session to force a reload. (Earlier write-ups described hooks as snapshotted at startup; the current docs state edits are picked up automatically.)

---

## Configuration shape (settings.json)

The `hooks` object is keyed by **event name**. Each event maps to an array of **matcher groups**. Each group has an optional `matcher` plus a `hooks` array of **handler objects**.

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-or-script",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

Concrete two-group example (format on edit + notify on attention):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write" }
        ]
      }
    ],
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "notify-send 'Claude Code' 'needs your attention'" }
        ]
      }
    ]
  }
}
```

> Note: if your settings file already has a `hooks` key, add the new event as a **sibling** key inside the single `hooks` object — do not replace the whole object.

### Handler object fields (`type: "command"`)

| Field | Meaning |
| :--- | :--- |
| `type` | `"command"` (default), `"http"`, `"mcp_tool"`, `"prompt"`, or `"agent"`. |
| `command` | Executable or shell string. Shell form (no `args`) runs via a shell; exec form (with `args`) spawns directly. |
| `args` | Argument list. Presence switches to **exec form** (no shell, no quoting headaches). |
| `timeout` | Seconds before the hook is killed. Defaults: `command`/`http`/`mcp_tool` = 600 (10 min); `prompt` = 30; `agent` = 60. `UserPromptSubmit` lowers `command`/`http`/`mcp_tool` to 30s and `MessageDisplay` lowers them to 10s. |
| `shell` | `"bash"` (default) or `"powershell"`. Ignored when `args` is set (exec form runs no shell). |
| `if` | Permission-rule filter on tool name **and arguments** (e.g. `"Bash(git *)"`, `"Edit(*.ts)"`). Tool events only. Requires v2.1.85+; earlier versions ignore it and run on every matched call. |
| `statusMessage` | Custom spinner text shown in the UI while the hook runs (e.g. `"Validating command..."`). |
| `once` | If `true`, the hook runs once per session then is removed. Skill/agent frontmatter only. |
| `async` / `asyncRewake` | Run in background without blocking; `asyncRewake` wakes Claude on exit code 2 (implies `async`). Command hooks only. |
| `model` | For `prompt`/`agent` hooks: the model to use. Defaults to a fast model (Haiku). |

Top-level toggle: `"disableAllHooks": true` turns every hook off (managed-settings hooks still run unless also disabled there).

### Path placeholders

| Placeholder | Resolves to |
| :--- | :--- |
| `${CLAUDE_PROJECT_DIR}` | Project root. Use it to reference `.claude/hooks/*.sh` reliably. |
| `${CLAUDE_PLUGIN_ROOT}` | A plugin's install directory (in plugin hooks). |
| `${CLAUDE_PLUGIN_DATA}` | A plugin's persistent data directory. |

---

## Hook locations and scope

| Location | Scope | Shareable |
| :--- | :--- | :--- |
| `~/.claude/settings.json` | All your projects | No (local to your machine) |
| `.claude/settings.json` | Single project | Yes (commit to repo) |
| `.claude/settings.local.json` | Single project | No (gitignored) |
| Managed / policy settings | Organization-wide | Yes (admin-controlled) |
| Plugin `hooks/hooks.json` | While plugin enabled | Yes (bundled with plugin) |
| Skill / agent frontmatter | While component active | Yes (in the component file) |

See **[./settings.md](./settings.md)** for the full settings precedence model and **[../capabilities/plugins.md](../capabilities/plugins.md)** for bundling hooks in plugins.

---

## The full event catalog

Events fire at three cadences: **once per session**, **once per turn**, and **per tool call** in the agentic loop. The catalog below reflects current docs (2026); newer events are version-gated, so confirm availability with `/hooks`.

| Event | When it fires | Can block (exit 2)? |
| :--- | :--- | :--- |
| `SessionStart` | A session begins or resumes | No (stdout = context) |
| `Setup` | Launched with `--init-only`, or `--init`/`--maintenance` in `-p` mode | No (stderr → user only) |
| `SessionEnd` | A session terminates | No |
| `UserPromptSubmit` | You submit a prompt, before Claude processes it | **Yes** (blocks prompt) |
| `UserPromptExpansion` | A typed command expands into a prompt, before Claude sees it | **Yes** (blocks expansion) |
| `PreToolUse` | Before a tool call executes | **Yes** (deny/allow/ask) |
| `PermissionRequest` | A permission dialog is about to appear | **Yes** (does not fire in `-p`) |
| `PermissionDenied` | A tool call denied by the auto-mode classifier; return `{retry:true}` | No |
| `PostToolUse` | After a tool call succeeds | No (observability) |
| `PostToolUseFailure` | After a tool call fails | No |
| `PostToolBatch` | After a batch of parallel tool calls resolves, before the next model call | **Yes** (stops the loop) |
| `Notification` | Claude Code emits a notification | No |
| `MessageDisplay` | While assistant message text is displayed (display-only) | No |
| `SubagentStart` | A subagent is spawned | No |
| `SubagentStop` | A subagent finishes | **Yes** (keeps subagent working) |
| `TaskCreated` | A task is created via `TaskCreate` | **Yes** (rolls back creation) |
| `TaskCompleted` | A task is marked completed | **Yes** (prevents completion) |
| `Stop` | Claude finishes responding (every turn, not just task end) | **Yes** |
| `StopFailure` | Turn ends due to an API error (output/exit code ignored) | No |
| `TeammateIdle` | An agent-team teammate is about to go idle | **Yes** (teammate keeps working) |
| `InstructionsLoaded` | `CLAUDE.md` / `.claude/rules/*.md` loaded into context | No |
| `ConfigChange` | A config file changes during the session | **Yes** (blocks the change) |
| `CwdChanged` | Working directory changes (e.g. Claude runs `cd`) | No |
| `FileChanged` | A watched file changes on disk (matcher = filenames) | No |
| `WorktreeCreate` | A worktree is being created (`--worktree` / `isolation: "worktree"`) | **Yes** (non-zero fails creation) |
| `WorktreeRemove` | A worktree is being removed | No |
| `PreCompact` | Before context compaction | **Yes** (blocks compaction) |
| `PostCompact` | After compaction completes | No |
| `Elicitation` | An MCP server requests user input during a tool call | **Yes** (denies elicitation) |
| `ElicitationResult` | After the user responds to an MCP elicitation | **Yes** |

> WARN: the exact set is version-dependent. Older docs and write-ups list a smaller "classic" set (PreToolUse, PostToolUse, UserPromptSubmit, Stop, SubagentStop, Notification, SessionStart, SessionEnd, PreCompact). Always verify with `/hooks` on your install.

---

## Matchers

A **matcher** scopes a hook group to a subset of occurrences. The empty string `""`, `"*"`, or an omitted matcher fires on **everything**.

How a matcher string is evaluated:

| Pattern characters | Interpreted as | Example |
| :--- | :--- | :--- |
| Letters, digits, `_`, spaces, `,`, `\|` only | Exact name or `\|`-separated list | `Bash`, `Edit\|Write` |
| Anything else | JavaScript regex | `^Notebook`, `mcp__memory__.*` |

> On v2.1.191+ the list separators `\|` and `,` are interchangeable for tool-name matchers, so `Edit|Write` and `Edit,Write` are equivalent. Matchers are **case-sensitive**.

### What each event's matcher filters

| Event(s) | Matcher filters on | Example values |
| :--- | :--- | :--- |
| `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied` | tool name | `Bash`, `Edit\|Write`, `mcp__github__.*` |
| `SessionStart` | how it started | `startup`, `resume`, `clear`, `compact` |
| `Setup` | which flag triggered it | `init`, `maintenance` |
| `SessionEnd` | why it ended | `clear`, `resume`, `logout`, `prompt_input_exit`, `bypass_permissions_disabled`, `other` |
| `Notification` | notification type | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog`, `elicitation_complete`, `elicitation_response` |
| `SubagentStart`, `SubagentStop` | agent type | `general-purpose`, `Explore`, `Plan`, custom names |
| `PreCompact`, `PostCompact` | trigger | `manual`, `auto` |
| `ConfigChange` | config source | `user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills` |
| `StopFailure` | error type | `rate_limit`, `overloaded`, `authentication_failed`, `oauth_org_not_allowed`, `billing_error`, `invalid_request`, `model_not_found`, `server_error`, `max_output_tokens`, `unknown` |
| `InstructionsLoaded` | load reason | `session_start`, `nested_traversal`, `path_glob_match`, `include`, `compact` |
| `FileChanged` | literal filenames (split on `\|`, not regex) | `.envrc\|.env` |
| `Elicitation`, `ElicitationResult`, `UserPromptExpansion` | MCP server / command name | your configured names |
| `UserPromptSubmit`, `PostToolBatch`, `Stop`, `CwdChanged`, `MessageDisplay`, `WorktreeCreate`, `WorktreeRemove`, `TaskCreated`, `TaskCompleted`, `TeammateIdle` | **no matcher support** | always fires |

### MCP tools

MCP tools are named `mcp__<server>__<tool>`, so match them with regex:

```json
{ "matcher": "mcp__memory__.*",   "hooks": [ /* all memory-server tools */ ] }
{ "matcher": "mcp__.*__write.*",  "hooks": [ /* any write across servers */ ] }
{ "matcher": "mcp__github__search_repositories", "hooks": [ /* one tool */ ] }
```

### `if` — filter by tool name AND arguments

`matcher` only filters by tool name at the group level. The `if` field (v2.1.85+) applies **permission-rule syntax** so the hook process spawns only when the tool *arguments* also match — useful to avoid launching a script for every Bash call.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "if": "Bash(git *)", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check-git-policy.sh" }
        ]
      }
    ]
  }
}
```

`if` only works on tool events (`PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied`); it is **best-effort and fails open** (runs the hook anyway when a command can't be parsed). Use the permission system, not `if`, for hard allow/deny.

| `if` pattern | Bash command | Hook runs? | Why |
| :--- | :--- | :--- | :--- |
| `Bash(git *)` | `git push` | yes | command name matches |
| `Bash(git *)` | `FOO=bar git push` | yes | leading `VAR=value` assignments are stripped, then `git push` matches |
| `Bash(git *)` | `npm test && git push` | yes | each subcommand checked |
| `Bash(git *)` | `echo $(git log)` | yes | commands inside `$()`/backticks checked |
| `Bash(git *)` | `echo $(date)` | no | no subcommand matches |
| `Bash(git push *)` | `echo $(date)` | yes | a pattern more specific than the command name fails open on `$()`/backticks/`$VAR` |

To match multiple tool names, use **separate handlers each with its own `if`**, or move alternation to the `matcher` level (which supports `|`). The `if` filter does not accept `|`-lists.

---

## Hook input (stdin JSON)

Every event passes a JSON object on stdin. **Common fields** present on all events:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "cwd": "/Users/sarah/myproject",
  "hook_event_name": "PreToolUse",
  "permission_mode": "default"
}
```

`permission_mode` is one of `default`, `plan`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`. Every event also carries `effort: { "level": "low|medium|high|xhigh|max" }`; subagent/agent contexts additionally carry `agent_id` and `agent_type`. A `model` field appears only on `SessionStart` and is not guaranteed to be present. Some env vars are also exported to the hook process: `$CLAUDE_EFFORT`, and `$CLAUDE_ENV_FILE` (for `SessionStart`, `Setup`, `CwdChanged`, `FileChanged`).

### Per-event input highlights

| Event | Key event-specific input fields |
| :--- | :--- |
| `PreToolUse` | `tool_name`, `tool_input` (e.g. `{ "command": "npm test" }`), `tool_use_id` |
| `PostToolUse` | `tool_name`, `tool_input`, `tool_response` / `tool_output` |
| `PostToolUseFailure` | `tool_name`, `tool_input`, error details |
| `UserPromptSubmit` | `prompt` (the submitted text), `permission_mode` |
| `SessionStart` | `source` (`startup`/`resume`/`clear`/`compact`), `model` |
| `SessionEnd` | end `reason` (matches the matcher values) |
| `Notification` | notification `message` and type |
| `Stop` / `SubagentStop` | `stop_hook_active` (true if already triggered a continuation) |
| `PreCompact` / `PostCompact` | compaction `trigger` (`manual`/`auto`) |
| `ConfigChange` | `source`, `file_path` |
| `FileChanged` / `CwdChanged` | changed path(s) |

A real `PreToolUse` stdin payload:

```json
{
  "session_id": "abc123",
  "cwd": "/Users/sarah/myproject",
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": { "command": "npm test" }
}
```

Parse it in a script with `jq`:

```bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')
```

---

## Hook output: exit codes

The exit code is the simplest control channel.

| Exit code | Meaning | JSON parsed? | Behavior |
| :--- | :--- | :--- | :--- |
| `0` | Success / no objection | Yes | The action proceeds; for `UserPromptSubmit`, `UserPromptExpansion`, and `SessionStart`, **stdout is injected into Claude's context**. |
| `2` | Blocking error | No | Action blocked; **stderr is fed back to Claude** as feedback so it can adjust. |
| other | Non-blocking error | No | Action proceeds; transcript shows `<hook name> hook error` + first stderr line; full stderr → debug log. |

A `PreToolUse` exit `0` does **not** approve the tool — the normal permission flow still applies. Exit `2` semantics vary by event:

**Blockable** (exit 2 stops/denies the action): `PreToolUse`, `PermissionRequest`, `UserPromptSubmit`, `UserPromptExpansion`, `Stop`, `SubagentStop`, `TeammateIdle`, `TaskCreated`, `TaskCompleted`, `ConfigChange`, `PreCompact`, `PostToolBatch`, `Elicitation`, `ElicitationResult`, `WorktreeCreate`.

**Non-blockable** (exit 2 only surfaces stderr, then continues): `PostToolUse`, `PostToolUseFailure`, `PermissionDenied`, `Notification`, `SubagentStart`, `SessionStart`, `Setup`, `SessionEnd`, `CwdChanged`, `FileChanged`, `WorktreeRemove`, `PostCompact`, `InstructionsLoaded`, `StopFailure`, `MessageDisplay`.

| Event(s) | Effect of exit 2 |
| :--- | :--- |
| `PreToolUse`, `UserPromptSubmit`, `UserPromptExpansion`, `PermissionRequest` | Blocks the action; stderr → Claude (or user) |
| `Stop`, `SubagentStop`, `PostToolBatch`, `TeammateIdle` | Prevents stopping / going idle — keeps work going |
| `TaskCreated` | Rolls back the task creation |
| `TaskCompleted` | Prevents the task from being marked completed |
| `ConfigChange` | Blocks the config change from taking effect |
| `PreCompact` | Blocks compaction |
| `Elicitation`, `ElicitationResult` | Denies the MCP elicitation |
| `WorktreeCreate` | Any non-zero fails worktree creation |
| `PostToolUse`, `PostToolUseFailure`, `PermissionDenied` | Cannot block; shows stderr to Claude |
| `SessionStart`, `Setup`, `SubagentStart`, `SessionEnd`, `Notification`, `StopFailure` | Cannot block; shows stderr to the user, execution continues |

> Do not mix channels: if you exit `2`, Claude Code ignores any JSON you printed. Use exit-2-with-stderr **or** exit-0-with-JSON, not both.

---

## Hook output: structured JSON

For richer control, exit `0` and print a JSON object to stdout.

### Universal (top-level) fields

```json
{
  "continue": false,
  "stopReason": "Build failed, fix errors before continuing",
  "suppressOutput": false,
  "systemMessage": "Warning: this action is deprecated",
  "terminalSequence": "]777;notify;Title;Message"
}
```

| Field | Effect |
| :--- | :--- |
| `continue` | If `false`, stops all further processing (overrides event-specific decisions). |
| `stopReason` | Message shown when `continue:false`. |
| `suppressOutput` | If `true`, hides the hook's stdout from the transcript. |
| `systemMessage` | Warning text surfaced to the user. |
| `terminalSequence` | Emit one or more allowlisted terminal escape sequences (used for desktop/taskbar notifications). |

**`terminalSequence` allowlist** (everything else — CSI cursor/color, OSC 8 hyperlinks, OSC 52 clipboard, OSC 1337 — is rejected and the field ignored):

- OSC `0`, `1`, `2` — window and icon titles.
- OSC `9` — iTerm2, ConEmu, Windows Terminal, WezTerm notifications, including `9;4` taskbar progress.
- OSC `99` — Kitty notifications.
- OSC `777` — urxvt, Ghostty, Warp notifications.
- Bare `BEL`.

> Output strings (`additionalContext`, `systemMessage`, plain stdout) are capped at **10,000 characters**; oversized output is saved to a file with a preview and the file path provided in its place.

### Decision control by event

Two families of events use a **top-level `decision: "block"` + `reason`**: `UserPromptSubmit`, `UserPromptExpansion`, `PostToolUse`, `PostToolUseFailure`, `PostToolBatch`, `Stop`, `SubagentStop`, `ConfigChange`, and `PreCompact`. The rest use `hookSpecificOutput`.

| Event | Decision mechanism |
| :--- | :--- |
| `PreToolUse` | `hookSpecificOutput.permissionDecision` = `allow` / `deny` / `ask` / `defer` (+ `permissionDecisionReason`, optional `updatedInput`, `additionalContext`) |
| `PostToolUse` / `PostToolUseFailure` / `PostToolBatch` | top-level `decision: "block"` + `reason`; or `hookSpecificOutput.additionalContext` / `updatedToolOutput` |
| `UserPromptSubmit` / `UserPromptExpansion` | top-level `decision: "block"` + `reason`; or `hookSpecificOutput.additionalContext` to inject text |
| `Stop` / `SubagentStop` | top-level `decision: "block"` + `reason` (keeps Claude/subagent working); `hookSpecificOutput.additionalContext` |
| `ConfigChange` / `PreCompact` | top-level `decision: "block"` + `reason` |
| `PermissionRequest` | `hookSpecificOutput.decision.behavior` = `allow` / `deny` (+ optional `updatedInput`, `updatedPermissions`) |
| `PermissionDenied` | `hookSpecificOutput.retry: true` to let the model retry |
| `SessionStart` | `hookSpecificOutput.additionalContext`, plus `sessionTitle`, `watchPaths`, `reloadSkills`, `initialUserMessage` |
| `Setup` / `SubagentStart` | `hookSpecificOutput.additionalContext` |
| `MessageDisplay` | `hookSpecificOutput.displayContent` (rewrite what is shown) |
| `Elicitation` / `ElicitationResult` | `hookSpecificOutput.action` = `accept` / `decline` / `cancel` (+ `content` to override form values) |
| `WorktreeCreate` | `hookSpecificOutput.worktreePath` (or path on stdout) |

`PreToolUse` `permissionDecision` values:

- `"allow"` — skip the interactive prompt. **Deny/ask rules (including managed deny lists) still apply.**
- `"deny"` — cancel the call; `permissionDecisionReason` goes to Claude.
- `"ask"` — show the normal permission prompt.
- `"defer"` — only in non-interactive `-p` mode; preserves the call so an Agent SDK wrapper can resume.

> Hooks can **tighten** but not **loosen**: a hook returning `allow` never overrides a settings/managed deny rule. Conversely a `PreToolUse` `deny` fires **before** any permission-mode check, so it blocks even in `bypassPermissions` / `--dangerously-skip-permissions`.

### Combining multiple hooks

When several hooks match one event, **all run to completion** and the results merge. For `PreToolUse` permission decisions the **most restrictive wins**, in order `deny` > `defer` > `ask` > `allow`. `additionalContext` from every hook is concatenated and passed to Claude. One hook's `deny` does **not** stop sibling hooks from running — never rely on a `deny` to suppress another hook's side effects.

> Edge case: if multiple `PreToolUse` hooks return `updatedInput` to rewrite a tool's arguments, the **last hook to finish wins**. Because hooks run in parallel the finish order is non-deterministic, so avoid having more than one hook rewrite the same tool's input.

---

## Worked examples

### 1. Auto-format on `PostToolUse(Edit|Write)`

Run Prettier on every file Claude edits. The matcher restricts it to file-editing tools; `jq` extracts the path.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write" }
        ]
      }
    ]
  }
}
```

### 2. Block dangerous Bash on `PreToolUse`

Deny destructive commands with a feedback message. Two equivalent styles — exit code, or structured JSON.

Exit-code style (`.claude/hooks/block-rm-rf.sh`):

```bash
#!/bin/bash
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  echo "Blocked: 'rm -rf' is not allowed by project policy" >&2  # stderr → Claude
  exit 2                                                          # exit 2 = block
fi
exit 0
```

Structured-JSON style (deny with a precise reason):

```bash
#!/bin/bash
COMMAND=$(jq -r '.tool_input.command')
if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "Destructive command blocked by hook"
    }
  }'
else
  exit 0
fi
```

Register it:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-rm-rf.sh" }
        ]
      }
    ]
  }
}
```

Two hooks on the same matcher run **in parallel**; one can log while another blocks. Here the logger always records the command (exit 0) and the guardrail denies `rm -rf` (exit 2) — the deny wins, but the log entry is still written:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "jq -r .tool_input.command >> ~/.claude/bash.log" },
          { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-rm-rf.sh" }
        ]
      }
    ]
  }
}
```

### 3. Block edits to protected files

```bash
#!/bin/bash
# .claude/hooks/protect-files.sh
INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')
PROTECTED_PATTERNS=(".env" "package-lock.json" ".git/")
for pattern in "${PROTECTED_PATTERNS[@]}"; do
  if [[ "$FILE_PATH" == *"$pattern"* ]]; then
    echo "Blocked: $FILE_PATH matches protected pattern '$pattern'" >&2
    exit 2
  fi
done
exit 0
```

```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Edit|Write",
        "hooks": [ { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-files.sh" } ] }
    ]
  }
}
```

Make scripts executable on macOS/Linux: `chmod +x .claude/hooks/protect-files.sh`.

### 4. Inject context on `UserPromptSubmit`

Anything written to stdout (exit 0) is appended to Claude's context for that prompt.

```json
{
  "hooks": {
    "UserPromptSubmit": [
      { "hooks": [ { "type": "command", "command": "echo \"Active branch: $(git rev-parse --abbrev-ref HEAD)\"" } ] }
    ]
  }
}
```

Or block a prompt that fails a policy and explain why:

```bash
#!/bin/bash
PROMPT=$(jq -r '.prompt')
if ! echo "$PROMPT" | grep -qi 'acceptance criteria'; then
  echo '{"decision":"block","reason":"Prompts must include acceptance criteria."}'
fi
exit 0
```

### 5. Desktop notification on `Notification`

Fire a native notification whenever Claude needs you. Cross-platform variants:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command",
            "command": "osascript -e 'display notification \"Claude Code needs your attention\" with title \"Claude Code\"'" }
        ]
      }
    ]
  }
}
```

- Linux: `notify-send 'Claude Code' 'Claude Code needs your attention'`
- Windows: `powershell.exe -Command "[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms'); [System.Windows.Forms.MessageBox]::Show('Claude Code needs your attention', 'Claude Code')"`

Scope to specific notification types via the matcher: `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog`, etc.

### 6. Guardrails on `Stop` (keep working until done)

`Stop` hooks fire whenever Claude finishes a turn. Returning `decision: "block"` makes Claude keep working. **Always guard against the 8-iteration block cap** by checking `stop_hook_active`:

```bash
#!/bin/bash
INPUT=$(cat)
if [ "$(echo "$INPUT" | jq -r '.stop_hook_active')" = "true" ]; then
  exit 0   # already continued once — let Claude stop
fi
if ! npm test >/dev/null 2>&1; then
  echo '{"decision":"block","reason":"Tests are failing. Fix them before stopping."}'
fi
exit 0
```

Claude Code overrides a `Stop` hook after it blocks 8 times consecutively without progress. Raise the limit with the `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` env var if your loop legitimately needs more iterations. Note that `Stop` fires on every finished turn but **not** on user interrupts; API errors fire `StopFailure` (whose output and exit code are ignored) instead.

### 7. Re-inject context after compaction (`SessionStart` + `compact`)

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          { "type": "command",
            "command": "echo 'Reminder: use Bun, not npm. Run bun test before committing. Sprint: auth refactor.'" }
        ]
      }
    ]
  }
}
```

### 8. Auto-approve a specific permission prompt (`PermissionRequest`)

Skip the dialog for `ExitPlanMode` only — keep the matcher narrow.

```json
{
  "hooks": {
    "PermissionRequest": [
      {
        "matcher": "ExitPlanMode",
        "hooks": [
          { "type": "command",
            "command": "echo '{\"hookSpecificOutput\": {\"hookEventName\": \"PermissionRequest\", \"decision\": {\"behavior\": \"allow\"}}}'" }
        ]
      }
    ]
  }
}
```

To set a mode at the same time, include `updatedPermissions`:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PermissionRequest",
    "decision": {
      "behavior": "allow",
      "updatedPermissions": [
        { "type": "setMode", "mode": "acceptEdits", "destination": "session" }
      ]
    }
  }
}
```

> `PermissionRequest` hooks do **not** fire in non-interactive `-p` mode — use `PreToolUse` for automated approvals there.

### 9. Reload environment with `direnv` on `CwdChanged` / `FileChanged`

Both write to `$CLAUDE_ENV_FILE`, which Claude Code runs as a preamble before each Bash command.

```json
{
  "hooks": {
    "SessionStart": [ { "hooks": [ { "type": "command", "command": "direnv export bash > \"$CLAUDE_ENV_FILE\"" } ] } ],
    "CwdChanged":   [ { "hooks": [ { "type": "command", "command": "direnv export bash > \"$CLAUDE_ENV_FILE\"" } ] } ],
    "FileChanged":  [ { "matcher": ".envrc|.env", "hooks": [ { "type": "command", "command": "direnv export bash > \"$CLAUDE_ENV_FILE\"" } ] } ]
  }
}
```

### 10. Audit configuration changes (`ConfigChange`)

`ConfigChange` fires when an external process or editor modifies a settings/skills file mid-session. Log each change for compliance (exit 2 or `{"decision":"block"}` would block it instead):

```json
{
  "hooks": {
    "ConfigChange": [
      {
        "matcher": "",
        "hooks": [
          { "type": "command", "command": "jq -c '{timestamp: now | todate, source: .source, file: .file_path}' >> ~/claude-config-audit.log" }
        ]
      }
    ]
  }
}
```

> Coverage tip: `PostToolUse(Edit|Write)` misses files Claude creates by **running shell commands** through `Bash`. For full audit coverage, add a `Stop` hook that scans the working tree once per turn (e.g. `git status --porcelain`), or also match `Bash` and diff the tree per call.

---

## Non-command handler types

Beyond `type: "command"`, four handler types let hooks reach out, call models, or invoke MCP.

### HTTP hooks (`type: "http"`)

POST the same event JSON to a URL; the response body uses the same output format. Useful for shared audit/policy services.

```json
{
  "type": "http",
  "url": "http://localhost:8080/hooks/tool-use",
  "headers": { "Authorization": "Bearer $MY_TOKEN" },
  "allowedEnvVars": ["MY_TOKEN"],
  "timeout": 600
}
```

Only env vars listed in `allowedEnvVars` are interpolated into headers; any other `$VAR` resolves to empty. Non-2xx responses are non-blocking errors — to block, return a 2xx with the appropriate `hookSpecificOutput`.

### MCP tool hooks (`type: "mcp_tool"`)

Call a tool on an already-connected MCP server; its text output is treated like command stdout. Supports `${path}` substitution from the hook input.

```json
{ "type": "mcp_tool", "server": "my_server", "tool": "security_scan",
  "input": { "file_path": "${tool_input.file_path}" }, "timeout": 600 }
```

### Prompt hooks (`type: "prompt"`)

Single-turn LLM judgment (Haiku by default; override with `model`). Claude Code sends your `prompt` plus the hook input to the model, which returns `{"ok": true|false, "reason": "..."}`. On `ok:false` the effect depends on the event:

- `Stop` / `SubagentStop` — `reason` is fed back so Claude keeps working.
- `PreToolUse` — the tool call is denied and `reason` is returned to Claude as the tool error.
- `PostToolUse`, `PostToolBatch`, `UserPromptSubmit`, `UserPromptExpansion` — the turn ends and `reason` appears in the chat as a warning line.

Use `$ARGUMENTS` in the prompt to interpolate the hook input JSON; escape a literal `$` with a backslash (`\$1.00`).

```json
{
  "hooks": {
    "Stop": [
      { "hooks": [ { "type": "prompt",
        "prompt": "Check if all tasks are complete. If not, respond {\"ok\": false, \"reason\": \"what remains\"}." } ] }
    ]
  }
}
```

### Agent hooks (`type: "agent"`) — experimental

A subagent that can `Read`/`Grep`/`Glob`/run commands to verify a condition (e.g. that tests pass) before deciding. Same `ok`/`reason` format, longer default timeout (60s), up to 50 tool turns. Prefer command hooks for production. See **[./subagents.md](./subagents.md)**.

```json
{
  "hooks": {
    "Stop": [
      { "hooks": [ { "type": "agent",
        "prompt": "Verify that all unit tests pass. Run the suite and check results. $ARGUMENTS", "timeout": 120 } ] }
    ]
  }
}
```

---

## The `/hooks` menu

Run `/hooks` in any Claude Code session to open the hooks browser. It lists every available event with a count of configured hooks next to each. Select an event to see registered hooks; select a hook to see its event, matcher, type, source file, and command.

The `/hooks` menu is **read-only** — it cannot add, edit, or delete hooks. To change a hook, edit the settings JSON directly or ask Claude to make the change. Press `Esc` to return to the CLI. See **[./slash-commands.md](./slash-commands.md)** for the full command list.

---

## Debugging

- **Transcript view** (`Ctrl+O`) shows a one-line summary per hook: success is silent, blocking errors show stderr, non-blocking errors show `<hook name> hook error` + first stderr line.
- **Debug log**: launch `claude --debug-file /tmp/claude.log` and `tail -f /tmp/claude.log`, or run `/debug` mid-session, to see which hooks matched, exit codes, stdout, and stderr.
- **Test a hook manually** by piping sample JSON:
  ```bash
  echo '{"tool_name":"Bash","tool_input":{"command":"ls"}}' | ./my-hook.sh; echo $?
  ```

Common gotchas:

| Symptom | Likely cause / fix |
| :--- | :--- |
| Hook never fires | Matcher mismatch (case-sensitive), wrong event, or `PermissionRequest` used in `-p`. Check `/hooks`. |
| `command not found` | Use absolute paths or `${CLAUDE_PROJECT_DIR}`; add `"args": []` for exec form. |
| `jq: command not found` | Install `jq` or parse JSON with Python/Node. |
| Script doesn't run | `chmod +x` it. |
| JSON validation failed | A non-interactive shell profile printed junk before your JSON — guard `echo`s with `if [[ $- == *i* ]]`. |
| `/hooks` empty after edit | Invalid JSON (no trailing commas/comments), wrong file path, or restart the session to force a reload. |
| Stop hook block cap | Honor `stop_hook_active`; raise `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP` if needed. |

---

## Security considerations

> **Hooks execute arbitrary shell commands automatically, with your full user credentials and environment, with no confirmation.** The official docs carry a "USE AT YOUR OWN RISK" disclaimer: you are solely responsible for the commands you configure, hooks are provided as-is, and a malicious or buggy hook can exfiltrate data, delete files, or damage your system. Review every hook before adding it. (The exact disclaimer wording lives in the [Security considerations](https://code.claude.com/docs/en/hooks#security-considerations) section of the reference.)

Because hooks live in `settings.json` files, a **malicious repository** can ship a `.claude/settings.json` with a hostile hook. Anthropic's own writeup notes that, between mid-2025 and January 2026, responsibly disclosed vulnerabilities included cases where cloning a repo to review a PR could trigger hooks in `.claude/settings.json` that executed **before** the user accepted the trust prompt. Treat untrusted `.claude/` config like untrusted code, and do not open untrusted repos in Claude Code without inspecting their settings first.

Best practices:

- **Validate and sanitize inputs.** Never trust `tool_input` blindly; check paths and arguments.
- **Quote shell variables** — always `"$VAR"`, never bare `$VAR`.
- **Block path traversal** — reject `..` and resolve to absolute paths before acting.
- **Use absolute paths** for scripts (`${CLAUDE_PROJECT_DIR}/...`) so the wrong binary can't be picked up.
- **Skip sensitive files** — never let a hook read or transmit `.env`, `.git/`, keys, or secrets.
- **Scope HTTP secrets** with `allowedEnvVars`; prefer short-lived, narrowly scoped tokens.
- **Keep rewrite hooks (`updatedInput`/`updatedToolOutput`) minimal** and re-validate the result; prefer rejecting an input you don't fully understand over rewriting it.
- **Keep `PermissionRequest` auto-approve matchers narrow** — matching `.*` or an empty matcher auto-approves every prompt, including file writes and shell commands.

Defense-in-depth: `PreToolUse` `deny` fires before permission-mode checks and cannot be bypassed by the user's permission mode, which is exactly what makes it useful for enforcing policy — and exactly why a malicious one is dangerous. Use **deny rules in [./settings.md](./settings.md)** (which hooks cannot loosen) for hard guarantees, and consider `"disableAllHooks": true` in managed settings for locked-down environments.

---

## Related pages

- [./settings.md](./settings.md) — settings.json structure, precedence, permissions, env vars
- [./slash-commands.md](./slash-commands.md) — the `/hooks`, `/debug`, and `/config` commands
- [./subagents.md](./subagents.md) — subagents and the `agent`-type hook
- [./cli-and-shortcuts.md](./cli-and-shortcuts.md) — `--debug-file`, `--init-only`, transcript shortcuts
- [../capabilities/plugins.md](../capabilities/plugins.md) — bundling hooks in plugins (`hooks/hooks.json`)
- [../capabilities/mcp.md](../capabilities/mcp.md) — MCP servers, `mcp_tool` hooks, elicitation

## Open questions / to verify

- Which **older builds** expose which subset of the 30-event catalog (the catalog, blockability, timeouts, 10k cap, and `if`/matcher syntax here are all verified against the *current* reference). Verify your install with `/hooks`.
- The full per-event **stdin field schemas** beyond the highlights documented here (each event's reference section lists its own extra fields).
- `agent`-type hooks remain **experimental** — behavior, config, the 50-turn cap, and `async`/`asyncRewake` semantics may change between releases.
- Exact verbatim wording of the **Security considerations** disclaimer (its substance is verified; the literal block sits past what automated fetches return — read it live at the reference link).

## Sources

- [Hooks reference — code.claude.com/docs/en/hooks](https://code.claude.com/docs/en/hooks)
- [Automate actions with hooks (guide) — code.claude.com/docs/en/hooks-guide](https://code.claude.com/docs/en/hooks-guide)
- [Settings — code.claude.com/docs/en/settings](https://code.claude.com/docs/en/settings)
- [Permissions — code.claude.com/docs/en/permissions](https://code.claude.com/docs/en/permissions)
- [Environment variables — code.claude.com/docs/en/env-vars](https://code.claude.com/docs/en/env-vars)
- [Security guidance (security plugin built on hooks) — code.claude.com/docs/en/security-guidance](https://code.claude.com/docs/en/security-guidance)
- [Bash command validator example (anthropics/claude-code)](https://github.com/anthropics/claude-code/blob/main/examples/hooks/bash_command_validator_example.py)
