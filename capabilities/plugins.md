---
type: Extension System
title: Claude Code Plugins
description: Shareable bundles that add slash commands, subagents, hooks, MCP servers, skills, and more to Claude Code at once, distributed through git-hosted marketplaces.
domain: capabilities
tags: [plugins, marketplace, claude-code, plugin.json, marketplace.json, extensions, distribution, hooks, mcp, skills, subagents]
related: [skills, subagents, hooks, slash-commands, mcp, settings, claude-code-overview]
resource: https://code.claude.com/docs/en/plugins
timestamp: 2026-06-26T00:00:00Z
confidence: high
verified: 2026-06-26
okf_version: "0.1"
sources:
  - https://code.claude.com/docs/en/plugins
  - https://code.claude.com/docs/en/plugins-reference
  - https://code.claude.com/docs/en/plugin-marketplaces
  - https://code.claude.com/docs/en/discover-plugins
  - https://github.com/anthropics/claude-plugins-official
  - https://github.com/anthropics/claude-plugins-community
---

# Claude Code Plugins

A **plugin** is a self-contained, shareable directory that extends Claude Code with custom functionality — bundling slash commands, subagents, hooks, MCP servers, skills, LSP servers, monitors, themes, and output styles into one versioned unit you can install with a single command. Plugins solve the "scatter problem": instead of asking teammates to hand-copy a `.claude/commands/` file here, paste a hook into `settings.json` there, and configure an MCP server somewhere else, you ship all of it as one git-distributable package. They are distributed through **marketplaces** (a `marketplace.json` catalog hosted in a git repo) and managed with the `/plugin` command and the `claude plugin` CLI.

## At a glance

| | |
|---|---|
| **What it is** | A bundle of Claude Code extensions (commands, agents, hooks, MCP/LSP servers, skills, monitors, themes, output styles) packaged behind a `.claude-plugin/plugin.json` manifest |
| **Where you find it** | The `/plugin` command (interactive manager) inside Claude Code; the `claude plugin ...` CLI subcommands; `--plugin-dir` / `--plugin-url` flags for local testing |
| **Who can use it by plan** | Anyone running Claude Code (the plugin system is a CLI feature, not gated by Claude.ai plan). Org admins on Team/Enterprise get managed-settings controls (`strictKnownMarketplaces`, `extraKnownMarketplaces`, `enabledPlugins`) |
| **Status** | Generally available and actively evolving. Several sub-features are version-gated (noted inline, e.g. `defaultEnabled` requires v2.1.154+). Themes and monitors are flagged experimental |

> **Trust warning (load-bearing):** Plugins and marketplaces are *highly trusted* components that can execute arbitrary code on your machine with your user privileges. Anthropic does not control what MCP servers, files, or software a third-party plugin includes and cannot verify it works as intended. Only install plugins and add marketplaces from sources you trust. Organizations can lock this down with [`strictKnownMarketplaces`](../claude-code/settings.md).

---

## What a plugin can bundle

A single plugin can contribute any combination of these components. Each one is documented in depth on its own page — this page covers how they are *packaged and shipped*.

| Component | Default location | Cross-reference |
|---|---|---|
| **Slash commands / skills** | `skills/<name>/SKILL.md` (preferred) or `commands/*.md` (flat, legacy), or a single `SKILL.md` at plugin root | [skills.md](./skills.md), [slash-commands.md](../claude-code/slash-commands.md) |
| **Subagents** | `agents/*.md` | [subagents.md](../claude-code/subagents.md) |
| **Hooks** | `hooks/hooks.json` (or inline in `plugin.json`) | [hooks.md](../claude-code/hooks.md) |
| **MCP servers** | `.mcp.json` (or inline) | [mcp.md](./mcp.md) |
| **LSP servers** | `.lsp.json` (or inline) — real-time code intelligence | [capabilities-and-modes.md](../models/capabilities-and-modes.md) |
| **Background monitors** | `monitors/monitors.json` (experimental) | — |
| **Themes** | `themes/*.json` (experimental) | — |
| **Output styles** | `output-styles/*.md` | — |
| **Bundled executables** | `bin/` — added to the Bash tool's `PATH` while the plugin is enabled | — |
| **Default settings** | `settings.json` at plugin root (only `agent` and `subagentStatusLine` keys honored) | [settings.md](../claude-code/settings.md) |

Plugin skills, agents, and commands are **namespaced** by the plugin name to prevent collisions. A skill `hello` in a plugin named `my-first-plugin` is invoked as `/my-first-plugin:hello`; an agent `agent-creator` in `plugin-dev` shows up as `plugin-dev:agent-creator`.

> A `CLAUDE.md` file at the plugin root is **not** loaded as project context. To ship instructions into Claude's context, put them in a skill.

### Plugin vs. standalone `.claude/` configuration

| Approach | Skill names | Best for |
|---|---|---|
| **Standalone** (`.claude/` directory) | `/hello` | Personal workflows, project-specific tweaks, quick experiments |
| **Plugins** | `/plugin-name:hello` | Sharing with teammates/community, versioned releases, reuse across projects, marketplace distribution |

The docs' guidance: start standalone in `.claude/` for fast iteration, then convert to a plugin when ready to share.

---

## The plugin manifest: `.claude-plugin/plugin.json`

