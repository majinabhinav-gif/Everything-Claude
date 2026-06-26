---
type: Extension System
title: Agent Skills
description: Agent Skills are on-demand folders of instructions, scripts, and resources (a SKILL.md plus optional files) that teach Claude a repeatable capability, loaded progressively across Claude apps, Claude Code, and the API.
domain: capabilities
tags: [skills, SKILL.md, progressive-disclosure, agent-skills, extensions, code-execution, pdf, docx, pptx, xlsx, claude-code, api]
related: [plugins, claude-code-overview, connectors, capabilities-and-modes, mcp, subagents, projects, slash-commands]
resource: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
  - https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
  - https://platform.claude.com/docs/en/build-with-claude/skills-guide
  - https://code.claude.com/docs/en/skills
  - https://support.claude.com/en/articles/12512198-creating-custom-skills
  - https://support.claude.com/en/articles/12512176-what-are-skills
  - https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
  - https://github.com/anthropics/skills
  - https://agentskills.io/specification
verified: 2026-06-26
---

# Agent Skills

Agent Skills are reusable, filesystem-based packages that give Claude domain-specific expertise: a folder containing a `SKILL.md` file (instructions + YAML metadata) plus optional scripts, reference docs, and assets. The key design idea is **progressive disclosure** — Claude reads only a short name + description for each installed Skill at startup, and loads the full body (and any linked files or scripts) only when a request actually matches. This lets you install many Skills with almost no context cost, transforming a general-purpose agent into a specialist on demand.

Skills work the same way conceptually everywhere, but where you put them, how you install them, and what runtime they get differ by surface (Claude apps, Claude Code, Claude API). This page is the exhaustive reference.

> **At a glance**
>
> - **What it is:** An open, folder-based extension format. Entry point is `SKILL.md` (YAML frontmatter `name`/`description` + Markdown instructions); optional bundled files (`scripts/`, `references/`, `assets/`) loaded on demand.
> - **Where you find it:** claude.ai → Skills settings (upload zips — see the **menu-path note** below); **Claude Code** → `.claude/skills/<name>/SKILL.md` and `/skills`; **Claude API** → `container.skills` parameter + `/v1/skills` endpoints. Pre-built Anthropic Skills (pptx/xlsx/docx/pdf) run automatically in claude.ai and via `skill_id` in the API.
> - **Who can use it by plan:** Pre-built Skills: built into claude.ai document creation for everyone. Custom Skills on claude.ai require **code execution enabled**; plan eligibility is documented inconsistently — the platform **overview** says **Pro, Max, Team, Enterprise** (not Free); the **help center** says **Free, Pro, Max, Team, Enterprise**. WARN: both are official as of 2026-06; treat Free as unconfirmed. Claude Code: anyone (filesystem-based, no upload). API: any workspace with the beta headers (workspace-wide sharing).
> - **Status:** Generally available across surfaces in 2026; API access is still behind beta headers (`skills-2025-10-02`). Open standard published at agentskills.io; Anthropic open-sources Skills at github.com/anthropics/skills. Changing fast — Claude Code recently **merged custom slash commands into Skills**.

---

## What a Skill is (and isn't)

A Skill packages three kinds of content into one directory, "organized like an onboarding guide you'd create for a new team member":

| Content type | Examples | How Claude uses it |
| --- | --- | --- |
| **Instructions** | `SKILL.md` body, `FORMS.md`, `reference.md` | Read into context when relevant |
| **Code** | `scripts/fill_form.py`, `validate.sh` | Executed via bash; only the **output** enters context (the code never does) |
| **Resources** | DB schemas, API docs, templates, sample data | Read on demand for factual lookup |

**Skills vs. other mechanisms:**

- **vs. Prompts:** A prompt is conversation-level and one-off. A Skill is created once and loads automatically across many conversations.
- **vs. CLAUDE.md / Projects custom instructions:** CLAUDE.md and project instructions are *always* in context. A Skill's body costs almost nothing until triggered — ideal for long procedures and reference material. (See [./projects.md](./projects.md).)
- **vs. Connectors / MCP:** Connectors/MCP add *tools* (live API calls). Skills add *know-how* — instructions and code that may themselves call those tools. They compose: a Skill can tell Claude to use a specific MCP tool. (See [./connectors.md](./connectors.md), [./mcp.md](./mcp.md).)
- **vs. Plugins:** A Plugin is a distribution bundle that can contain Skills *plus* hooks, agents, MCP servers, and commands. A Skill is one capability. (See [./plugins.md](./plugins.md).)

