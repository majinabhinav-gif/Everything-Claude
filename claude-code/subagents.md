---
type: Feature
title: Claude Code — Subagents
description: Specialized, context-isolated agents that Claude Code's main thread delegates to via the Agent (formerly Task) tool, with their own context window, tools, model, and system prompt.
domain: claude-code
tags: [subagents, agents, task-tool, agent-tool, delegation, context-isolation, parallel-fan-out, code-reviewer, explore, plan, dot-claude-agents]
related: [slash-commands, settings, hooks, plugins, dispatch-remote, skills, cli-shortcuts, capabilities-and-modes, model-families, mcp]
resource: https://code.claude.com/docs/en/sub-agents
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://code.claude.com/docs/en/sub-agents
  - https://code.claude.com/docs/en/agent-sdk/subagents
  - https://code.claude.com/docs/en/model-config
  - https://code.claude.com/docs/en/agent-teams
  - https://code.claude.com/docs/en/plugins
  - https://code.claude.com/docs/en/hooks
---

# Claude Code — Subagents

Subagents are specialized AI assistants that the Claude Code main thread can delegate work to. Each runs in its **own fresh context window** with a custom system prompt, a restricted tool set, its own model, and independent permissions; it works in isolation and returns only a summary to the parent. Use them to keep verbose side-work (searches, logs, file dumps) out of your main conversation, to enforce tool/permission constraints, to specialize behavior, and to run independent investigations in parallel.

## At a glance