The manifest lives at `.claude-plugin/plugin.json`. **It is optional** — if omitted, Claude Code auto-discovers components in their default locations and derives the plugin name from the directory name. Provide a manifest when you need metadata or custom component paths.

> **Critical structure rule:** Only `plugin.json` goes inside `.claude-plugin/`. Every other directory (`skills/`, `commands/`, `agents/`, `hooks/`, `themes/`, `monitors/`, `output-styles/`) lives at the **plugin root**, not inside `.claude-plugin/`. This is the single most common mistake.

### Minimal manifest

```json
{
  "name": "my-first-plugin",
  "description": "A greeting plugin to learn the basics",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

If you include a manifest, `name` is the **only required field**.

### Complete schema

```json
{
  "name": "plugin-name",
  "displayName": "Plugin Name",
  "version": "1.2.0",
  "description": "Brief plugin description",
  "author": {
    "name": "Author Name",
    "email": "author@example.com",
    "url": "https://github.com/author"
  },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/author/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "skills": "./custom/skills/",
  "commands": ["./custom/commands/special.md"],
  "agents": ["./custom/agents/reviewer.md"],
  "hooks": "./config/hooks.json",
  "mcpServers": "./mcp-config.json",
  "outputStyles": "./styles/",
  "lspServers": "./.lsp.json",
  "experimental": {
    "themes": "./themes/",
    "monitors": "./monitors.json"
  },
  "dependencies": [
    "helper-lib",
    { "name": "secrets-vault", "version": "~2.1.0" }
  ]
}
```

### Metadata fields

| Field | Type | Notes |
|---|---|---|
| `name` | string | **Required.** Unique identifier, kebab-case, no spaces. Used for namespacing. |
| `displayName` | string | Human-readable name shown in the `/plugin` picker; may contain spaces/casing. Falls back to `name`. *Requires v2.1.143+.* |
| `version` | string | Optional semver. **Setting it pins the plugin** — users only get updates when you bump it. Omit to fall back to the git commit SHA (every commit = a new version). If also set in the marketplace entry, `plugin.json` wins silently. |
| `description` | string | Shown in the plugin manager when browsing/installing. |
| `author` | object | `{ name, email?, url? }`. `name` required if present. |
| `homepage` | string | Documentation URL. |
| `repository` | string | Source code URL. |
| `license` | string | SPDX identifier, e.g. `MIT`, `Apache-2.0`. |
| `keywords` | array | Discovery tags. |
| `defaultEnabled` | boolean | Whether the plugin starts enabled when the user has no prior setting. Default `true`. Set `false` to install disabled until opt-in. *Requires v2.1.154+.* |
| `$schema` | string | JSON Schema URL for editor autocomplete (`https://json.schemastore.org/claude-code-plugin-manifest.json`). Ignored at load time. |

**Unrecognized top-level fields are ignored** (warned, not errored) — so one `plugin.json` can double as a `package.json`, VS Code/Cursor extension manifest, or MCPB/DXT manifest. Wrong *types* still fail (e.g. `keywords` as a string). Use `claude plugin validate --strict` to treat warnings as errors in CI.

### Component path fields

When you point a manifest key at a custom path, behavior differs by field:

- **Replaces the default directory**: `commands`, `agents`, `outputStyles`, `experimental.themes`, `experimental.monitors`. To keep the default *and* add more, list it explicitly: `"commands": ["./commands/", "./extras/"]`.
- **Adds to the default**: `skills`. The default `skills/` is always scanned; listed paths load alongside it.
- **Own merge rules**: `hooks`, `mcpServers`, `lspServers` (see each component's section/page).

All paths must be **relative to the plugin root and start with `./`**. Absolute paths and `../` traversal are rejected.

### Advanced manifest keys

| Field | Purpose |
|---|---|
| `userConfig` | Values Claude Code prompts the user for when the plugin is enabled (instead of hand-editing settings). |
| `channels` | Declare message channels (Telegram/Slack/Discord-style) bound to a plugin MCP server. |
| `dependencies` | Other plugins this one requires, optionally with semver constraints — auto-installed on install. |

#### `userConfig` example

```json
{
  "userConfig": {
    "api_endpoint": {
      "type": "string",
      "title": "API endpoint",
      "description": "Your team's API endpoint"
    },
    "api_token": {
      "type": "string",
      "title": "API token",
      "description": "API authentication token",
      "sensitive": true
    }
  }
}
```

Supported option fields: `type` (`string`/`number`/`boolean`/`directory`/`file`), `title`, `description` (all required), plus optional `sensitive`, `required`, `default`, `multiple`, `min`/`max`. Each value is available as `${user_config.KEY}` inside MCP/LSP configs, hook commands, and monitor commands, and exported to subprocesses as `CLAUDE_PLUGIN_OPTION_<KEY>`. Non-sensitive values land in `settings.json` under `pluginConfigs[<plugin-id>].options`; sensitive ones go to the system keychain (~2 KB budget).

---

## Conventional directory layout

A fully-featured plugin looks like this. Almost all directories are optional.

```text
enterprise-plugin/
├── .claude-plugin/           # Metadata directory (optional)
│   └── plugin.json           #   ← the ONLY thing in here
├── skills/                   # Skills as <name>/SKILL.md
│   ├── code-reviewer/
│   │   └── SKILL.md
│   └── pdf-processor/
│       ├── SKILL.md
│       └── scripts/
├── commands/                 # Skills as flat .md files (legacy; prefer skills/)
│   ├── status.md
│   └── logs.md
├── agents/                   # Subagent definitions
│   ├── security-reviewer.md
│   └── compliance-checker.md
├── output-styles/            # Output style definitions
│   └── terse.md
├── themes/                   # Color themes (experimental)
│   └── dracula.json
├── monitors/                 # Background monitors (experimental)
│   └── monitors.json
├── hooks/                    # Hook configurations
│   └── hooks.json
├── bin/                      # Executables added to the Bash tool's PATH
│   └── my-tool
├── settings.json             # Default settings (agent / subagentStatusLine only)
├── .mcp.json                 # MCP server definitions
├── .lsp.json                 # LSP server configurations
├── scripts/                  # Hook and utility scripts
│   ├── format-code.py
│   └── deploy.js
├── LICENSE
└── CHANGELOG.md
```

A plugin that ships exactly one skill can place `SKILL.md` directly at the plugin root (no `skills/` dir needed) — Claude Code loads it as a single skill using the frontmatter `name` field (v2.1.142+).

### Path variables for bundled files

Because marketplace plugins are *copied to a cache* (not run in place), you can't hardcode paths. Three substitution variables are available inline in skill/agent content, hook/monitor commands, and MCP/LSP configs, and are also exported as env vars to subprocesses:

| Variable | Resolves to | Use for |
|---|---|---|
| `${CLAUDE_PLUGIN_ROOT}` | Absolute path to the plugin's install directory | Referencing bundled scripts, binaries, config. **Changes on every update — do not write state here.** |
| `${CLAUDE_PLUGIN_DATA}` | `~/.claude/plugins/data/{id}/` — persistent, survives updates | `node_modules`, venvs, caches, generated state |
| `${CLAUDE_PROJECT_DIR}` | The project root Claude Code was launched in | Project-local scripts/config |

In shell-form hook and monitor commands, wrap the variable in double quotes to handle spaces: `"${CLAUDE_PLUGIN_ROOT}"/scripts/format-code.sh`.

---

## Bundling each component type

### Hooks (`hooks/hooks.json`)

Same format as user-defined hooks in `settings.json` — copy the `hooks` object verbatim. Plugin hooks respond to the full set of lifecycle events (`SessionStart`, `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop`, `SubagentStart`, `FileChanged`, `PreCompact`, and many more — see [hooks.md](../claude-code/hooks.md)) and support hook types `command`, `http`, `mcp_tool`, `prompt`, and `agent`.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

### MCP servers (`.mcp.json`)

Standard MCP config. Plugin servers start automatically when the plugin is enabled and appear as normal MCP tools.

```json
{
  "mcpServers": {
    "plugin-database": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": { "DB_PATH": "${CLAUDE_PLUGIN_ROOT}/data" }
    }
  }
}
```

### LSP servers (`.lsp.json`)

Give Claude real-time diagnostics and code navigation. Requires the language-server binary already on `PATH`.

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": { ".go": "go" }
  }
}
```

**Required fields** are `command` and `extensionToLanguage`; optional fields include `args`, `transport` (`stdio` default / `socket`), `env`, `initializationOptions`, `settings`, `workspaceFolder`, `startupTimeout`, `maxRestarts`, and `diagnostics` (default `true` — set `false` to keep navigation but suppress automatic diagnostic injection).

The official marketplace ships pre-built LSP plugins for 11 languages — install these rather than building your own unless your language isn't covered:

| Language | Plugin | Binary required |
|---|---|---|
| C/C++ | `clangd-lsp` | `clangd` |
| C# | `csharp-lsp` | `csharp-ls` |
| Go | `gopls-lsp` | `gopls` |
| Java | `jdtls-lsp` | `jdtls` |
| Kotlin | `kotlin-lsp` | `kotlin-language-server` |
| Lua | `lua-lsp` | `lua-language-server` |
| PHP | `php-lsp` | `intelephense` |
| Python | `pyright-lsp` | `pyright-langserver` |
| Rust | `rust-analyzer-lsp` | `rust-analyzer` |
| Swift | `swift-lsp` | `sourcekit-lsp` |
| TypeScript | `typescript-lsp` | `typescript-language-server` |

The language-server binary must already be on `PATH`; otherwise the plugin surfaces `Executable not found in $PATH` in the `/plugin` **Errors** tab. Press **Ctrl+O** when the "diagnostics found" indicator appears to view diagnostics inline.

### Background monitors (`monitors/monitors.json`) — experimental

An array of long-running watch commands; each stdout line is delivered to Claude as a notification. Requires v2.1.105+.

```json
[
  {
    "name": "error-log",
    "command": "tail -F ./logs/error.log",
    "description": "Application error log",
    "when": "on-skill-invoke:debug"
  }
]
```

`when` is `"always"` (default) or `"on-skill-invoke:<skill-name>"`.

---

## Marketplaces: the distribution layer

A **marketplace** is a catalog (a `.claude-plugin/marketplace.json` file in a git repo) that lists one or more plugins and where to fetch each. Using a marketplace is two steps, like adding an app store: **add the marketplace** (registers the catalog), then **install individual plugins** from it.

### `marketplace.json` schema

```json
{
  "name": "company-tools",
  "owner": {
    "name": "DevTools Team",
    "email": "devtools@example.com"
  },
  "plugins": [
    {
      "name": "code-formatter",
      "source": "./plugins/formatter",
      "description": "Automatic code formatting on save",
      "version": "2.1.0",
      "author": { "name": "DevTools Team" }
    },
    {
      "name": "deployment-tools",
      "source": {
        "source": "github",
        "repo": "company/deploy-plugin"
      },
      "description": "Deployment automation tools"
    }
  ]
}
```

**Required top-level fields:** `name` (kebab-case; public-facing, users see it as `plugin@marketplace`), `owner` (`{ name, email? }`), and `plugins` (array). Optional: `$schema`, `description`, `version`, `metadata.pluginRoot` (base dir prepended to relative sources), `allowCrossMarketplaceDependenciesOn`.

> **Reserved names:** These marketplace names are reserved for official Anthropic use and rejected for third-party marketplaces: `claude-code-marketplace`, `claude-code-plugins`, `claude-plugins-official`, `claude-plugins-community`, `claude-community`, `anthropic-marketplace`, `anthropic-plugins`, `agent-skills`, `anthropic-agent-skills`, `knowledge-work-plugins`, `life-sciences`, `claude-for-legal`, `claude-for-financial-services`, `financial-services-plugins`. Names that *impersonate* official marketplaces (e.g. `official-claude-plugins`, `anthropic-tools-v2`) are also blocked.

**Per-plugin entry** requires `name` + `source`. It can also carry any `plugin.json` field (`description`, `version`, `author`, `homepage`, `repository`, `license`, `keywords`, `commands`, `agents`, `hooks`, `mcpServers`, `lspServers`, `skills`) plus marketplace-only fields:

| Field | Notes |
|---|---|
| `category` | Plugin category for organization. |
| `tags` | Tags for searchability. |
| `strict` | Whether `plugin.json` is authoritative (default `true`; see below). |
| `displayName` | Human-readable name in UI. *Requires v2.1.143+.* |
| `relevance` | Signals telling Claude Code when to suggest this plugin. **Takes effect only for marketplaces an admin allowlists via `pluginSuggestionMarketplaces` in managed settings.** *Requires v2.1.152+.* |
| `defaultEnabled` | Whether enabled after install (default `true`). **Takes precedence over `plugin.json`'s `defaultEnabled`.** *Requires v2.1.154+.* |

> When both `version` and the manifest are present and both set `version`, the `plugin.json` value wins **silently** — avoid setting `version` in both places, or a stale manifest can mask the marketplace value.

### Plugin source types

The `source` field of each plugin entry tells Claude Code where to fetch that plugin:

| Source | Form | Fields | Notes |
|---|---|---|---|
| Relative path | `"./my-plugin"` | — | Local dir in the marketplace repo; must start with `./`; resolved from marketplace root (the dir containing `.claude-plugin/`), **not** from `.claude-plugin/`. Only works for git-added marketplaces, not URL-added ones |
| `github` | object | `repo`, `ref?`, `sha?` | `repo` in `owner/repo` form |
| `url` | object | `url`, `ref?`, `sha?` | Any git URL (`https://` or `git@`); `.git` suffix optional, so Azure DevOps / AWS CodeCommit URLs work. GitLab, Bitbucket, self-hosted |
| `git-subdir` | object | `url`, `path`, `ref?`, `sha?` | Subdir of a monorepo; sparse partial clone. `url` also accepts `owner/repo` shorthand and SSH URLs |
| `npm` | object | `package`, `version?`, `registry?` | Installed via `npm install`; `version` accepts ranges (`^2.0.0`, `~1.5.0`); `registry` for private registries |