---

## Progressive disclosure: the three levels

This is the central concept. Claude loads Skill content in stages, so only relevant content occupies the context window.

| Level | When loaded | Token cost | Content |
| --- | --- | --- | --- |
| **Level 1 — Metadata** | Always (at startup) | ~100 tokens per Skill | `name` + `description` from YAML frontmatter, injected into the system prompt |
| **Level 2 — Instructions** | When the Skill is triggered | Under ~5k tokens | The `SKILL.md` body (workflows, guidance), read from disk via bash |
| **Level 3+ — Resources & code** | As needed | Effectively unlimited | Bundled files read on demand; scripts executed via bash (output only) |

Worked example — a PDF Skill loading:

1. **Startup:** System prompt includes `PDF Processing — Extract text and tables from PDF files, fill forms, merge documents.`
2. **User request:** "Extract the text from this PDF and summarize it."
3. **Claude invokes:** `bash: read pdf-skill/SKILL.md` → instructions enter context.
4. **Claude determines:** form filling isn't needed → `FORMS.md` is *not* read.
5. **Claude executes:** completes the task from `SKILL.md` alone.

Because files don't consume context until accessed, a Skill can bundle dozens of reference files, large datasets, or comprehensive API docs with **no context penalty for the parts you don't use**. When Claude runs `validate_form.py`, the script's *code* never loads — only `Validation passed` or specific errors. This is what makes scripts more efficient than having Claude regenerate equivalent code each time.

---

## The SKILL.md file and frontmatter

Every Skill requires a `SKILL.md` with YAML frontmatter between `---` markers, followed by a Markdown body.