| | |
|---|---|
| **What it is** | Delegatable, context-isolated agent instances spawned via the **Agent** tool (renamed from **Task** in v2.1.63; `Task(...)` still works as an alias). Built-in types ship by default; custom types are Markdown-with-frontmatter files. |
| **Where you find it** | `/agents` interactive manager; definition files in `.claude/agents/*.md` (project) and `~/.claude/agents/*.md` (user); `--agents` JSON CLI flag; plugin `agents/` directories; the SDK `agents` option. Live subagents show in a panel below the prompt input and in `/agents` → **Running**. |
| **Who can use it** | Any Claude Code user (Pro / Max / Team / Enterprise plans that include Claude Code, and API/SDK users). Org admins can deploy **managed** subagents and restrict them via managed settings. |
| **Status** | GA and actively evolving. Recent additions (version gates confirmed against the docs' inline `min-version` annotations): Agent rename from Task (v2.1.63), forks GA (v2.1.117) and `/fork` default-on (v2.1.161), MCP-restriction coverage of frontmatter servers (v2.1.153), nested subagents (v2.1.172), nearest-wins for nested project dirs (v2.1.178), background permission prompts (v2.1.186), background-depth fix (v2.1.187). |

> Mental model: a **Skill** is a reusable prompt/workflow that runs *in your current context*; a **subagent** runs in an *isolated* context and returns a summary; an **agent team** / **background agent** runs many independent long-lived sessions. Reach for a subagent when a side task would flood your main conversation but you only need the answer back.

---

## What subagents are (and are not)

A subagent is **not** a separate session and **not** a separate model deployment — it is a child conversation spun up *within* your current session by the Agent tool. Key properties:

- **Fresh, isolated context.** The subagent does not see your conversation history, the files you've already read, or the skills you've already invoked. The *only* channel from parent to child is the Agent tool's `prompt` string — so any file paths, error messages, or decisions the subagent needs must be written into that prompt. (The one exception is a **fork**, covered below, which inherits the whole conversation.)
- **Returns only its final message.** Intermediate tool calls/results stay inside the child; the parent receives just the last message as the Agent tool result (and may further summarize it in its own reply).
- **Own system prompt.** Subagents receive *their* prompt plus minimal environment details (working directory, etc.) — **not** the full Claude Code system prompt.
- **Own model, tools, permissions.** Configurable per agent (see below).

### When to use a subagent vs. the main conversation

| Use the **main conversation** when | Use a **subagent** when |
|---|---|
| Frequent back-and-forth / iterative refinement | The task produces verbose output you don't need in main context |
| Multiple phases share lots of context (plan → implement → test) | You want to enforce specific tool/permission restrictions |
| Quick, targeted change | The work is self-contained and can return a summary |
| Latency matters (subagents start fresh and re-gather context) | You want parallel, independent investigations |

For a quick question about something already in your conversation, prefer **`/btw`** (sees full context, no tools, answer discarded) over a subagent. For reusable prompts that should run in main context, prefer a **Skill** (see `./../capabilities/skills.md`).

---

## The Agent tool (formerly Task) and fan-out

The **Agent tool** is how the parent spawns subagents. It was renamed from **Task** in Claude Code **v2.1.63**; existing `Task(...)` references in settings and agent definitions still work as aliases. Note the lingering naming split: current SDK releases emit `"Agent"` in `tool_use` blocks but still report `"Task"` in the `system:init` tools list and in `result.permission_denials[].tool_name` — check both when detecting invocations programmatically.

To **block all delegation**, deny the `Agent` tool itself in `permissions.deny`. To block one type, deny `Agent(<name>)`.

**Detecting an invocation programmatically (SDK).** A subagent spawn shows up as a `tool_use` block whose `name` is `"Agent"` (match `"Task"` too for older SDK builds); the requested type is in `block.input.subagent_type`. Any message produced *inside* a subagent carries a `parent_tool_use_id` field, so you can tell parent turns from child turns. The Agent tool result that comes back includes an `agentId: <id>` text trailer used for [resuming](#resuming-subagents).

### Parallel fan-out

Spawning several subagents in one turn lets independent subtasks finish in the time of the slowest one rather than the sum. Just ask for it in natural language:

```text
Research the authentication, database, and API modules in parallel using separate subagents.
```

```text
During this review, run the style-checker, security-scanner, and test-coverage subagents simultaneously.
```

> Caution: when subagents complete, their results return to the main conversation. Running many subagents that each return detailed results can itself consume significant context. For sustained parallelism beyond your context window, use **agent teams** (each worker gets its own context) — see `./dispatch-remote-routines.md`.

### Chaining (sequential)

```text
Use the code-reviewer subagent to find performance issues, then use the optimizer subagent to fix them.
```

Each subagent returns to Claude, which threads relevant context into the next one's prompt.

---

## Built-in subagents

Claude Code registers several built-in subagents that it invokes automatically. Each inherits the parent's permissions plus extra tool restrictions. **Explore** and **Plan** skip your `CLAUDE.md` files and the parent's git status to stay fast and cheap; every other built-in and custom subagent loads both.

| Type | Model | Tools | Purpose |
|---|---|---|---|
| **Explore** | Haiku (fast, low-latency) | Read-only (Write/Edit denied) | File discovery, code search, codebase exploration. Claude passes a thoroughness level: **quick** / **medium** / **very thorough**. One-shot (no resumable agent ID). |
| **Plan** | Inherits main model | Read-only (Write/Edit denied) | Research during **plan mode** so exploration output stays in a separate window while the main thread stays read-only. One-shot. |
| **general-purpose** | Inherits main model | All tools | Complex, multi-step work needing exploration *and* modification, complex reasoning, or dependent steps. Resumable. |
| **statusline-setup** | Sonnet | — | Invoked when you run `/statusline` to configure your status line. |
| **claude-code-guide** | Haiku | — | Invoked when you ask questions about Claude Code features. |
| **fork** | Same as main session | Same as main session | Inherits the full conversation (see [Forks](#forks-inherit-the-whole-conversation)). |

You rarely invoke these directly — Claude routes to them based on what you ask. Built-in subagents are **always registered in interactive sessions**. The current docs list only `statusline-setup` and `claude-code-guide` under the "Other" helper-agent tab — there is **no `output-style-setup` subagent** in the live roster (the original task brief mentioned it, but it is not present; this open question is now resolved). The set can still change between releases, so confirm against `/agents` → **Library** in your installed version. In non-interactive / SDK mode, set `CLAUDE_AGENT_SDK_DISABLE_BUILTIN_AGENTS=1` to remove all built-in types and supply only your own.

---

## Custom subagents: definition files

Subagents are Markdown files with YAML frontmatter; the **body becomes the system prompt**. Only `name` and `description` are required.

```markdown
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

> Loading semantics: files on disk are read **at session start** — edit a file directly and you must restart the session to pick it up. Subagents created/edited through `/agents` take effect **immediately**.

### Scope and precedence

Store the file in the location matching its scope. When two definitions share a `name`, the higher-priority location wins:

| Location | Scope | Priority | Created by |
|---|---|---|---|
| Managed settings dir `.claude/agents/` | Organization-wide | 1 (highest) | Deployed via managed settings |
| `--agents` CLI flag (JSON) | Current session only | 2 | Passed at launch; not saved to disk |
| `.claude/agents/` | Current project | 3 | `/agents` or manual; check into VCS |
| `~/.claude/agents/` | All your projects | 4 | `/agents` or manual |
| Plugin `agents/` directory | Where plugin is enabled | 5 (lowest) | Installed with a plugin |

Discovery details:
- **Project** dirs are found by walking up from the working directory, so every `.claude/agents/` between there and the repo root is scanned. As of **v2.1.178**, when nested project dirs define the same `name`, the one **closest to the working directory** wins.
- Directories added with `--add-dir` are also scanned (their `.claude/agents/` loads alongside project subagents).
- Both project and user scopes scan **recursively** — organize into `agents/review/`, `agents/research/`, etc. Identity comes only from `name`, not the path. Keep names unique across the whole tree; duplicate names *within one scope* silently discard one.
- **Plugin** subfolders become part of a **scoped identifier**: `agents/review/security.md` in plugin `my-plugin` registers as `my-plugin:review:security`.

### Supported frontmatter fields

| Field | Required | Notes |
|---|---|---|
| `name` | **Yes** | Lowercase letters and hyphens. Hooks receive it as `agent_type`. Filename need not match. |
| `description` | **Yes** | When Claude should delegate to this subagent. This is the routing signal — be specific; add "use proactively"/"use immediately after…" to encourage delegation. |
| `tools` | No | Allowlist of tools (see [Tool restriction](#tool-restriction)). Inherits all tools if omitted. Use `skills` (not `Skill` here) to preload skills. |
| `disallowedTools` | No | Denylist; removed from the inherited/specified set. |
| `model` | No | `sonnet`, `opus`, `haiku`, `fable`, a full ID (e.g. `claude-opus-4-8`, `claude-sonnet-4-6`), or `inherit`. **Defaults to `inherit`.** |
| `permissionMode` | No | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, or `plan`. Ignored for plugin subagents. |
| `maxTurns` | No | Max agentic turns before the subagent stops. |
| `skills` | No | Skills to **preload** (full content injected at startup). Unlisted skills remain invocable via the Skill tool. |
| `mcpServers` | No | MCP servers for this subagent — a name referencing an already-configured server, or an inline definition. Ignored for plugin subagents. |
| `hooks` | No | Lifecycle hooks scoped to this subagent. Ignored for plugin subagents. |
| `memory` | No | Persistent memory scope: `user`, `project`, or `local`. Enables cross-session learning. |
| `background` | No | `true` to always run as a background task. Default `false`. |
| `effort` | No | `low`, `medium`, `high`, `xhigh`, `max` — or a raw `number` in the SDK `AgentDefinition` (available levels depend on the model). Overrides session effort; defaults to inheriting the session level. |
| `isolation` | No | `worktree` runs the subagent in a temporary git worktree (isolated repo copy, branched by default from your default branch). Auto-cleaned if no changes. |
| `color` | No | Display color in the task list/transcript: `red`, `blue`, `green`, `yellow`, `purple`, `orange`, `pink`, `cyan`. |
| `initialPrompt` | No | Auto-submitted as the first user turn **only when this agent runs as the main session** (via `--agent` or the `agent` setting). Commands/skills are processed; prepended to any user prompt. Ignored when invoked as a subagent. |

> The body of the file = the system prompt. The `--agents` JSON flag and the SDK use a `prompt` field for the same content. Subagents get only this prompt plus basic environment details — not the full Claude Code system prompt.

### Working-directory note

A subagent starts in the main conversation's current working directory. Inside a subagent, `cd` does **not** persist between Bash/PowerShell calls and does not affect the main conversation's cwd. For an isolated repo copy, use `isolation: worktree` (see `./dispatch-remote-routines.md` for worktrees).

---

## The `/agents` interactive manager

Run `/agents` to open the tabbed manager:

- **Running** tab — lists live and recently finished subagents; open or stop them.
- **Library** tab — view all subagents (built-in, user, project, plugin); create new (guided setup or **Generate with Claude**); edit config and tool access; delete custom ones; see which wins when duplicates exist.

Quickstart flow (creating a user-level reviewer):

```text
/agents
```

1. **Library** → **Create new agent** → **Personal** (saves to `~/.claude/agents/`).
2. **Generate with Claude** → describe the agent, e.g. *"A code improvement agent that scans files and suggests improvements for readability, performance, and best practices…"* — Claude writes the `name`, `description`, and system prompt.
3. **Select tools** → deselect all except **Read-only tools** for a reviewer.
4. **Select model** → e.g. **Sonnet**.
5. **Choose a color** → for UI identification.
6. **Configure memory** → **User scope** for a persistent memory dir, or **None**.
7. Press `s`/`Enter` to save, or `e` to save and edit in your editor. Available immediately:

```text
Use the code-improver agent to suggest improvements in this project
```

### `--agents` JSON flag (session-scoped, not saved)

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  },
  "debugger": {
    "description": "Debugging specialist for errors and test failures.",
    "prompt": "You are an expert debugger. Analyze errors, identify root causes, and provide fixes."
  }
}'
```

On Windows PowerShell, wrap the JSON in a single-quoted here-string (`@'...'@`). The flag accepts the same fields as file frontmatter, with `prompt` standing in for the markdown body.

---

## Invoking subagents: automatic vs. explicit

Claude decides delegation from your request, the subagent's `description`, and current context. Three escalating ways to invoke explicitly:

| Pattern | Behavior | Example |
|---|---|---|
| **Natural language** | Name it; Claude decides whether to delegate | `Use the test-runner subagent to fix failing tests` |
| **@-mention** | Guarantees that subagent runs for one task | `@"code-reviewer (agent)" look at the auth changes` — or type `@agent-code-reviewer`, or `@agent-my-plugin:code-reviewer` for plugins |
| **Session-wide** | The whole session adopts that subagent's prompt, tools, and model | `claude --agent code-reviewer`, or set `"agent": "code-reviewer"` in `.claude/settings.json` |

With `--agent`, the subagent's system prompt **replaces** the default Claude Code system prompt entirely (like `--system-prompt`); `CLAUDE.md` and project memory still load. The name appears as `@<name>` in the startup header and persists on resume. The CLI flag overrides the `agent` setting if both are present. For a plugin agent, pass `--agent security-reviewer` (or the scoped `my-plugin:security-reviewer` / `my-plugin:review:security` to disambiguate).

---

## Tool restriction

Subagents inherit internal tools and MCP tools from the main conversation by default. Restrict with `tools` (allowlist) or `disallowedTools` (denylist). If both are set, `disallowedTools` applies **first**, then `tools` resolves against what remains; a tool in both is removed.

```yaml
# Allowlist: exactly these four tools, nothing else
---
name: safe-researcher
description: Research agent with restricted capabilities
tools: Read, Grep, Glob, Bash
---
```

```yaml
# Denylist: inherit everything except file writes
---
name: no-writes
description: Inherits every tool except file writes
disallowedTools: Write, Edit
---
```

**MCP server-level patterns** work in both fields: `mcp__<server>` or `mcp__<server>__*` grants/removes every tool from that server; in `disallowedTools`, `mcp__*` removes every MCP tool from any server.

```yaml
---
name: local-only
description: Inherits every tool except those from the github MCP server
disallowedTools: mcp__github
---
```

**Tools never available to subagents** (they depend on the main UI/session state), even if listed: `AskUserQuestion`, `EnterPlanMode`, `ScheduleWakeup`, `WaitForMcpServers`, and `ExitPlanMode` (unless the subagent's `permissionMode` is `plan`).

**Common combinations:**

| Use case | Tools |
|---|---|
| Read-only analysis | `Read`, `Grep`, `Glob` |
| Test execution | `Bash`, `Read`, `Grep` |
| Code modification | `Read`, `Edit`, `Write`, `Grep`, `Glob` |
| Full access | (omit `tools`) |

### Restricting which subagents can be spawned

When an agent runs as the **main thread** via `claude --agent`, restrict which child types it may spawn using `Agent(...)` allowlist syntax in `tools`:

```yaml
---
name: coordinator
description: Coordinates work across specialized agents
tools: Agent(worker, researcher), Read, Bash
---
```

- `Agent(worker, researcher)` — only those two types may be spawned.
- `Agent` (no parens) — may spawn any type.
- Omit `Agent` entirely — cannot spawn any subagent.

Inside a *subagent* definition, listing `Agent` lets it spawn nested subagents, but any type list in the parens is ignored. To block specific types while allowing others, use `permissions.deny` instead (next).

### Disabling specific subagents

```json
{
  "permissions": {
    "deny": ["Agent(Explore)", "Agent(my-custom-agent)"]
  }
}
```

Or via CLI: `claude --disallowedTools "Agent(Explore)"`. Works for both built-in and custom types.

---

## Model selection and effort

`model` resolves in this order (first match wins):

1. `CLAUDE_CODE_SUBAGENT_MODEL` environment variable, if set
2. The per-invocation `model` parameter Claude passes when spawning
3. The subagent's `model` frontmatter
4. The main conversation's model

All three configurable sources are checked against your org's `availableModels` allowlist; a value resolving to an excluded model is ignored and the subagent runs on the inherited model. Route cheap/fast work (search, triage) to `haiku`, deep reasoning (security review) to `opus`. `effort` (`low`…`max`) overrides session effort while the subagent is active.

---

## Permission modes

`permissionMode` controls how the subagent handles permission prompts:

| Mode | Behavior |
|---|---|
| `default` | Standard permission checking with prompts |
| `acceptEdits` | Auto-accept edits + common filesystem commands within the working dir / additional dirs |
| `auto` | Background classifier reviews commands and protected-dir writes |
| `dontAsk` | Auto-deny prompts (explicitly allowed tools still work) |
| `bypassPermissions` | Skip permission prompts (use with caution) |
| `plan` | Plan mode (read-only exploration) |

Inheritance rules: if the **parent** uses `bypassPermissions` or `acceptEdits`, that takes precedence and the child cannot override it. If the parent is in `auto` mode, the child inherits auto mode and its own `permissionMode` is ignored. See `./settings.md` and `./hooks.md` for the broader permission system.

---

## Skills, MCP, memory, and hooks per subagent

### Preload skills

```yaml
---
name: api-developer
description: Implement API endpoints following team conventions
skills:
  - api-conventions
  - error-handling-patterns
---

Implement API endpoints. Follow the conventions and patterns from the preloaded skills.
```

Full skill content is injected at startup (not just the description). This controls what's *preloaded*, not what's *accessible* — without it the subagent can still discover/invoke skills via the Skill tool. You cannot preload a skill marked `disable-model-invocation: true`. See `./../capabilities/skills.md`.

### Scope MCP servers to a subagent

```yaml
---
name: browser-tester
description: Tests features in a real browser using Playwright
mcpServers:
  - playwright:               # inline: scoped to this subagent only
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
  - github                    # reference: reuses an already-configured server
---

Use the Playwright tools to navigate, screenshot, and interact with pages.
```

Defining a server inline here (rather than in `.mcp.json`) keeps its tool descriptions out of the main conversation's context. As of v2.1.153, managed/`--strict-mcp-config`/`--bare` MCP restrictions also cover subagent-frontmatter servers. See `./../capabilities/mcp.md`.

### Persistent memory

```yaml
---
name: code-reviewer
description: Reviews code for quality and best practices
memory: user
---
```

| Scope | Location | Use when |
|---|---|---|
| `user` | `~/.claude/agent-memory/<name>/` | learnings apply across all projects |
| `project` | `.claude/agent-memory/<name>/` | project-specific, shareable via VCS (**recommended default**) |
| `local` | `.claude/agent-memory-local/<name>/` | project-specific, not checked in |

When enabled, the subagent's prompt gains read/write instructions plus the first 200 lines or 25KB of `MEMORY.md` (whichever comes first), and Read/Write/Edit are auto-enabled. Prompt it to consult memory before and update it after a task to build institutional knowledge.

### Hooks

Two places to hook:

1. **In subagent frontmatter** — run only while that subagent is active; cleaned up when it finishes. `Stop` is converted to `SubagentStop` at runtime.

```yaml
---
name: code-reviewer
description: Review code changes with automatic linting
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh $TOOL_INPUT"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
---
```

2. **In `settings.json`** — respond to subagent lifecycle in the main session: `SubagentStart` and `SubagentStop` (both support a matcher on the agent-type name).

```json
{
  "hooks": {
    "SubagentStart": [
      { "matcher": "db-agent", "hooks": [ { "type": "command", "command": "./scripts/setup-db-connection.sh" } ] }
    ],
    "SubagentStop": [
      { "hooks": [ { "type": "command", "command": "./scripts/cleanup-db-connection.sh" } ] }
    ]
  }
}
```

See `./hooks.md` for the full event/IO reference.

---

## Foreground vs. background, nesting, forks

### Foreground vs. background

- **Foreground** subagents block the main conversation; permission prompts pass through to you.
- **Background** subagents run concurrently while you keep working. As of **v2.1.186**, a background subagent's permission prompt surfaces in your main session, names the asking subagent, and you can approve or press **Esc** to deny that one call without killing it. (Before v2.1.186, background subagents auto-denied any prompting tool call.)

Claude chooses fg/bg per task. You can say "run this in the background", press **Ctrl+B** to background a running task, or set `background: true` in frontmatter. `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1` disables all background tasks.

### Nested subagents

As of **v2.1.172**, a subagent can spawn its own subagents (e.g. a reviewer that dispatches a verifier per finding). Only the top-level subagent's summary returns to you. The subagent panel shows the full tree (`(+N)` descendant counts). **Depth is capped**: a subagent at **depth five** does not receive the Agent tool and cannot spawn further — the limit is fixed and not configurable. As of v2.1.187, a background subagent's depth is fixed when first spawned and resuming doesn't change it. Omit `Agent` from `tools` to stop a subagent from spawning children.

### Forks inherit the whole conversation

A **fork** is a subagent that inherits the *entire conversation so far* instead of starting fresh — same system prompt, tools, model, and message history as the main session. Its own tool calls still stay out of your conversation; only the final result returns. Forks require **v2.1.117+**; the `/fork` command is **default-on from v2.1.161** (earlier needs `CLAUDE_CODE_FORK_SUBAGENT=1`).

```text
/fork draft unit tests for the parser changes so far
```

| | Fork | Named subagent |
|---|---|---|
| Context | Full conversation history | Fresh + the prompt you pass |
| System prompt + tools | Same as main session | From the definition file |
| Model | Same as main session | From `model` field |
| Prompt cache | **Shared** with main session (cheaper) | Separate cache |

Enabling fork mode also runs **every** subagent spawn in the background. A fork cannot spawn another fork (but can spawn other subagent types, which count toward the depth limit). Panel controls while forks run: `↑`/`↓` navigate, `Enter` opens transcript / sends follow-ups, `x` dismiss/stop, `Esc` return to prompt.

---

## What loads at startup (context contents)

A non-fork subagent's initial context contains:

- **System prompt** — the agent's own prompt + appended environment details (not the full Claude Code system prompt).
- **Task message** — the delegation prompt Claude writes at hand-off.
- **CLAUDE.md and memory** — every level of the memory hierarchy the main conversation loads (`~/.claude/CLAUDE.md`, project rules, `CLAUDE.local.md`, managed policy). **Explore and Plan skip this.**
- **Git status** — a snapshot from the start of the parent session (absent outside a git repo or when `includeGitInstructions` is `false`). Explore and Plan skip it regardless.
- **Preloaded skills** — full content of any skill in the `skills` field.

Because the only parent→child channel is the prompt, restate any rule that *must* reach the child (e.g. "ignore the `vendor/` directory") in the delegation prompt — especially for Explore/Plan, which skip CLAUDE.md.

---

## Resuming subagents

Each invocation normally creates a new instance with fresh context. To **continue** instead of restarting, ask Claude to resume — resumed subagents retain full history (tool calls, results, reasoning).

```text
Use the code-reviewer subagent to review the authentication module
[Agent completes]

Continue that code review and now analyze the authorization logic
[Claude resumes the subagent with full prior context]
```

When a subagent completes, Claude receives its **agent ID**; it uses the `SendMessage` tool (with the ID or name in the `to` field) to resume. Sending a message to a stopped subagent auto-resumes it in the background without a new `Agent` invocation. **Explore and Plan are one-shot and return no agent ID — use `general-purpose` or a custom subagent when you need to resume.** Transcripts live at `~/.claude/projects/{project}/{sessionId}/subagents/agent-{agentId}.jsonl`, persist independently of main-conversation compaction, and are cleaned up per `cleanupPeriodDays` (default 30).

**From the SDK**, resuming is explicit: capture `session_id` from the messages of the first `query()`, parse the `agentId: <id>` trailer out of the Agent tool result's text, then run a second `query({ ..., resume: sessionId })` and name the agent ID in the prompt (e.g. *"Resume agent abc123 and …"*). You must resume the **same session** (each `query()` starts a fresh session by default) and pass the **same** agent definition in `agents` on both calls. `SendMessage` is always available for resuming by ID or name; the structured team-protocol messages (`shutdown_request`, `plan_approval_response`) require **agent teams** to be enabled.

**Auto-compaction.** Subagents auto-compact with the same logic as the main conversation (and honor `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`). A compaction is logged in the subagent transcript as a `system` / `compact_boundary` event whose `compactMetadata.preTokens` records the token count just before compaction:

```json
{ "type": "system", "subtype": "compact_boundary", "compactMetadata": { "trigger": "auto", "preTokens": 167189 } }
```

---

## Worked examples

### Code reviewer (read-only, focused)

```markdown
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code is clear and readable; functions/variables well-named
- No duplicated code; proper error handling
- No exposed secrets or API keys; input validation implemented
- Good test coverage; performance considerations addressed

Provide feedback by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)

Include specific examples of how to fix each issue.
```

### Test-runner (Bash + read, summary-only)

```markdown
---
name: test-runner
description: Runs and analyzes test suites. Use proactively for test execution and coverage analysis.
tools: Bash, Read, Grep
model: haiku
---

You are a test execution specialist. Run the project's tests and report only the
failing tests with their error messages and the most likely root cause for each.
Do not modify code. Keep the summary concise.
```

Invoke with isolation so the verbose output stays out of main context:

```text
Use the test-runner subagent to run the test suite and report only the failing tests with their error messages.
```

### Parallel fan-out search

```text
Spawn three Explore subagents in parallel: one to map the auth flow, one to map the
billing module, and one to map the public API surface. Have each return a 5-bullet summary,
then synthesize a single architecture overview.
```

### Database query validator (Bash gated by a PreToolUse hook)

```markdown
---
name: db-reader
description: Execute read-only database queries. Use when analyzing data or generating reports.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database analyst with read-only access. Execute SELECT queries to answer
questions about the data. If asked to INSERT/UPDATE/DELETE or modify schema, explain
that you only have read access.
```

```bash
#!/bin/bash
# ./scripts/validate-readonly-query.sh — blocks SQL writes, allows SELECT
INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')
[ -z "$COMMAND" ] && exit 0
# Match the official example's keyword set, including REPLACE and MERGE
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE|REPLACE|MERGE)\b' > /dev/null; then
  echo "Blocked: Write operations not allowed. Use SELECT queries only." >&2
  exit 2   # exit code 2 blocks the tool call and feeds the message back to Claude via stderr
fi
exit 0
```

On Windows, write the script in PowerShell and add `shell: powershell` to the hook entry.

### SDK: programmatic subagents (TypeScript)

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Review the authentication module for security issues",
  options: {
    // Include "Agent" so subagent invocations auto-approve without a prompt
    allowedTools: ["Read", "Grep", "Glob", "Agent"],
    agents: {
      "code-reviewer": {
        description: "Expert code review specialist. Use for quality, security, and maintainability reviews.",
        prompt: "You are a code review specialist...",
        tools: ["Read", "Grep", "Glob"],
        model: "sonnet"
      },
      "test-runner": {
        description: "Runs and analyzes test suites.",
        prompt: "You are a test execution specialist...",
        tools: ["Bash", "Read", "Grep"]
      }
    }
  }
})) {
  if ("result" in message) console.log(message.result);
}
```

The `AgentDefinition` fields mirror frontmatter: `description` (required, `string`), `prompt` (required, `string`, = the body), `tools` (`string[]`), `disallowedTools` (`string[]`), `model` (`string`), `skills` (`string[]`), `memory` (`'user'|'project'|'local'`), `mcpServers` (`(string|object)[]`), `initialPrompt` (`string`), `maxTurns` (`number`), `background` (`boolean`), `effort` (`'low'|'medium'|'high'|'xhigh'|'max'|number`), `permissionMode`. In the Python SDK these field names use **camelCase** to match the wire format. Programmatically defined agents take precedence over filesystem agents of the same name.

You can build definitions **dynamically at query time** (a factory that returns an `AgentDefinition`) to vary the model/prompt per request — e.g. route strict security reviews to `opus` and routine ones to `sonnet`. Two gotchas: (1) include `Agent` in `allowedTools` (or handle it in your `canUseTool` callback), or invocations are denied — especially under `dontAsk`; (2) on **Windows**, very long `prompt` strings can hit the 8191-character command-line limit, so keep SDK prompts concise or move long instructions into filesystem agents.

For runs coordinating *dozens to hundreds* of agents, use the **`Workflow`** tool (TypeScript Agent SDK v0.3.149+; add `Workflow` to `allowedTools` to auto-approve) instead of turn-by-turn delegation — it moves orchestration into a script the runtime executes outside the conversation context.

---

## Relationship to plugins and teams

- **Plugins** can ship subagents in their `agents/` directory (lowest precedence). They appear in `/agents` and the @-typeahead under scoped names (`my-plugin:code-reviewer`). For security, **plugin subagents ignore `hooks`, `mcpServers`, and `permissionMode`** — copy the file into `.claude/agents/` to use those. See `./../capabilities/plugins.md`.
- **Agent teams** and **background agents** are the next tier up: many independent, long-lived workers each with their own context, monitored from one place, optionally communicating via structured protocol messages (`shutdown_request`, `plan_approval_response`). Subagent definitions can seed teammates (their `tools`/`model` apply, body appended to the teammate's prompt). See `./dispatch-remote-routines.md`.

---

## Related pages

- [Slash commands](./slash-commands.md) — `/agents`, `/fork`, `/statusline`, `/btw`, and custom commands
- [Settings](./settings.md) — `agent` setting, `permissions.deny`, `cleanupPeriodDays`, env vars like `CLAUDE_CODE_SUBAGENT_MODEL`
- [Hooks](./hooks.md) — `SubagentStart`/`SubagentStop`, `PreToolUse`/`PostToolUse`, exit codes
- [CLI and shortcuts](./cli-and-shortcuts.md) — `--agent`, `--agents`, `--disallowedTools`, `--add-dir`, Ctrl+B
- [Dispatch, Remote Control, Routines](./dispatch-remote-routines.md) — agent teams, background agents, worktrees, Agent View
- [Plugins](../capabilities/plugins.md) — bundling subagents in a plugin
- [Skills](../capabilities/skills.md) — preloading skills; subagents vs. skills
- [MCP](../capabilities/mcp.md) — scoping MCP servers to a subagent
- [Capabilities and modes](../models/capabilities-and-modes.md) — extended thinking/effort, models
- [Model families](../models/model-families.md) — model IDs (`claude-opus-4-8`, `claude-sonnet-4-6`), aliases, pricing/context

## Open questions / to verify

- **Resolved (2026-06-26):** The "Other" built-in helper-agent roster is `statusline-setup` (Sonnet) and `claude-code-guide` (Haiku) only — there is **no `output-style-setup` subagent** in the live docs. The roster can still shift between releases; reconfirm via `/agents` → Library on your build.
- **Resolved (2026-06-26):** All version gates are confirmed verbatim against the docs' inline `min-version` annotations: v2.1.63 (Agent rename), v2.1.117 (forks GA) / v2.1.161 (`/fork` default-on), v2.1.153 (frontmatter-MCP restrictions, and `/model`-saves-default), v2.1.172 (nesting), v2.1.178 (nearest-wins), v2.1.186 (background permission prompts), v2.1.187 (background-depth fix).
- The nested-subagent **depth limit of five** is documented as "fixed and not configurable." Confirm it has not changed in builds newer than the captured docs.
- `effort` accepts `low|medium|high|xhigh|max` (and a raw `number` in the SDK), with the docs stating "available levels depend on the model." The exact per-model availability of `xhigh`/`max` is not enumerated in these pages — verify against `models/capabilities-and-modes.md` / the model docs.
- Whether the `general-purpose` subagent is itself resumable in all builds (docs say one-shot Explore/Plan return no agent ID, implying general-purpose does) — confirmed by the SDK resume example, but reconfirm if behavior changes.

## Sources

- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents)
- [Subagents in the SDK — Claude Agent SDK Docs](https://code.claude.com/docs/en/agent-sdk/subagents)
- [Model configuration — Claude Code Docs](https://code.claude.com/docs/en/model-config)
- [Agent teams — Claude Code Docs](https://code.claude.com/docs/en/agent-teams)
- [Plugins — Claude Code Docs](https://code.claude.com/docs/en/plugins)
- [Hooks — Claude Code Docs](https://code.claude.com/docs/en/hooks)