When both `ref` and `sha` are set, `sha` is the effective pin: Claude Code checks out that commit directly, so installation succeeds even if the `ref` branch/tag was deleted upstream — as long as the commit is still reachable. (Servers that can't fetch by SHA, e.g. AWS CodeCommit, still require the `ref` to exist with the commit reachable from it.) `sha` must be the full 40-character commit hash.

```json
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo",
    "ref": "v2.0.0",
    "sha": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
  }
}
```

> **Marketplace source ≠ plugin source.** The *marketplace source* (where the `marketplace.json` itself lives, set via `/plugin marketplace add`) supports `ref` but not `sha`. The *plugin source* (inside each entry) supports both. They can point at entirely different repos.

### Strict mode

The `strict` field (default `true`) controls whether `plugin.json` is the authority for component definitions:

| Value | Behavior |
|---|---|
| `true` (default) | `plugin.json` is authoritative; the marketplace entry can *supplement* it (both merge). The plugin needs its own `plugin.json`. |
| `false` | The marketplace entry is the *entire* definition; the plugin needs no `plugin.json`. If the plugin also declares components, that's a conflict and it fails to load. |

Use `strict: false` when a marketplace operator wants full control over how a plugin's raw files are exposed.

---

## The `/plugin` command and `claude plugin` CLI

### Interactive `/plugin` manager

Run `/plugin` inside Claude Code to open a tabbed interface (cycle with **Tab** / **Shift+Tab**):

- **Discover** — browse plugins from all your marketplaces. Selecting a plugin shows a **Context cost** estimate (v2.1.143+), a **Last updated** date (v2.1.144+), and a **Will install** inventory of commands/agents/skills/hooks/MCP/LSP servers (v2.1.145+).
- **Installed** — view, favorite (`f`), enable, disable, or uninstall; filter by typing; grouped by scope with errors first, disabled folded at the bottom. A **Not used recently** group (v2.1.187+) surfaces stale plugins.
- **Marketplaces** — add, remove, update; toggle per-marketplace auto-update.
- **Errors** — plugin load errors (e.g. `Executable not found in $PATH` for an LSP binary).

Shortcuts: `/plugin market` aliases `/plugin marketplace`; `rm` aliases `remove`.

### Managing marketplaces

```shell
# Add — GitHub owner/repo shorthand
/plugin marketplace add anthropics/claude-code

# Add — pin to a branch/tag (@ref for GitHub, #ref for git URLs)
/plugin marketplace add acme-corp/claude-plugins@v2.0
/plugin marketplace add https://gitlab.com/company/plugins.git#v1.0.0

# Add — git URL (include .git so it clones rather than treating it as a raw file)
/plugin marketplace add git@gitlab.com:company/plugins.git

# Add — local path (for testing) or a remote marketplace.json
/plugin marketplace add ./my-marketplace
/plugin marketplace add https://example.com/marketplace.json

# List / update / remove
/plugin marketplace list
/plugin marketplace update claude-plugins-official   # omit name to update all
/plugin marketplace remove company-tools
```

> Removing a marketplace from its **last** scope also uninstalls every plugin you installed from it. To refresh without losing plugins, use `update`, not `remove`.

### Managing plugins

```shell
# Install (defaults to user scope)
/plugin install commit-commands@claude-code-plugins

# Enable / disable without uninstalling
/plugin disable plugin-name@marketplace-name
/plugin enable plugin-name@marketplace-name

# Completely remove
/plugin uninstall plugin-name@marketplace-name

# List installed; filter by state
/plugin list
/plugin list --enabled
/plugin list --disabled

# Apply install/enable/disable changes mid-session without restarting
/reload-plugins        # add --force if it warns about cache invalidation (v2.1.163+)
```

> `/reload-plugins` reloads all active plugins plus their skills, agents, hooks, and plugin MCP/LSP servers, and prints counts for each. It has a token cost on the next request: newly loaded components are appended to the conversation while existing history reads from the prompt cache. A plugin contributing MCP servers whose tools aren't deferred by tool search invalidates the cache (the next request re-reads the whole conversation); in that case `/reload-plugins` warns and **does not** apply — pass `--force` to override (v2.1.163+).

### Non-interactive `claude plugin` subcommands

Every interactive action has a scriptable CLI equivalent (useful for CI, Dockerfiles, dotfiles):

| Command | Purpose |
|---|---|
| `claude plugin init <name> [--with skills agents hooks mcp lsp output-style channel]` | Scaffold a new plugin under `~/.claude/skills/<name>/`, auto-loading as `<name>@skills-dir` |
| `claude plugin install <plugin> [-s user\|project\|local]` | Install; `--scope project` writes to `.claude/settings.json` |
| `claude plugin uninstall <plugin> [--keep-data] [--prune] [-y]` | Remove (aliases `remove`, `rm`); deletes the data dir unless `--keep-data` |
| `claude plugin enable / disable <plugin> [-s scope]` | Toggle without uninstalling |
| `claude plugin update <plugin> [-s scope]` | Update to latest |
| `claude plugin prune [--dry-run] [-y]` | Remove orphaned auto-installed dependencies (alias `autoremove`; v2.1.121+) |
| `claude plugin list [--json] [--available]` | List installed plugins |
| `claude plugin details <name>` | Component inventory + projected always-on / on-invoke token cost |
| `claude plugin validate [path] [--strict]` | Validate `marketplace.json` or a plugin's `plugin.json` + frontmatter |
| `claude plugin marketplace add/list/remove/update ...` | Marketplace management (mirrors `/plugin marketplace`) |
| `claude plugin tag [--push] [--dry-run]` | Create a release git tag from inside the plugin folder |

`claude plugin marketplace add` adds `--scope <user|project|local>` and `--sparse <paths...>` (limit checkout for monorepos).

### Local development & testing

```bash
# Load a plugin directly without installing (repeat the flag for many)
claude --plugin-dir ./my-first-plugin
claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two

# Load a zipped plugin (v2.1.128+) or a hosted zip artifact
claude --plugin-dir ./my-plugin.zip
claude --plugin-url https://example.com/my-plugin.zip
```

A `--plugin-dir` plugin shadows an installed plugin of the same name for that session (great for testing edits without uninstalling). Run `/reload-plugins` after edits to pick up changes to skills, agents, hooks, and plugin MCP/LSP servers.

---

## Installation scopes

| Scope | Settings file | Use case |
|---|---|---|
| `user` | `~/.claude/settings.json` | Personal, across all projects (**default**) |
| `project` | `.claude/settings.json` | Team plugins shared via version control |
| `local` | `.claude/settings.local.json` | Project-specific, gitignored |
| `managed` | [Managed settings](../claude-code/settings.md) | Admin-installed, read-only (update only) |

Installed plugins are recorded in `enabledPlugins` in the chosen scope's settings file. Plugins use the same scope system as the rest of Claude Code configuration — see [settings.md](../claude-code/settings.md).

---

## Worked example: author, publish, and install a plugin

This end-to-end example creates a marketplace containing one plugin that ships **one skill, one agent, and one hook**, then installs it.

### 1. Scaffold the structure

```bash
mkdir -p my-marketplace/.claude-plugin
mkdir -p my-marketplace/plugins/quality-kit/.claude-plugin
mkdir -p my-marketplace/plugins/quality-kit/skills/review
mkdir -p my-marketplace/plugins/quality-kit/agents
mkdir -p my-marketplace/plugins/quality-kit/hooks
mkdir -p my-marketplace/plugins/quality-kit/scripts
```

### 2. The plugin manifest — `plugins/quality-kit/.claude-plugin/plugin.json`

```json
{
  "name": "quality-kit",
  "description": "A skill, an agent, and a lint-on-save hook",
  "version": "1.0.0",
  "author": { "name": "Your Name", "email": "you@example.com" },
  "license": "MIT"
}
```

### 3. A skill — `plugins/quality-kit/skills/review/SKILL.md`

```markdown
---
description: Review selected code or recent changes for bugs, security, and performance
disable-model-invocation: true
---

Review the code I've selected or the recent changes for:
- Potential bugs or edge cases
- Security concerns
- Performance issues

Be concise and actionable.
```

Invoked as `/quality-kit:review`.

### 4. An agent — `plugins/quality-kit/agents/security-reviewer.md`

```markdown
---
name: security-reviewer
description: Audits diffs for injection, secrets, and auth flaws. Invoke before merging.
model: sonnet
effort: medium
disallowedTools: Write, Edit
---

You are a security reviewer. Examine the diff for injection vulnerabilities,
hardcoded secrets, broken auth checks, and unsafe deserialization. Report
findings ranked by severity with file:line references.
```

> Plugin agents support `name`, `description`, `model`, `effort`, `maxTurns`, `tools`, `disallowedTools`, `skills`, `memory`, `background`, and `isolation` (only `"worktree"`). For security, `hooks`, `mcpServers`, and `permissionMode` are **not** allowed in plugin-shipped agents.

### 5. A hook — `plugins/quality-kit/hooks/hooks.json`

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs \"${CLAUDE_PLUGIN_ROOT}\"/scripts/lint.sh"
          }
        ]
      }
    ]
  }
}
```

`scripts/lint.sh` runs your linter; make it executable with `chmod +x`.

### 6. The marketplace catalog — `my-marketplace/.claude-plugin/marketplace.json`

```json
{
  "name": "my-plugins",
  "owner": { "name": "Your Name" },
  "plugins": [
    {
      "name": "quality-kit",
      "source": "./plugins/quality-kit",
      "description": "A skill, an agent, and a lint-on-save hook"
    }
  ]
}
```

### 7. Validate, publish, install

```bash
# Validate locally (the community review pipeline runs the same check)
claude plugin validate ./my-marketplace
claude plugin validate ./my-marketplace/plugins/quality-kit