```yaml
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---

# PDF Processing

## Quick start
Use pdfplumber to extract text from PDFs:

```python
import pdfplumber
with pdfplumber.open("document.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

For advanced form filling, see [FORMS.md](FORMS.md).
```

### Core (cross-surface) frontmatter fields

These are the fields the **base Agent Skills format** validates (claude.ai + API + the open standard):

| Field | Required | Constraints |
| --- | --- | --- |
| `name` | Yes | Max **64 chars**; lowercase letters, numbers, hyphens only; no XML tags; cannot contain reserved words **`anthropic`** or **`claude`** |
| `description` | Yes | Non-empty; max **1024 chars**; no XML tags; should state **what** the Skill does **and when** to use it; write in **third person** |

**Optional fields (open standard).** The [agentskills.io specification](https://agentskills.io/specification) defines three optional cross-tool keys beyond `name`/`description`:

| Field | Purpose |
| --- | --- |
| `license` | SPDX identifier, e.g. `Apache-2.0`. |
| `metadata` | Arbitrary key-value map for author info, commonly `author` and `version`. |
| `compatibility` | Declares environment needs (intended product, required system packages, network access). Most Skills don't need it. |
| `allowed-tools` | **Experimental** in the open standard — a pre-approved tool list; support varies across agent implementations (Claude Code fully supports it; see below). |

```yaml
---
name: pdf-processing
description: Extract PDF text, fill forms, merge files. Use when working with PDFs.
license: Apache-2.0
metadata:
  author: example-org
  version: "1.0"
---
```

Note: the Claude **API uploader** formally validates only `name` and `description`; the optional keys are accepted/ignored per the open standard. Claude Code reads its own extended set (below) and ignores keys it doesn't recognize.

### Claude Code frontmatter (extended)

Claude Code follows the open Agent Skills standard but adds many fields. **All fields are optional; only `description` is recommended.**

| Field | Description |
| --- | --- |
| `name` | Display name in skill listings. **Defaults to the directory name** (does not change the command you type, except for a plugin-root `SKILL.md`). |
| `description` | What the skill does + when to use it. If omitted, uses the first paragraph of the body. Combined with `when_to_use`, truncated at **1,536 chars** in the listing. |
| `when_to_use` | Extra trigger phrases / example requests, appended to `description`; counts toward the 1,536-char cap. |
| `argument-hint` | Autocomplete hint, e.g. `[issue-number]` or `[filename] [format]`. |
| `arguments` | Named positional args for `$name` substitution. Space-separated string or YAML list. |
| `disable-model-invocation` | `true` = only **you** can run it (`/name`); Claude won't auto-load it. Also keeps it out of subagent preloads. Default `false`. |
| `user-invocable` | `false` = hide from the `/` menu (background knowledge only Claude uses). Default `true`. |
| `allowed-tools` | Tools Claude may use **without per-use approval** while the skill is active. Space/comma string or YAML list. Does **not** restrict the pool. |
| `disallowed-tools` | Tools **removed** from the pool while active (e.g. block `AskUserQuestion` in a background loop). Clears on your next message. |
| `model` | Model to use while active (same values as `/model`, or `inherit`). Reverts next prompt. |
| `effort` | Effort level while active: `low`, `medium`, `high`, `xhigh`, `max` (model-dependent). |
| `context` | `fork` = run the skill in a forked subagent context. |
| `agent` | Which subagent type to use when `context: fork` (e.g. `Explore`, `Plan`, `general-purpose`, or a custom agent). |
| `hooks` | Hooks scoped to this skill's lifecycle. See [../claude-code/hooks.md](../claude-code/hooks.md). |
| `paths` | Glob patterns limiting auto-activation to matching files. |
| `shell` | `bash` (default) or `powershell` for inline `` !`command` `` blocks (PowerShell needs `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`). |

> Note: the cross-surface base format uses `name` + `description` (+ the optional `license`/`metadata`/`compatibility`/`allowed-tools` from the open standard). Fields like `context`, `disable-model-invocation`, `when_to_use`, `effort`, and `paths` are **Claude Code extensions** — a Skill authored for the API won't honor them, and vice versa.

### Invocation control: who can run a skill, and when it loads

Two Claude Code fields gate invocation. The table shows the combined effect (default = both blank):

| Frontmatter | You can invoke (`/name`) | Claude can auto-invoke | When loaded into context |
| --- | --- | --- | --- |
| *(default)* | Yes | Yes | Description always in context; full body loads when invoked |
| `disable-model-invocation: true` | Yes | No | Description **not** in context; full body loads only when **you** invoke |
| `user-invocable: false` | No | Yes | Description always in context; full body loads when Claude invokes |

`user-invocable: false` only hides the `/` menu entry — it does **not** block the `Skill` tool. To stop Claude invoking a skill programmatically, use `disable-model-invocation: true` (which also keeps it out of [subagent](../claude-code/subagents.md) preloads).

### Dynamic context injection (Claude Code)

Claude Code preprocesses `SKILL.md` before Claude sees it, expanding inline shell and substitution placeholders. This is **preprocessing, not tool calls** — Claude only sees the final rendered text.

- **Inline shell:** `` !`<command>` `` runs the command and replaces the placeholder with its stdout. Only recognized when `!` starts a line or follows whitespace (so `` KEY=!`cmd` `` is left literal). Substitution runs **once**; output is not re-scanned for further placeholders.
- **Fenced shell block** (multi-line):

  ````markdown
  ```!
  node --version
  git status --short
  ```
  ````
- **Shell selection:** the `shell` frontmatter field picks `bash` (default) or `powershell` (PowerShell needs `CLAUDE_CODE_USE_POWERSHELL_TOOL=1`).
- **Disable:** set `disableSkillShellExecution: true` (best in managed settings) to replace each command with `[shell command execution disabled by policy]`. Bundled and managed skills are unaffected.
- **Deeper reasoning:** include the literal word `ultrathink` anywhere in skill content to request extended reasoning when the skill runs.

### String substitutions (Claude Code)

| Variable | Expands to |
| --- | --- |
| `$ARGUMENTS` | All arguments as typed. If absent from the body, Claude Code appends `ARGUMENTS: <value>`. |
| `$ARGUMENTS[N]` / `$N` | The Nth argument (0-based); `$0` is the first. Shell-style quoting — wrap multi-word args in quotes. |
| `$name` | A named arg declared in the `arguments` frontmatter list (positional mapping). |
| `${CLAUDE_SESSION_ID}` | Current session ID (logging, session-scoped files). |
| `${CLAUDE_EFFORT}` | Active effort: `low`/`medium`/`high`/`xhigh`/`max` (ultracode reports as `xhigh`). |
| `${CLAUDE_SKILL_DIR}` | Directory holding this `SKILL.md` (for plugin skills, the skill subdir, not the plugin root). Use it to reference bundled scripts regardless of cwd. |

Escape a literal `$` before a digit/`ARGUMENTS`/declared name with a single backslash (`\$1.00`).

### Skill content lifecycle

When invoked, the **rendered** `SKILL.md` enters the conversation as a single message and **stays for the rest of the session** — Claude Code does not re-read the file on later turns. Write standing instructions, not one-time steps. On [auto-compaction](../claude-code/cli-and-shortcuts.md), the most recent invocation of each skill is re-attached after the summary, keeping the **first 5,000 tokens** per skill within a **combined 25,000-token budget** (filled from the most recently invoked skill, so older ones can be dropped). If a large skill stops influencing behavior after many turns, re-invoke it.

---

## Folder structure

A Skill is a directory with `SKILL.md` as the entry point. Bundle additional files and reference them *from* `SKILL.md`.

```text
pdf/
├── SKILL.md              # Main instructions (loaded when triggered)
├── FORMS.md              # Form-filling guide (loaded as needed)
├── reference.md          # API reference (loaded as needed)
├── examples.md           # Usage examples (loaded as needed)
└── scripts/
    ├── analyze_form.py   # Utility script (executed, not loaded)
    ├── fill_form.py      # Form filling script
    └── validate.py       # Validation script
```

Common conventions (not strictly enforced, but used in Anthropic's examples):

- `scripts/` — executable utilities Claude runs via bash.
- `references/` (or `reference/`) — long factual docs, organized **by domain** so Claude reads only the relevant one (e.g. `reference/finance.md`, `reference/sales.md`).
- `assets/` — templates, sample files, images Claude renders or copies.

**Keep references one level deep from `SKILL.md`.** Claude may only partially read (`head -100`) files referenced from *other* referenced files, producing incomplete information. Link every reference file directly from `SKILL.md`. For reference files over ~100 lines, add a table of contents at the top so partial reads still reveal the full scope.

---

## Where Skills run

> Custom Skills **do not sync across surfaces.** A Skill uploaded to claude.ai is not on the API; an API Skill is not on claude.ai; Claude Code Skills are filesystem-only. Manage each surface separately.

### claude.ai (web/desktop/mobile)

- **Pre-built Skills** (pptx/xlsx/docx/pdf) work behind the scenes when you create documents — no setup.
- **Custom Skills:** upload as a **zip** with the skill folder at the **zip root** (`my-skill.zip` → `my-skill/` → `SKILL.md`, *not* nested in an extra subfolder). Requires **code execution enabled**, then enable the skill in the Skills settings.
  - **WARN — menu path differs by source.** The Claude **help center** says **Settings → Customize → Skills** (URL `claude.ai/customize/skills`); the platform **overview** doc says **Settings → Features**. Both are official as of 2026-06; the help center is the more recent/UI-specific source.
- **Editing:** edit a skill's files directly in chat by highlighting text and choosing **Edit with Claude** — changes can apply across multiple files in one pass. Test with prompts to confirm it triggers.
- **Sharing scope:** **individual user only** — each teammate uploads separately. No centralized admin / org-wide distribution on claude.ai. **Network access at runtime varies** by user/admin settings (full, partial, or none).

### Claude Code (CLI / IDE / desktop / web)

- **Custom Skills only** (no pre-built document Skills); filesystem-based, **no API upload required**. See [../platform/claude-code.md](../platform/claude-code.md).
- Invoke a Skill directly with `/skill-name`, or let Claude auto-load it when relevant.
- **Custom commands have been merged into Skills:** `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both create `/deploy`. Existing `commands/` files keep working; if a skill and command share a name, the **skill wins**.
- **Bundled Skills** ship with Claude Code (prompt-based, not fixed logic): `/code-review`, `/batch`, `/debug`, `/loop`, `/claude-api`, plus the run/verify trio `/run`, `/verify`, `/run-skill-generator` (these three require Claude Code **v2.1.145+**). Disable all of them with the `disableBundledSkills` setting. They appear in the commands reference marked **Skill** in the Purpose column. See [../claude-code/slash-commands.md](../claude-code/slash-commands.md).
- **A few built-in commands are also reachable through the `Skill` tool** (so Claude can invoke them): `/init`, `/review`, `/security-review`. Others like `/compact` are **not**.

**Where Skills live** (precedence: enterprise > personal > project; any level overrides a bundled skill of the same name):

| Location | Path | Applies to |
| --- | --- | --- |
| Enterprise | managed settings | All users in the org |
| Personal | `~/.claude/skills/<name>/SKILL.md` | All your projects |
| Project | `.claude/skills/<name>/SKILL.md` | This project only |
| Plugin | `<plugin>/skills/<name>/SKILL.md` | Where the plugin is enabled (`plugin-name:skill-name` namespace) |

Extras: nested `.claude/skills/` directories load on demand (monorepos), surfacing as `apps/web:deploy`. Skills also load from `--add-dir` directories (an exception to the usual "file access only" rule). **Live change detection** picks up edits to `SKILL.md` within the session; creating a brand-new top-level skills dir needs a restart.

### Claude API

Supports both pre-built and custom Skills, integrated identically via the **`container.skills`** array (up to **8 Skills per request**) plus the **code execution tool**. Custom Skills are **workspace-wide**.

**Required beta headers:**

```
anthropic-beta: code-execution-2025-08-25,skills-2025-10-02,files-api-2025-04-14
```

- `code-execution-2025-08-25` — Skills run in the code execution container.
- `skills-2025-10-02` — enables the Skills feature/API.
- `files-api-2025-04-14` — upload/download files to/from the container (needed to retrieve generated docx/xlsx/pptx/pdf).

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: code-execution-2025-08-25,skills-2025-10-02,files-api-2025-04-14" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-opus-4-8",
    "max_tokens": 4096,
    "container": {
      "skills": [
        { "type": "anthropic", "skill_id": "pptx", "version": "latest" }
      ]
    },
    "messages": [
      { "role": "user", "content": "Create a presentation about renewable energy" }
    ],
    "tools": [
      { "type": "code_execution_20250825", "name": "code_execution" }
    ]
  }'