# Publish: push the marketplace repo to GitHub
cd my-marketplace && git init && git add -A && git commit -m "Add quality-kit" \
  && git remote add origin git@github.com:you/my-plugins.git && git push -u origin main

# A user installs it
/plugin marketplace add you/my-plugins
/plugin install quality-kit@my-plugins
/reload-plugins

# Use it
/quality-kit:review
```

---

## Enterprise: pre-install, allowlist, and lock down

Admins control plugins through **managed settings** and project `.claude/settings.json`. See [settings.md](../claude-code/settings.md) for the full reference.

### Require a marketplace for a team

Add to a project's `.claude/settings.json`; collaborators are prompted to install when they trust the folder:

```json
{
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": { "source": "github", "repo": "your-org/claude-plugins" },
      "autoUpdate": true
    }
  },
  "enabledPlugins": {
    "code-formatter@company-tools": true,
    "deployment-tools@company-tools": true
  }
}
```

### Restrict which marketplaces users may add

`strictKnownMarketplaces` in **managed** settings (cannot be overridden by users/projects):

```json
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/approved-plugins" },
    { "source": "hostPattern", "hostPattern": "^github\\.example\\.com$" }
  ]
}
```

- Undefined (default): no restriction.
- `[]` (empty array): complete lockdown — no new marketplaces.
- List of sources: only allowlisted marketplaces. Matching is **exact** for `github`/`url` sources (no normalization — a trailing slash, `.git` suffix, or `ssh://` vs `https://` form are all treated as different values; prefer a `hostPattern` when several URL forms exist). For `github` the `repo` is required and any specified `ref`/`path` must also match; for `url` the full URL must match. `hostPattern` and `pathPattern` are **regex** on the host or filesystem path respectively. Pair with `extraKnownMarketplaces` to also auto-register them — `strictKnownMarketplaces` only *restricts*, it does not register.