```

- Pre-built `skill_id`s: `pptx`, `xlsx`, `docx`, `pdf`. `type: "anthropic"`. `version` is date-based (e.g. `20251013`) or `latest`.
- Custom Skills: `type: "custom"`, `skill_id` like `skill_01AbCdEfGhIjKlMnOpQrStUv`, `version` an epoch like `1759178010641129` or `latest`.
- Multi-turn: reuse the same `container.id` across requests. Long jobs surface the `pause_turn` stop reason — continue in the next request.

See [../models/capabilities-and-modes.md](../models/capabilities-and-modes.md) for the code execution tool and container details.

---

## Installing and managing Skills

### Skills API (`/v1/skills`)

Custom Skills are managed via REST (header `anthropic-beta: skills-2025-10-02`). Upload requires a `SKILL.md` at the zip/upload root with valid `name`/`description`, total upload **under 30 MB**.

```bash
# Create / upload a custom skill (multipart files or a zip)
curl -X POST "https://api.anthropic.com/v1/skills" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "anthropic-beta: skills-2025-10-02" \
  -F "display_title=Financial Analysis" \
  -F "files[]=@skill.zip"

# List (optionally filter to custom)
curl "https://api.anthropic.com/v1/skills?source=custom" -H ...

# Retrieve one
curl "https://api.anthropic.com/v1/skills/skill_01AbCdEfGhIjKlMnOpQrStUv" -H ...