Restrictions are checked **before any network or filesystem operation**, on add and on install/update/refresh/auto-update. A marketplace added before the policy existed is blocked from future installs/updates if its source no longer matches. The same enforcement applies to `blockedMarketplaces` (a managed-settings denylist). To block `@skills-dir` plugins scaffolded by `claude plugin init`, add `{"source": "skills-dir"}` to `blockedMarketplaces`.

### Suggest (not require) plugins for a directory

`pluginSuggestionMarketplaces` (managed settings, v2.1.154+) allowlists marketplaces whose plugins may be *suggested* based on the current working directory. Only for marketplaces in this list does a plugin entry's `relevance` field (v2.1.152+) take effect: relevant plugins are pinned at the top of the **Discover** tab with a **suggested for this directory** label. This is a softer control than requiring/locking down — it surfaces plugins without auto-installing them.

### Pre-populate plugins for containers / CI

Build a seed directory (mirrors `~/.claude/plugins`) and point `CLAUDE_CODE_PLUGIN_SEED_DIR` at it so Claude Code starts with marketplaces and plugins already available, no runtime cloning:

```bash
CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed claude plugin marketplace add your-org/plugins
CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed claude plugin install my-tool@your-plugins
# then at runtime:
export CLAUDE_CODE_PLUGIN_SEED_DIR=/opt/claude-seed
```