# New version
curl -X POST ".../skills/skill_01.../versions" -F "files[]=@updated/SKILL.md;filename=updated/SKILL.md" -H ...

# List versions / Delete (delete all versions first, then the skill)
curl ".../skills/skill_01.../versions" -H ...
curl -X DELETE ".../skills/skill_01..." -H ...
```

### Claude Code: `/skills` menu and settings

- `/skills` opens the management menu. Highlight a skill and press **Space** to cycle its visibility state, **Enter** to save to `.claude/settings.local.json`.
- **`skillOverrides`** (settings) sets visibility without editing a skill's frontmatter — useful for shared-repo or MCP-provided skills:

  | Value | Listed to Claude | In `/` menu |
  | --- | --- | --- |
  | `"on"` | name + description | Yes |
  | `"name-only"` | name only | Yes |
  | `"user-invocable-only"` | hidden | Yes |
  | `"off"` | hidden | hidden |

  ```json
  { "skillOverrides": { "legacy-context": "name-only", "deploy": "off" } }
  ```

- **Restrict Claude's access:** deny the `Skill` tool entirely (add `Skill` to deny rules), or scope it — `Skill(commit)` (exact), `Skill(review-pr *)` (prefix). See [../claude-code/settings.md](../claude-code/settings.md).
- **Listing budget:** all skill **names** are always listed, but **descriptions** are shortened to fit a character budget that defaults to **1% of the model's context window**. On overflow, the least-invoked skills' descriptions are dropped first. Raise it with `skillListingBudgetFraction` (e.g. `0.02` = 2%) or the `SLASH_COMMAND_TOOL_CHAR_BUDGET` env var (fixed char count). The per-entry cap (combined `description` + `when_to_use`) is `maxSkillDescriptionChars`, default **1,536**.
- **Diagnostics:** `/doctor` reports how many skill descriptions are shortened or dropped and which skills are affected. If frontmatter YAML is malformed, Claude Code loads the body with **empty metadata** — `/skill-name` still works but Claude has no `description` to match against; run with `--debug` to see the parse error.

### Community CLI: `npx skills`

Beyond Anthropic's own tooling, an **open ecosystem** has formed around the agentskills.io standard. The community `skills` CLI (from vercel-labs) installs Skills into any detected agent:

```bash
# list skills in a repo
npx skills add vercel-labs/agent-skills --list
# install a skill (auto-detects installed agents; installs to .agents/skills/ and symlinks)
npx skills add anthropics/skills
```

WARN: `npx skills` is a third-party tool, not an Anthropic product; verify a repo's trustworthiness before installing. The MCP server instructions in some products also suggest `npx skills add supabase/agent-skills` for vendor-published Skills.

---

## Built-in / Anthropic Skills

| Skill | `skill_id` | What it does |
| --- | --- | --- |
| **PowerPoint** | `pptx` | Create presentations, edit slides, analyze slide content |
| **Excel** | `xlsx` | Build spreadsheets, analyze data, generate charts/reports |
| **Word** | `docx` | Create/edit documents, format text, tracked changes (redlining) |
| **PDF** | `pdf` | Generate formatted PDFs and reports; extract text/tables, fill forms |

Available on the Claude API, claude.ai, Claude Platform on AWS, and Microsoft Foundry. Anthropic also publishes **open-source Skills** at [github.com/anthropics/skills](https://github.com/anthropics/skills), including a **Claude API** skill (up-to-date API/SDK reference for 8 languages) that is bundled with Claude Code.

---

## Authoring best practices

The single most important rule: **"Concise is key."** Claude is already smart — only add context it doesn't have. Every token in `SKILL.md` competes with conversation history once loaded.

**Descriptions (the trigger):** Claude chooses a Skill from potentially 100+ purely by its `description`. Write third person, include **what + when**, and pack in concrete trigger keywords.

```yaml
# Good
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
# Bad
description: Helps with documents
description: I can help you process Excel files   # wrong POV
```

**Naming:** prefer **gerund form** — `processing-pdfs`, `analyzing-spreadsheets`. Avoid vague names (`helper`, `utils`, `tools`) and reserved words (`anthropic`, `claude`).

**Keep the body lean:** under **500 lines**; split overflow into linked files. Match **degrees of freedom** to the task: high freedom (open instructions) for judgment-heavy tasks, low freedom (exact scripts, "do not modify the command") for fragile/destructive ones.

**Progressive-disclosure patterns:**
1. *High-level guide with references* — quick start inline; link `FORMS.md`, `REFERENCE.md`, `EXAMPLES.md`.
2. *Domain-specific organization* — `reference/finance.md`, `reference/sales.md`, so a sales question never loads finance schemas.
3. *Conditional details* — basic inline, advanced behind links read only when needed.

**Scripts:** "Solve, don't punt" — handle errors in scripts rather than letting Claude improvise; avoid voodoo constants (justify every value); always use **forward slashes** in paths (Windows backslashes break on Unix). Make execution intent explicit ("**Run** `analyze_form.py`" vs "**See** `analyze_form.py` for the algorithm").

**Workflows & feedback loops:** give a copyable checklist for multi-step tasks; use the run-validator → fix → repeat loop and the plan → validate → execute pattern for batch/destructive operations.

**MCP references:** use fully qualified `ServerName:tool_name` (e.g. `GitHub:create_issue`) so tools resolve.

**Avoid time-sensitive info:** put deprecated guidance in a collapsible "Old patterns" section, not inline "before August 2025…".

**Evals first:** build ~3 evaluation scenarios *before* writing extensive docs; measure a **baseline without the Skill** in a fresh session (leftover authoring context masks gaps), then write minimal instructions to close the gap. Test against **Haiku, Sonnet, and Opus** — what works for Opus may need more detail for Haiku. A simple eval entry pairs `skills`, `query`, `files`, and `expected_behavior` (there is no built-in runner for these JSON evals on the API/platform side; you supply the harness).

**`skill-creator` (Claude Code):** the official plugin automates the loop — `/plugin install skill-creator@claude-plugins-official`, then `/reload-plugins`, then ask e.g. *"evaluate my summarize-changes skill with skill-creator."* It stores test cases in `evals/evals.json`, runs each in an isolated subagent, writes pass/fail to `grading.json`, aggregates with-vs-without-skill pass rate / time / tokens into `benchmark.json`, runs blind A/B between two versions, and tunes the `description` against should-trigger / should-not-trigger prompts.

**Dev with two Claudes:** the docs recommend using one Claude instance ("Claude A") to author/refine the Skill and a fresh instance ("Claude B") to test it on real tasks, bringing observed gaps back to Claude A. Claude writes well-formed `SKILL.md` natively — no special "skill-writing skill" needed.

Pre-share checklist (abridged): description specific + what/when • body < 500 lines • references one level deep • consistent terminology • forward-slash paths • scripts handle errors • ≥3 evals • tested across models.

---

## Worked example: a complete custom Skill

A `commit-helper` Skill that generates conventional-commit messages from staged changes. Folder layout:

```text
commit-helper/
├── SKILL.md
├── reference/
│   └── conventions.md
└── scripts/
    └── staged_diff.sh