Seed marketplaces are read-only (auto-update disabled), take precedence over user config, and resolve by probing `marketplaces/<name>/` at runtime so they work even when mounted at a different path.

### Relevant environment variables

| Variable | Effect |
|---|---|
| `CLAUDE_CODE_PLUGIN_SEED_DIR` | Pre-populated, read-only plugins dir (`:`/`;` separated for layering) |
| `CLAUDE_CODE_PLUGIN_CACHE_DIR` | Override the install target during a seed build |
| `CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE=1` | Keep the cached clone when `git pull` fails (offline/airgapped) |
| `CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS` | Raise the default 120 s git timeout for large repos |
| `DISABLE_AUTOUPDATER` + `FORCE_AUTOUPDATE_PLUGINS=1` | Disable Claude Code auto-update but keep plugin auto-updates |
| `GITHUB_TOKEN`/`GH_TOKEN`, `GITLAB_TOKEN`/`GL_TOKEN`, `BITBUCKET_TOKEN` | Auth for background auto-updates from private marketplaces |

---

## Caching, versioning, and skills-directory plugins

### Caching

Marketplace plugins are **copied** to the local cache at `~/.claude/plugins/cache` (one directory per version) rather than run in place — for security and verifiability. Consequences:

- A plugin **cannot reference files outside its own directory** (`../shared-utils` won't resolve post-install). Share files within a marketplace using symlinks (preserved if they resolve inside the plugin; dereferenced if elsewhere in the same marketplace; skipped if outside, for security).
- Old version directories are kept ~7 days after update/uninstall (so concurrent sessions don't break), then auto-removed. Glob/Grep skip orphaned dirs.
- Persistent state belongs in `${CLAUDE_PLUGIN_DATA}` (`~/.claude/plugins/data/{id}/`), which survives updates and is deleted on final-scope uninstall unless you pass `--keep-data`.

### Version resolution

Claude Code resolves a plugin's version from the first of: (1) `version` in `plugin.json`, (2) `version` in the marketplace entry, (3) the git commit SHA of the source, (4) `unknown` for npm/local sources. **If you set `version`, you must bump it on every release** — pushing commits without bumping does nothing for existing users. Omit `version` to use the commit SHA so every commit is an update (best for internal/fast-moving plugins). Release channels ("stable"/"latest") are built by pointing two marketplaces at different `ref`s/SHAs of the same repo and assigning each to a user group via managed settings.

### Skills-directory plugins (`@skills-dir`)

Any folder under a skills directory that contains a `.claude-plugin/plugin.json` loads automatically as `<name>@skills-dir` — no marketplace, no install. `~/.claude/skills/` loads in every project; `<cwd>/.claude/skills/` loads only after you trust the workspace (and project-scope `@skills-dir` plugins have MCP/LSP/monitor restrictions). Scaffold one with `claude plugin init <name>`. To remove, delete the folder or `claude plugin disable <name>@skills-dir`.

---

## The official and community marketplaces

| Marketplace | Name | How to use |
|---|---|---|
| **Official** (Anthropic-curated) | `claude-plugins-official` | Auto-registered on first interactive launch. `/plugin install github@claude-plugins-official`. Browse at [claude.com/plugins](https://claude.com/plugins). Inclusion is at Anthropic's discretion. |
| **Community** (third-party, screened) | `claude-community` (repo `anthropics/claude-plugins-community`) | Add manually: `/plugin marketplace add anthropics/claude-plugins-community`, then `/plugin install <name>@claude-community`. Submissions pass automated validation + safety screening, pinned to a commit SHA. |
| **Demo / examples** | `claude-code-plugins` (repo `anthropics/claude-code`) | `/plugin marketplace add anthropics/claude-code` |

The official marketplace ships several categories of notable bundles:

- **Code intelligence (LSP):** the 11 `*-lsp` plugins listed above (`clangd-lsp`, `csharp-lsp`, `gopls-lsp`, `jdtls-lsp`, `kotlin-lsp`, `lua-lsp`, `php-lsp`, `pyright-lsp`, `rust-analyzer-lsp`, `swift-lsp`, `typescript-lsp`).
- **External integrations (MCP):** source control (`github`, `gitlab`); project management (`atlassian` for Jira/Confluence, `asana`, `linear`, `notion`); design (`figma`); infrastructure (`vercel`, `firebase`, `supabase`); communication (`slack`); monitoring (`sentry`).
- **Automatic security review:** `security-guidance` reviews each change Claude makes for common vulnerabilities and instructs Claude to fix them in-session.
- **Development workflows:** `commit-commands` (commit/push/PR), `pr-review-toolkit`, `agent-sdk-dev`, `plugin-dev`.
- **Output styles:** `explanatory-output-style`, `learning-output-style`.

**Submitting to the community marketplace** (not the official one — Anthropic curates that separately and there is no application process):

- **claude.ai form:** [claude.ai/admin-settings/directory/submissions/plugins/new](https://claude.ai/admin-settings/directory/submissions/plugins/new) — requires a **Team or Enterprise organization with directory-management access** (org Owners have it by default).
- **Console form:** [platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit) — the fallback for individual authors who aren't in a Team/Enterprise org.

Run `claude plugin validate` locally first (the review pipeline runs the same check plus automated safety screening). Approved plugins are pinned to a specific commit SHA in the catalog; CI bumps the pin as you push, and the public catalog syncs **nightly**, so expect a delay between approval and appearance in `marketplace.json`. If Anthropic lists your plugin in the official marketplace, your CLI can prompt users to install it — see plugin hints / [Recommend your plugin](https://code.claude.com/docs/en/plugin-hints).

---

## Relationship to other extension primitives

Plugins are the **packaging and distribution layer** that wraps the individual primitives — they don't replace them. A plugin is "a box you put skills/agents/hooks/MCP servers in so they travel together and version together."

- **[Skills](./skills.md)** — model-invoked capabilities (`SKILL.md`). Inside a plugin they become namespaced `/plugin:skill`. A skill can also be a standalone `@skills-dir` plugin.
- **[Subagents](../claude-code/subagents.md)** — specialized agents in `agents/`. Plugin agents have a restricted frontmatter (no `hooks`/`mcpServers`/`permissionMode`).
- **[Hooks](../claude-code/hooks.md)** — lifecycle event handlers; plugin hooks use the same `hooks.json` format and event set as user hooks.
- **[Slash commands](../claude-code/slash-commands.md)** — `/plugin`, `/reload-plugins`, and the namespaced commands a plugin contributes.
- **[MCP](./mcp.md)** — plugins are the cleanest way to ship a pre-configured MCP server with no manual setup.

---

## Related pages

- [./skills.md](./skills.md) — Agent Skills, `SKILL.md`, progressive disclosure (a plugin's most common payload)
- [./mcp.md](./mcp.md) — Model Context Protocol servers bundled via `.mcp.json`
- [../claude-code/subagents.md](../claude-code/subagents.md) — Subagent definitions in `agents/`
- [../claude-code/hooks.md](../claude-code/hooks.md) — Hook events and `hooks.json` format
- [../claude-code/slash-commands.md](../claude-code/slash-commands.md) — `/plugin`, `/reload-plugins`, namespaced commands
- [../claude-code/settings.md](../claude-code/settings.md) — `enabledPlugins`, `extraKnownMarketplaces`, `strictKnownMarketplaces`, `pluginSuggestionMarketplaces`, `blockedMarketplaces`, scopes
- [../platform/claude-code.md](../platform/claude-code.md) — Claude Code overview and surfaces (CLI/IDE/desktop/web)

## Open questions / to verify

- Exact minimum Claude Code version where the plugin system first shipped as GA. The docs gate many *sub-features* by version (v2.1.105 monitors, v2.1.121 prune, v2.1.128 zip, v2.1.142 single-skill, v2.1.143 displayName/Context-cost, v2.1.144 Last-updated, v2.1.145 Will-install, v2.1.152 relevance, v2.1.154 defaultEnabled/pluginSuggestionMarketplaces, v2.1.163 reload --force, v2.1.187 Not-used-recently) but do **not** state the original GA version. **WARN: verify.**
- Whether `experimental.themes` / `experimental.monitors` have moved out of the `experimental` key. Verified (2026-06-26): docs still place both under `experimental`, and state declaring them at the top level still works but warns, with a future release set to *require* `experimental.*`. So the migration is in progress — re-check per release.
- The precise current contents of `claude-plugins-official` change over time; the catalog above reflects the docs snapshot (11 LSP plugins + the listed MCP/workflow/output-style bundles) and should be re-checked at [claude.com/plugins](https://claude.com/plugins).
- Claude.ai submission plan gating confirmed (2026-06-26): the claude.ai form **requires a Team or Enterprise org with directory-management access** (Owners have it by default); individual authors outside such an org use the Console form. Re-verify if Anthropic changes the directory program.

## Sources

- [Create plugins — Claude Code Docs](https://code.claude.com/docs/en/plugins)
- [Plugins reference — Claude Code Docs](https://code.claude.com/docs/en/plugins-reference)
- [Create and distribute a plugin marketplace — Claude Code Docs](https://code.claude.com/docs/en/plugin-marketplaces)
- [Discover and install prebuilt plugins — Claude Code Docs](https://code.claude.com/docs/en/discover-plugins)
- [anthropics/claude-plugins-official (GitHub)](https://github.com/anthropics/claude-plugins-official)
- [anthropics/claude-plugins-community (GitHub)](https://github.com/anthropics/claude-plugins-community)