```

`SKILL.md`:

````markdown
---
name: commit-helper
description: Generate Conventional Commit messages by analyzing staged git changes. Use when the user asks for a commit message, wants to review staged changes, or is about to commit.
license: Apache-2.0
allowed-tools: Bash(git diff *) Bash(git status *)
---

# Commit Helper

## Workflow
Copy this checklist and check items off:

```
- [ ] 1. Read the staged diff
- [ ] 2. Classify the change type
- [ ] 3. Draft a Conventional Commit message
- [ ] 4. Verify it matches our conventions
```

## 1. Read the staged diff
Run the bundled script and read its output:

```bash
bash scripts/staged_diff.sh
```

## 2-3. Draft the message
Use `type(scope): summary` then a blank line and a body. See
[reference/conventions.md](reference/conventions.md) for the allowed
types and scopes. Examples:

- Input: added JWT auth  → `feat(auth): implement JWT-based authentication`
- Input: fixed date bug  → `fix(reports): correct date formatting in timezone conversion`

## 4. Verify
If the type is not in conventions.md, return to step 2.
````

`scripts/staged_diff.sh`:

```bash
#!/usr/bin/env bash
# Print staged changes; exit cleanly if nothing is staged.
set -euo pipefail
if git diff --cached --quiet; then
  echo "No staged changes."
  exit 0
fi
git diff --cached --stat
echo "---"
git diff --cached
```

`reference/conventions.md` (loaded only when Claude needs the full list):

```markdown
# Commit conventions

## Contents
- Allowed types
- Scopes
- Body rules

## Allowed types
feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert

## Scopes
auth, api, ui, reports, db, deps
...
```

**Install it:**

- **Claude Code (personal):** `mkdir -p ~/.claude/skills/commit-helper/{scripts,reference}` and save the files; invoke with `/commit-helper` or just ask "write me a commit message."
- **claude.ai:** zip the `commit-helper/` folder (folder at zip root), upload via the Skills settings (**Settings → Customize → Skills**, or **Settings → Features** — see the menu-path note above), enable it.
- **API:** `POST /v1/skills -F "files[]=@commit-helper.zip"`, then reference the returned `skill_id` (type `custom`) in `container.skills`.

---

## Constraints, security, and data retention

**Runtime by surface:**

| Surface | Network | Package install |
| --- | --- | --- |
| **claude.ai** | Varies by user/admin settings (full, partial, or none); can install from npm/PyPI/GitHub when allowed | Allowed when network is on |
| **Claude API** | **None** — no external calls/internet | **None at runtime** — only pre-installed packages in the code execution container |
| **Claude Code** | Full (same as any program on your machine) | Allowed, but install **locally**, not globally |

**Security:** Treat installing a Skill like installing software. Use Skills only from trusted sources (yourself or Anthropic). A malicious Skill can direct Claude to misuse tools, exfiltrate data, or run harmful code; Skills that fetch external URLs are especially risky. Audit every bundled file — `SKILL.md`, scripts, assets — before use. In Claude Code, project-level `allowed-tools` only take effect after you accept the **workspace trust** dialog; `disableSkillShellExecution: true` (best set in managed settings) replaces `` !`command` `` injection with a disabled-by-policy placeholder.

**Data retention:** Agent Skills are **not eligible for Zero Data Retention (ZDR)**; skill definitions and execution data follow Anthropic's standard retention policy.

---

## Related pages

- [./plugins.md](./plugins.md) — bundle and distribute Skills alongside hooks, agents, and MCP servers (`skill-creator`, marketplaces, `/plugin`).
- [../platform/claude-code.md](../platform/claude-code.md) — the Claude Code surface that runs filesystem Skills.
- [../claude-code/slash-commands.md](../claude-code/slash-commands.md) — bundled Skills (`/code-review`, `/debug`, `/loop`) and the command↔skill merge.
- [../claude-code/settings.md](../claude-code/settings.md) — `disableBundledSkills`, `skillOverrides`, `skillListingBudgetFraction`, `disableSkillShellExecution`, `Skill(...)` permissions.
- [../claude-code/subagents.md](../claude-code/subagents.md) — `context: fork`, the `agent` field, and preloading skills into subagents.
- [../claude-code/hooks.md](../claude-code/hooks.md) — the `hooks` frontmatter field scoping hooks to a skill.
- [./connectors.md](./connectors.md) and [./mcp.md](./mcp.md) — tools a Skill can drive (fully qualified `ServerName:tool_name`).
- [../models/capabilities-and-modes.md](../models/capabilities-and-modes.md) — the code execution tool/container Skills run in.
- [./projects.md](./projects.md) — always-on knowledge vs. on-demand Skills.

## Open questions / to verify

- **claude.ai plan eligibility** for custom Skills: platform overview lists **Pro/Max/Team/Enterprise** (no Free); the help center lists **Free** too. Both are live official docs (2026-06) — the discrepancy is unresolved, so Free is treated as unconfirmed.
- **claude.ai menu path:** help center says **Settings → Customize → Skills** (`claude.ai/customize/skills`); platform overview still says **Settings → Features**. UI labels may be mid-migration.
- Per-surface validation of the open-standard optional keys (`license`, `metadata`, `compatibility`): defined by agentskills.io and accepted by the API uploader (which only *validates* `name`/`description`), but exact enforcement on claude.ai is not documented.
- Whether the API per-request limit of **8 Skills** and the **30 MB** upload cap remain current — both verified present in docs as of 2026-06.
- Latest pre-built Skill dated versions (docs example shows `20251013`; `version: "latest"` always resolves to current).

## Sources

- [Agent Skills overview — Claude Platform Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Skill authoring best practices — Claude Platform Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Using Agent Skills with the Claude API](https://platform.claude.com/docs/en/build-with-claude/skills-guide)
- [Extend Claude with skills — Claude Code Docs](https://code.claude.com/docs/en/skills)
- [Creating custom Skills — Claude Help Center](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [What are Skills? — Claude Help Center](https://support.claude.com/en/articles/12512176-what-are-skills)
- [Equipping agents for the real world with Agent Skills — Anthropic Engineering](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- [anthropics/skills (open-source Skills repo)](https://github.com/anthropics/skills)
- [Agent Skills specification — agentskills.io](https://agentskills.io/specification)
- [vercel-labs/skills (`npx skills` CLI)](https://github.com/vercel-labs/skills)
