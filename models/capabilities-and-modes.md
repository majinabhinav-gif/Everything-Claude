---
type: Model Reference
title: Model Capabilities & Modes
description: Exhaustive reference to what Claude models can do — extended/adaptive thinking, effort, fast mode, 1M context, vision, tool use, structured outputs, caching, batch, citations, computer use, multilingual, streaming, token counting — and how each is toggled across the API, Claude.ai, and Claude Code.
domain: models
tags: [thinking, effort, fast-mode, 1m-context, vision, tool-use, structured-outputs, prompt-caching, batch-api, citations, computer-use, streaming, token-counting, capabilities]
related: [model-families, slash-commands, cli-shortcuts, mcp, connectors, skills, artifacts]
resource: https://platform.claude.com/docs/en/build-with-claude/overview
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://platform.claude.com/docs/en/build-with-claude/extended-thinking
  - https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking
  - https://platform.claude.com/docs/en/build-with-claude/effort
  - https://platform.claude.com/docs/en/build-with-claude/context-windows
  - https://platform.claude.com/docs/en/build-with-claude/vision
  - https://platform.claude.com/docs/en/build-with-claude/pdf-support
  - https://platform.claude.com/docs/en/build-with-claude/structured-outputs
  - https://platform.claude.com/docs/en/build-with-claude/prompt-caching
  - https://platform.claude.com/docs/en/build-with-claude/batch-processing
  - https://platform.claude.com/docs/en/build-with-claude/citations
  - https://platform.claude.com/docs/en/build-with-claude/streaming
  - https://platform.claude.com/docs/en/build-with-claude/token-counting
  - https://platform.claude.com/docs/en/build-with-claude/fast-mode
  - https://code.claude.com/docs/en/fast-mode
  - https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
  - https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool
  - https://platform.claude.com/docs/en/build-with-claude/vision-coordinates
  - https://support.claude.com/en/articles/10574485-using-extended-thinking
  - https://platform.claude.com/docs/en/api/rate-limits
  - https://platform.claude.com/docs/en/api/service-tiers
  - https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons
---

# Model Capabilities & Modes

This page documents what the Claude model family can **do** — reasoning depth, vision, tool use, structured output, caching, batching, citations, computer use, multilingual generation, streaming, and token accounting — and, for each capability, exactly **where you turn it on**: an API parameter, a Claude.ai toggle, or a Claude Code flag/command. It is a behavior-and-toggle reference; for model IDs, pricing, and context sizes per model see [./model-families.md](./model-families.md).

The single most important shift to internalize for 2026 models: **the fixed "thinking budget" is gone.** On Fable 5 / Opus 4.8 / 4.7 / 4.6 and Sonnet 4.6, you control reasoning with **adaptive thinking** plus an **effort** dial, not `budget_tokens`. Sending `thinking: {type: "enabled", budget_tokens: N}` to a current model returns a `400`.

## At a glance

| | |
|---|---|
| **What it is** | The cross-cutting capabilities and runtime "modes" of Claude models — reasoning, perception, tool use, output control, cost/latency levers — independent of which specific model you pick. |
| **Where you find it** | **API:** request parameters on `POST /v1/messages` (`thinking`, `output_config`, `tools`, `cache_control`, `speed`, …). **Claude.ai:** model picker next to the send button → Extended thinking toggle, plus per-feature toggles. **Claude Code:** `/model`, `/fast`, `--model`, effort in `/model-config`, `settings.json`. |
| **Who can use it by plan** | Most capabilities are available on every API tier and on Pro/Max/Team/Enterprise. Plan-gated specifics are flagged inline (e.g. Fast mode requires usage credits; Enterprise gets larger Claude.ai context). |
| **Status** | Stable, with several betas (compaction, context editing, task budgets, advisor tool, MCP connector) and one research preview (Fast mode). 2026 models are adaptive-thinking-only. |

> Note on verification — capability, pricing, and API-parameter facts on this page were fact-checked against official docs (platform.claude.com, code.claude.com) in June 2026: context windows, vision/high-res limits, Fast mode pricing & toggles, and the computer-use tool version are all confirmed. The few consumer-surface labels and keyboard chords that change frequently on Claude.ai and in Claude Code are marked "WARN: verify" inline.

---

## 1. Extended thinking & adaptive reasoning

### What it is and when it helps

**Extended thinking** lets Claude produce internal reasoning (`thinking` content blocks) before its final answer — planning, exploring approaches, checking work. It improves accuracy on multi-step math, complex coding, agentic planning, and anything requiring deliberation. It costs more (thinking tokens bill as output) and adds latency, so it is not free.

On 2026 models the mechanism is **adaptive thinking**: Claude itself decides *whether* and *how much* to think per request, and automatically interleaves thinking between tool calls (no beta header). You no longer hand it a fixed token budget.

| Model family | How thinking is controlled |
|---|---|
| Fable 5 / Mythos 5 | **Always on.** Omit `thinking` entirely (or send `{type: "adaptive"}`). `{type: "disabled"}` and `{type: "enabled", budget_tokens: N}` both `400`. Depth via `effort`. |
| Opus 4.8 / 4.7 | Adaptive only. `thinking: {type: "adaptive"}` to enable; **off by default** when the field is omitted. `budget_tokens` `400`s. `{type: "disabled"}` is accepted. |
| Opus 4.6 / Sonnet 4.6 | Adaptive recommended (`{type: "adaptive"}`). `budget_tokens` is deprecated but still functional (transitional escape hatch). |
| Sonnet 4.5 / Haiku 4.5 / older | Classic extended thinking: `thinking: {type: "enabled", budget_tokens: N}` where `N < max_tokens` (min 1024). No `effort` parameter. |

### Turn it on — API

```python
# Fable 5 / Opus 4.8 / 4.7 / 4.6 — adaptive thinking, depth via effort
resp = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=16000,
    thinking={"type": "adaptive", "display": "summarized"},  # see "thinking display" below
    output_config={"effort": "high"},
    messages=[{"role": "user", "content": "Prove the AM-GM inequality, then check your proof."}],
)
for block in resp.content:
    if block.type == "thinking":
        print("REASONING:", block.thinking)   # empty string unless display="summarized"
    elif block.type == "text":
        print("ANSWER:", block.text)
```

```python
# Older model — classic budgeted thinking
client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=16000,
    thinking={"type": "enabled", "budget_tokens": 8000},  # must be < max_tokens
    messages=[...],
)
```

### Thinking display (Fable 5 / Opus 4.8 / 4.7)

`display` controls *visibility only* — thinking happens and is billed identically under every setting; the raw chain of thought is never returned.

- `"omitted"` (**default** on Fable 5 / Mythos 5 / Opus 4.8 / 4.7) — `thinking` blocks stream with an **empty** `thinking` field. To a user this looks like a long silent pause before output.
- `"summarized"` — returns a readable summary of the reasoning. Set this explicitly if you stream reasoning to users: `thinking={"type": "adaptive", "display": "summarized"}`.

> Replay rule: when continuing on the **same** model, echo `thinking` blocks back **unchanged** (including empty-text ones). The API rejects *modified* blocks, not read ones. A *different* model silently drops them from the prompt (and from billing).

### Turn it on — Claude.ai

Click the **model name next to the send button**, then switch the **"Extended" / Thinking toggle** on or off. (On supported models the assistant decides depth adaptively; the toggle gates whether extended reasoning is allowed at all.)

### Turn it on — Claude Code

Thinking is governed by the **effort** setting (next section) and the model; there is no separate "thinking budget" knob. Type `/model-config` (or `/model`) to choose model + effort. Historically, typing words like `think` / `think hard` / `ultrathink` in a prompt nudged more reasoning — on current models prefer raising effort. WARN: verify the current trigger-word behavior for your Claude Code version.

---

## 2. Reasoning effort (the depth/cost dial)

### What it is

`effort` controls how much Claude thinks **and acts** — thinking depth, number of tool calls, preamble length, terseness. It is GA (no beta header) and lives **inside `output_config`**, not at the top level. Default is `high` (equivalent to omitting it).

| Level | Best for | Notes |
|---|---|---|
| `low` | Latency-sensitive, scoped tasks, subagents | Fewest/most-consolidated tool calls, least preamble. On Fable 5, `low` can beat prior models' `xhigh`. |
| `medium` | Cost-sensitive balanced work | Good quality/cost tradeoff for many apps. |
| `high` | Most intelligence-sensitive work | The recommended **minimum** for hard work; the default. |
| `xhigh` | Coding & agentic use cases | Added in Opus 4.7; between `high` and `max`. The default in Claude Code. |
| `max` | Correctness over cost | Opus 4.6+ and Sonnet 4.6 only (not Haiku/earlier Sonnets). Can overthink. |

Supported on Fable 5, Opus 4.5/4.6/4.7/4.8, and Sonnet 4.6. **Errors** on Sonnet 4.5 / Haiku 4.5.

### Turn it on — API

```python
client.messages.create(
    model="claude-opus-4-8",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "xhigh"},   # low | medium | high | xhigh | max
    messages=[...],
)
```

### Turn it on — Claude Code

`/model-config` → **Adjust effort level**, or set it in settings. See [../claude-code/cli-and-shortcuts.md](../claude-code/cli-and-shortcuts.md) and [../claude-code/slash-commands.md](../claude-code/slash-commands.md). Effort and Fast mode are independent and **combinable**: fast mode + low effort = maximum speed on simple tasks.

### Task budgets (beta — agentic loops)

Distinct from `effort` (per-turn depth) and `max_tokens` (a hard, model-invisible per-response cap). A **task budget** tells the model how many tokens it has for an entire agentic loop; it sees a running countdown and paces itself.

```python
with client.beta.messages.stream(
    model="claude-opus-4-8",
    max_tokens=128000,
    output_config={"effort": "high", "task_budget": {"type": "tokens", "total": 64000}},
    betas=["task-budgets-2026-03-13"],
    messages=[...], tools=[...],
) as stream:
    final = stream.get_final_message()
```

Minimum `total` is 20,000. Fable 5 / Opus 4.8 / 4.7 only. In Claude Code, the equivalent intent is set via `/goal` (state the objective up front).

---

## 3. Fast mode (Claude Code & API research preview)

### What it is

**Fast mode** runs the same Opus model at **up to 2.5× higher output tokens per second**, at premium flat pricing. It is *not* a different model — same quality, lower latency. It speeds **OTPS** (streaming rate once output starts), **not TTFT** (the initial pause). Research preview; pricing/availability may change.

- **Supported models:** Opus 4.8, 4.7, 4.6. **Not** Sonnet/Haiku. Opus 4.6 fast mode is **deprecated** (removed ~30 days after the 4.8 launch; afterward `speed: "fast"` on 4.6 silently falls back to standard speed).
- **Pricing (per MTok in/out, flat across the full 1M context):** Opus 4.8 **$10 / $50**; Opus 4.7 & 4.6 **$30 / $150**.
- **Not available** on the Batch API, Priority Tier, Claude Platform on AWS, Bedrock, Vertex, or Foundry.

### Turn it on — Claude Code

```text
/fast        # press Tab to toggle on/off; switches you to Opus if you weren't on it
```

- Confirmation: "Fast mode ON"; a small `↯` icon shows next to the prompt while active. Toggling off keeps you on Opus (use `/model` to switch models).
- Or set `"fastMode": true` in user settings. Persists across sessions by default.
- Requires Claude Code **v2.1.36+** (`claude --version`). **Not** in the VS Code extension. The fast-mode model default is **Opus 4.8 on v2.1.154+** (Opus 4.7 on v2.1.142–2.1.153).
- Requires **usage credits** turned on (fast-mode tokens always bill at the fast rate, outside plan-included usage). Team/Enterprise: an Owner must enable it ([Admin Settings → Claude Code](https://claude.ai/admin-settings/claude-code)); Console API customers enable it in [Claude Code preferences](https://platform.claude.com/claude-code/preferences).
- Admin controls: `fastModePerSessionOptIn: true` resets fast mode off each session; `CLAUDE_CODE_DISABLE_FAST_MODE=1` disables it entirely.
- On rate-limit/credit exhaustion it auto-falls-back to standard speed (icon greys), then auto-re-enables after cooldown.

### Turn it on — API

Use the **beta** endpoint, pass the beta flag, and set `speed: "fast"` as a top-level parameter:

```python
client.beta.messages.create(
    model="claude-opus-4-8",
    max_tokens=4096,
    speed="fast",
    betas=["fast-mode-2026-02-01"],
    messages=[...],
)
# response.usage.speed reports which speed actually served the request
```

Switching speed mid-conversation invalidates the prompt cache, so enable it from the **start** of a session for best cost efficiency.

---

## 4. The 1M-token context window

### What it is

The maximum working memory for a request. Fable 5, Mythos 5, Opus 4.8/4.7/4.6, and Sonnet 4.6 have a **1M-token** context window on the Claude API, Amazon Bedrock, and Google Cloud. Opus 4.8 on Microsoft Foundry is 200K. Sonnet 4.5, Haiku 4.5, and older are 200K. On Fable 5 / Mythos 5 the 1M maximum is also the **default**.

> Long-context premium: Opus 4.8 and 4.7 both provide 1M at **standard API pricing with no long-context premium** (confirmed in the official model/migration docs). Some *earlier* models historically applied a >200K-token surcharge; check [./model-families.md](./model-families.md) and the pricing page for the exact model you use.

### Enable it — API

For current 1M-capable models (Fable 5/Mythos 5/Opus 4.8/4.7/4.6/Sonnet 4.6), the full 1M window is available **with no beta header** — confirmed against the official context-windows docs. (The `context-1m-2025-08-07` header was a legacy opt-in on an earlier model and is not needed on these.) Query the Models API for the live limit on any model:

```python
m = client.models.retrieve("claude-opus-4-8")
m.max_input_tokens   # 1000000
m.max_tokens         # 128000 (output cap)
```

### When to use it

Large codebases, long documents, multi-hour agent transcripts. But **more context isn't automatically better** — accuracy degrades as token count grows ("context rot"). For long-running work, prefer **compaction** (server-side summarization) or **context editing** (clearing stale tool results/thinking) over simply stuffing the window. Sonnet 4.6 / Sonnet 4.5 / Haiku 4.5 are **context-aware** — they receive a live `<budget:token_budget>` and `<system_warning>Token usage: …</system_warning>` so they pace long tasks.

### Overflow behavior

On Claude 4.5+ models, if `input + max_tokens` exceeds the window the request is accepted; if generation hits the limit it stops with `stop_reason: "model_context_window_exceeded"` (handle this distinctly from `max_tokens`).

### Claude.ai context

The 1M window is **API-oriented**. As of early 2026 the Claude.ai web interface uses a **200K** context across plans, with **Enterprise** getting a larger window (≈500K on some models). WARN: verify current Claude.ai/Enterprise context sizes — these change. See [../platform/claude-ai.md](../platform/claude-ai.md).

---

## 5. Vision (images, multi-image, PDFs, charts/diagrams)

### What it is

Claude reads images and documents: photos, screenshots, scanned pages, charts, diagrams, tables, handwriting. Multiple images per request supported:

| Surface | Max images/request |
|---|---|
| API, 1M-context models (Fable 5/Mythos 5/Opus 4.8/4.7/4.6/Sonnet 4.6) | **600** images or PDF pages |
| API, 200K models (Sonnet 4.5/Haiku 4.5/older) | **100** images or PDF pages |
| Claude.ai | **20** per turn |

Per-image limits: **10 MB** each (5 MB on Bedrock/Vertex), max **8000×8000 px**. Formats: **JPEG, PNG, GIF, WebP** (animations unsupported — only the first frame is read). If a request has **>20 images**, a stricter per-image dimension limit kicks in (resize so neither dimension exceeds 2000 px to stay safe). Claude views images in **28×28-px patches** ("visual tokens"); cost ≈ `⌈w/28⌉ × ⌈h/28⌉` tokens. Claude **cannot generate or edit images** and **cannot name people** in them (refuses by AUP).

- **High-resolution vision** — applies to **Fable 5 / Mythos 5 / Opus 4.8 / Opus 4.7**: up to **2576 px on the long edge** / **4784 visual tokens** (vs **1568 px / 1568 tokens** on all other models, the "standard tier"). Coordinates the model returns are absolute pixels relative to the (possibly resized) image it sees — see the [coordinates guide](https://platform.claude.com/docs/en/build-with-claude/vision-coordinates) (key for computer use). Automatic; **no beta header, no opt-in**. A high-res image can cost ~3× the tokens of the same image on standard tier — downsample client-side if you don't need the fidelity.
- Fable 5 is explicitly trained to **use bash/crop/zoom tools** on flipped, blurry, or noisy images.

### Turn it on — API (image, base64)

```python
import base64
img = base64.standard_b64encode(open("chart.png","rb").read()).decode()  # no newlines
client.messages.create(
    model="claude-opus-4-8", max_tokens=2048,
    messages=[{"role": "user", "content": [
        {"type": "image", "source": {"type": "base64", "media_type": "image/png", "data": img}},
        {"type": "text", "text": "Transcribe the data series in this chart as a table."},
    ]}],
)
```

Image by URL: `{"type": "image", "source": {"type": "url", "url": "https://…/x.png"}}`. Formats: JPEG, PNG, GIF, WebP.

### Turn it on — API (PDF / document)

```python
import base64
pdf = base64.standard_b64encode(open("report.pdf","rb").read()).decode()
client.messages.create(
    model="claude-opus-4-8", max_tokens=4096,
    messages=[{"role": "user", "content": [
        {"type": "document", "source": {"type": "base64", "media_type": "application/pdf", "data": pdf}},
        {"type": "text", "text": "List every figure caption and the page it's on."},
    ]}],
)
```

Limits: **32 MB** per request (the standard-endpoint request-size cap; lower on Bedrock/Vertex), **600 PDF pages** on 1M-context models (100 on 200K). PDF pages count toward the same 600/100 budget as images. For reuse across calls, upload via the Files API (`files-api-2025-04-14` beta) and reference `{"type": "document", "source": {"type": "file", "file_id": "…"}}` — the beta header is required on **both** the upload and the referencing `messages.create`.

### Turn it on — Claude.ai / Claude Code

**Claude.ai:** attach/drag an image or PDF into the composer. **Claude Code:** paste an image, or reference an image path; the `read` tool reads images, PDFs, and notebooks. See [../platform/claude-ai.md](../platform/claude-ai.md).

---

## 6. Tool use / function calling

### What it is

Claude can call tools you define (function calling) and Anthropic-hosted server tools. You declare tools by JSON schema; Claude emits `tool_use` blocks; you return `tool_result` blocks. Default is parallel tool use.

### Turn it on — API (custom tool)

```python
tools = [{
    "name": "get_weather",
    "description": "Get current weather. Call this when the user asks about current conditions.",
    "input_schema": {
        "type": "object",
        "properties": {"location": {"type": "string"}},
        "required": ["location"],
        "additionalProperties": False,
    },
    "strict": True,   # guarantees tool_use.input validates exactly (no beta header)
}]
resp = client.messages.create(model="claude-opus-4-8", max_tokens=1024, tools=tools, messages=[...])
```

`tool_choice`: `{"type":"auto"}` (default), `{"type":"any"}` (must use one), `{"type":"tool","name":"…"}` (force one), `{"type":"none"}`. Add `"disable_parallel_tool_use": true` to cap at one call. The SDK **tool runner** (`client.beta.messages.tool_runner(...)`) drives the loop for you.

> Parallel results go in **one** user message. Splitting `tool_result` blocks across messages trains Claude to stop making parallel calls. For a failed tool, return `tool_result` with `is_error: true` — don't drop it.

### Server-side tools (Anthropic-hosted)

Declare in `tools`; results return as content blocks in the same response. **No beta header** unless noted.

| Tool | `type` (latest) | Notes |
|---|---|---|
| Web search | `web_search_20260209` | Built-in dynamic filtering; do **not** also declare `code_execution`. Older models: `web_search_20250305`. Vertex: basic only. |
| Web fetch | `web_fetch_20260209` | Fetches URLs already present in the conversation. |
| Code execution | `code_execution_20260521` | Sandboxed Python 3.11 + data-science libs; returns `bash_code_execution_tool_result` (`.content.stdout`). |
| Tool search | `tool_search_tool_regex_20251119` / `…_bm25_20251119` | Discover tools without loading all schemas; mark others `defer_loading: true`. |
| Advisor (beta) | `advisor_20260301` | Pairs a cheap executor model with a more capable advisor model. |

> Server-tool errors return **HTTP 200** with an error content block, not an exception. A server-side loop that hits its iteration cap returns `stop_reason: "pause_turn"` — re-send to resume (don't add a "continue" message).

### Client-side Anthropic tools (schema-less)

Declare by type/name only — **no `input_schema`** — and execute locally: `{"type":"bash_20250124","name":"bash"}`, `{"type":"text_editor_20250728","name":"str_replace_based_edit_tool"}`, `{"type":"memory_20250818","name":"memory"}`.

### Turn it on — Claude.ai / Claude Code

**Claude.ai:** tools arrive via **Connectors** (MCP), built-in web search, code/analysis, and Skills — toggled in the message composer and settings. See [../capabilities/connectors.md](../capabilities/connectors.md), [../capabilities/mcp.md](../capabilities/mcp.md), [../capabilities/skills.md](../capabilities/skills.md). **Claude Code:** has built-in tools (bash, read/write/edit, glob, grep, web) plus MCP and subagents; gate them via permissions in `settings.json`. See [../claude-code/settings.md](../claude-code/settings.md), [../claude-code/subagents.md](../claude-code/subagents.md), [../claude-code/hooks.md](../claude-code/hooks.md).

---

## 7. Structured outputs / JSON mode

### What it is

Constrains responses to a JSON schema (guaranteed valid, parseable). Two flavors: **JSON outputs** via `output_config.format`, and **strict tool use** via `strict: true` on a tool. Supported on Fable 5, Opus 4.8, Sonnet 4.6, Haiku 4.5 (and legacy Opus 4.5/4.1). To select a model at runtime by capability, query the Models API: `client.models.retrieve("claude-opus-4-8").capabilities["structured_outputs"]["supported"]`.

### Turn it on — API

```python
schema = {
    "type": "object",
    "properties": {"name": {"type": "string"}, "email": {"type": "string", "format": "email"}},
    "required": ["name", "email"],
    "additionalProperties": False,
}
resp = client.messages.create(
    model="claude-opus-4-8", max_tokens=1024,
    output_config={"format": {"type": "json_schema", "schema": schema}},
    messages=[{"role": "user", "content": "Extract: Jane Doe, jane@co.com"}],
)
```

Use `client.messages.parse(...)` to validate automatically. **Notes:** the deprecated top-level `output_format` is replaced by `output_config.format`; new schemas pay a one-time compile cost (24h cache); **incompatible with citations** (returns 400) and with message prefilling; numeric/length constraints (`minimum`, `maxLength`) and recursive schemas are unsupported (`additionalProperties: false` is required).

---

## 8. Prompt caching

### What it is

Caching reuses an identical request **prefix** across calls — cache reads cost ~0.1× input price; writes cost 1.25× (5-min TTL) or 2× (1-hour TTL). It is a strict **prefix match**: any byte change anywhere in the prefix invalidates everything after it (render order `tools` → `system` → `messages`).

### Turn it on — API

```python
client.messages.create(
    model="claude-opus-4-8", max_tokens=16000,
    system=[{"type": "text", "text": LARGE_SHARED_PROMPT, "cache_control": {"type": "ephemeral"}}],
    messages=[{"role": "user", "content": "Summarize the key points"}],
)
# Verify:  resp.usage.cache_read_input_tokens  > 0  on the 2nd identical-prefix request
```

Top-level `cache_control` on `messages.create()` auto-places on the last cacheable block. Max **4** breakpoints. Minimum cacheable prefix is model-dependent (4096 tokens on Opus 4.8/4.7/4.6; 2048 on Fable 5/Sonnet 4.6) — shorter prefixes silently won't cache. **Automatic prompt caching is first-party / Foundry only** (not Bedrock/Vertex). Avoid silent invalidators: `datetime.now()` in the system prompt, unsorted JSON, a per-user tool set. See [../capabilities/mcp.md](../capabilities/mcp.md) for tool-list stability. **Mid-conversation system messages** (Opus 4.8): append `{"role":"system",...}` to `messages[]` instead of editing top-level `system`, to preserve the cached prefix.

---

## 9. The Batch API

### What it is

Asynchronous bulk processing at **50%** of standard token prices. Up to **100,000 requests / 256 MB** per batch; most finish within 1 hour (max 24h); results retained 29 days. First-party (and Claude Platform on AWS) only — not Bedrock/Vertex/Foundry.

### Turn it on — API

```python
batch = client.messages.batches.create(requests=[
    {"custom_id": "r1", "params": {"model": "claude-opus-4-8", "max_tokens": 1024,
        "messages": [{"role": "user", "content": "Summarize climate impacts"}]}},
    {"custom_id": "r2", "params": {"model": "claude-opus-4-8", "max_tokens": 1024,
        "messages": [{"role": "user", "content": "Explain quantum computing"}]}},
])
# poll batches.retrieve(batch.id).processing_status == "ended", then stream batches.results(batch.id)
```

> Results arrive in **any order** — key by `custom_id`, never by position. All Messages features (vision, tools, caching) work in batch. Fast mode does **not**.

---

## 10. Citations

### What it is

When enabled on document content blocks, Claude grounds its answer in the source and returns exact `cited_text` spans with location metadata. No beta header.

### Turn it on — API

```python
client.messages.create(model="claude-opus-4-8", max_tokens=2048, messages=[{"role":"user","content":[
    {"type": "document",
     "source": {"type": "text", "media_type": "text/plain", "data": SOURCE_TEXT},
     "title": "Q4 Report",
     "citations": {"enabled": True}},
    {"type": "text", "text": "What were the Q4 revenue drivers?"},
]}])
```

Cited blocks carry a `citations` array; each citation has `cited_text`, `document_index`, `document_title`, and a location by type: `char_location` (plain text), `page_location` (1-indexed PDF), or `content_block_location`. Set `citations.enabled` on **all or none** of the documents. **Incompatible with `output_config.format`.** On Claude.ai, citations surface automatically in web search and connected-source answers.

---

## 11. Computer use

### What it is

Claude operates a GUI — takes screenshots, moves the mouse, types — to drive desktop and web apps. Beta on all platforms (Bedrock/Vertex/Foundry included). Client-side: your harness runs the environment and executes the actions; Anthropic processes screenshots/actions but does not host the environment. Benefits from high-res vision (Fable 5/Opus 4.8/4.7): pixel-accurate coordinates, less downscaling.

### Tool version & beta header (verified)

| `type` | Beta header | Models |
|---|---|---|
| `computer_20251124` | `computer-use-2025-11-24` | Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 4.6, Opus 4.5 |
| `computer_20250124` | `computer-use-2025-01-24` | Sonnet 4.5, Haiku 4.5, Opus 4.1, Sonnet 4, Opus 4 |

### Turn it on — API

```python
client.beta.messages.create(
    model="claude-opus-4-8", max_tokens=2048,
    betas=["computer-use-2025-11-24"],
    tools=[
        {"type": "computer_20251124", "name": "computer",
         "display_width_px": 1024, "display_height_px": 768, "display_number": 1,
         "enable_zoom": True},                                   # zoom is new on computer_20251124
        {"type": "text_editor_20250728", "name": "str_replace_based_edit_tool"},
        {"type": "bash_20250124", "name": "bash"},
    ],
    messages=[{"role": "user", "content": "Open the Settings app and turn on dark mode."}],
)
```

- **Display config:** `display_width_px` / `display_height_px` required; `display_number` optional (X11). Keep the resolution at or below the model's vision limit so screenshots aren't downscaled.
- **`enable_zoom: true`** (only on `computer_20251124`) lets Claude `zoom` into a `[x1,y1,x2,y2]` region at full resolution to read small text/labels — use it when clicks land near but miss small targets.
- **Resolution guidance:** try **1280×720** as a baseline. **macOS Retina** screenshots come at 2× device-pixel-ratio — downscale by 2× before sending, or halve the coordinates Claude returns.
- **Model choice affects precision:** Sonnet 4.6 is mechanically precise and robust under heavy downscaling; Opus 4.7+ narrows the gap thanks to the higher resolution limit.
- Pairs with the schema-less **bash** (`bash_20250124`) and **text-editor** (`text_editor_20250728`) tools. Full reference + agent loop: [computer use tool docs](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool).

---

## 12. Multilingual

Claude generates and understands many languages and handles cross-lingual tasks (translate, summarize-in-another-language, code-switching). There is **no toggle** — just prompt in or request the target language. Token counts vary by language (non-English and code tokenize to more tokens than English), so re-baseline cost with `count_tokens` per language rather than assuming an English ratio. WARN: verify the current list of officially benchmarked languages on the model overview page.

---

## 13. Streaming

### What it is

Server-Sent Events deliver the response incrementally (`message_start`, `content_block_start/delta/stop`, `message_delta`, `message_stop`). **Required** for large `max_tokens` (>~16K) to avoid SDK HTTP timeouts — Fable 5/Opus 4.6/4.7/4.8 stream up to 128K output; Sonnet 4.6/Haiku 4.5 cap at 64K.

### Turn it on — API

```python
with client.messages.stream(model="claude-opus-4-8", max_tokens=64000,
        messages=[{"role":"user","content":"Write a long story"}]) as stream:
    for event in stream:
        if event.type == "content_block_delta" and event.delta.type == "text_delta":
            print(event.delta.text, end="", flush=True)
    final = stream.get_final_message()   # .finalMessage() in TS
```

Default behavior in Claude.ai and Claude Code is to stream tokens to the UI. **Fine-grained tool streaming** is not a beta — set `eager_input_streaming: true` on the tool and use the regular stream call.

---

## 14. Token counting

### What it is

`POST /v1/messages/count_tokens` returns the exact token count for a prompt **on a specific model** (counts are model-specific). Use it for cost estimates, context budgeting, and re-baselining when migrating models. **Do not use `tiktoken`** — it is OpenAI's tokenizer and undercounts Claude by ~15–20% (more on code/non-English).

### Turn it on — API

```python
n = client.messages.count_tokens(
    model="claude-opus-4-8",
    messages=[{"role": "user", "content": open("CLAUDE.md").read()}],
).input_tokens
```

CLI: `ant messages count-tokens --model claude-opus-4-8 --message '{role: user, content: "@./CLAUDE.md"}' --transform input_tokens -r`. In Claude Code, `/context` and the status line show live token usage. Fable 5 uses the **same tokenizer as Opus 4.8** (counts roughly unchanged from Opus 4.7/4.8); the Opus 4.7 tokenizer produces ~1×–1.35× the tokens of Opus 4.6 and earlier — re-measure when migrating.

---

## 15. Refusals & safety stops (handle before reading content)

Safety classifiers (notably on Fable 5, targeting research bio + most cyber content) can decline a request: **HTTP 200** with `stop_reason: "refusal"` and a `stop_details` object (`type: "refusal"`, `category` e.g. `"cyber"`, `"bio"`, `"reasoning_extraction"`, `"frontier_llm"`, or `null`, plus an `explanation`). Pre-output refusals have empty `content` and aren't billed at all (no input/output tokens, no rate-limit consumption); mid-stream refusals bill the partial — discard it. Note benign adjacent work (security tooling, life-sciences) can trigger false positives, which is why fallbacks matter. **Always check `stop_reason` before `response.content[0]`.** `stop_details` is `null` for every other stop reason — guard before reading `.category`.

```python
resp = client.messages.create(model="claude-fable-5", max_tokens=1024, messages=[...])
if resp.stop_reason == "refusal":
    handle_refusal(resp.stop_details)
else:
    print(resp.content[0].text)
```

New Fable 5 code should opt into fallbacks by default — server-side `fallbacks=[{"model":"claude-opus-4-8"}]` with `betas=["server-side-fallback-2026-06-01"]` re-serves a declined request on the fallback model in the same call.

---

## 16. Capability availability matrix (quick reference)

| Capability | API param / surface | Claude.ai | Claude Code |
|---|---|---|---|
| Adaptive thinking | `thinking: {type:"adaptive"}` | Extended toggle | model + effort |
| Effort | `output_config: {effort}` | (implicit) | `/model-config` |
| Fast mode | `speed:"fast"` (beta) | n/a | `/fast` (`↯`) |
| 1M context | model-dependent | 200K (Ent. larger) | yes (model) |
| Vision / PDF | `image`/`document` blocks | attach | `read` tool |
| Tool use | `tools`, `tool_choice` | Connectors/Skills | built-in + MCP |
| Structured outputs | `output_config: {format}` | n/a | via API |
| Prompt caching | `cache_control` | automatic | automatic |
| Batch API | `messages.batches` | n/a | n/a |
| Citations | `citations:{enabled}` | automatic | via API |
| Computer use | `computer_20251124` (beta) | n/a | via API/agents |
| Streaming | `messages.stream` | default | default |
| Token counting | `count_tokens` | n/a | `/context` |

---

## 17. Rate limits & usage tiers

### What it is

Two independent ceilings govern API usage, both at the **organization** level: a **spend limit** (max monthly cost) and **rate limits** (max requests/tokens per minute). This section covers the rate limits — for the per-model token-cost math see [./model-families.md](./model-families.md).

Rate limits are measured **per model class** in three dimensions, replenished continuously by a token-bucket algorithm (not reset on a fixed clock):

| Limit | Meaning | Notes |
|---|---|---|
| **RPM** | Requests per minute | Enforced over short windows too — a 60 RPM cap behaves like ~1/sec, so bursts can 429 even under the per-minute number. |
| **ITPM** | Input tokens per minute | **Cache-aware: only `input_tokens` + `cache_creation_input_tokens` count; `cache_read_input_tokens` do *not*** (except Haiku 3.5). Prompt caching raises effective ITPM — 80% cache hit ≈ 5× throughput. |
| **OTPM** | Output tokens per minute | Counted in real time as tokens generate; `max_tokens` does **not** pre-charge OTPM, so a high `max_tokens` costs nothing until tokens are actually produced. |

Limits apply **separately per model**, so you can run Opus, Sonnet, and Haiku each up to its own ceiling simultaneously. The Opus limit is **shared across all Opus 4.x versions** (4.8/4.7/4.6/4.5/4.1); the Sonnet limit is shared across Sonnet 4.6/4.5. Rate limits are shared across `inference_geo` values (US and global draw from one pool).

### Usage tiers 1–4

Each tier sets both a monthly spend ceiling and the rate-limit table. **You advance automatically** the moment your cumulative credit purchases (excl. tax) reach the next threshold — there is no application.

| Tier | Credit purchase to reach it | Monthly spend limit |
|---|---|---|
| Tier 1 | $5 | $500 |
| Tier 2 | $40 | $500 |
| Tier 3 | $200 | $1,000 |
| Tier 4 | $400 | $200,000 |
| Monthly Invoicing | (contact sales) | No limit |

Representative rate limits (Tier 1 → Tier 4), to show the scaling — **WARN: verify current numbers** on the [rate-limits docs](https://platform.claude.com/docs/en/api/rate-limits), they change:

| Model class | RPM (T1→T4) | ITPM (T1→T4) | OTPM (T1→T4) |
|---|---|---|---|
| Opus 4.x | 50 → 4,000 | 500K → 10M | 80K → 800K |
| Sonnet 4.x | 50 → 4,000 | 30K → 2M | 8K → 400K |
| Haiku 4.5 | 50 → 4,000 | 50K → 4M | 10K → 800K |
| Fable 5 | 50 → 4,000 | 100K → 4M | 20K → 800K |

> **Claude Platform on AWS** orgs start at Tier 1 and do **not** auto-advance — increases go through your account rep; spend limits and per-workspace limits don't apply there. Batch API, Managed Agents, and Fast mode each have their **own** separate rate-limit pools (Managed Agents: 300 RPM create / 600 RPM read per org; Fast mode reports `anthropic-fast-*` headers — see §3).

### The `anthropic-ratelimit-*` response headers

Every response carries headers reporting the enforced limit, current remaining, and reset time (RFC 3339). The `*-limit` / `*-remaining` / `*-reset` triple exists for each of: `requests`, `tokens` (combined), `input-tokens`, and `output-tokens`.

| Header (one of `-limit` / `-remaining` / `-reset`) | Reports |
|---|---|
| `anthropic-ratelimit-requests-*` | RPM limit / remaining / reset |
| `anthropic-ratelimit-tokens-*` | Most-restrictive token limit currently in effect (workspace or org) |
| `anthropic-ratelimit-input-tokens-*` | ITPM limit / remaining (rounded to nearest 1K) / reset |
| `anthropic-ratelimit-output-tokens-*` | OTPM limit / remaining (rounded to nearest 1K) / reset |
| `anthropic-priority-{input,output}-tokens-*` | Priority Tier limits — present **only** on Priority-eligible orgs (see §18) |

The `anthropic-ratelimit-tokens-*` headers always show the **tightest** binding limit (e.g. a workspace cap if you've hit it), so they tell you which constraint actually applies right now.

### 429 + `retry-after` handling

Exceeding any limit returns **HTTP 429** (`rate_limit_error`) with a `retry-after` header (seconds to wait — earlier retries fail). The SDKs **auto-retry 429 and 5xx with exponential backoff** (default `max_retries=2`), honoring `retry-after`, so most apps need no custom logic. A sudden traffic spike can also 429 on **acceleration limits** even below your tier ceiling — ramp gradually.

```python
# Reading the headers via .with_raw_response (Python)
raw = client.messages.with_raw_response.create(
    model="claude-opus-4-8", max_tokens=1024,
    messages=[{"role": "user", "content": "Hi"}],
)
h = raw.headers
print(h["anthropic-ratelimit-input-tokens-remaining"])   # e.g. "498000"
print(h["anthropic-ratelimit-output-tokens-reset"])       # RFC 3339 timestamp
msg = raw.parse()   # the Message object create() would have returned

# On a 429 the SDK already backs off; to read retry-after yourself:
import anthropic
try:
    client.messages.create(model="claude-opus-4-8", max_tokens=1024, messages=[...])
except anthropic.RateLimitError as e:
    wait = int(e.response.headers.get("retry-after", "60"))
```

To read configured limits without making a Messages call, use the **Rate Limits API** (`/docs/en/manage-claude/rate-limits-api`). WARN: verify the Rate Limits API endpoint path for your account.

---

## 18. Service tier / Priority Tier

### What it is

The `service_tier` request parameter selects which capacity pool serves a request, trading availability/predictability against your committed capacity. **Priority Tier** prioritizes your requests over standard traffic to minimize "server overloaded" (`529`) errors during peak times, targeting **99.5% uptime**.

> **Status (verify):** As of mid-2026 the docs state **Priority Tier capacity commitments are no longer available for new purchase** — existing commitments are honored through their contract end date. Treat Priority Tier as available **only to orgs with an existing commitment**; new orgs cannot opt in. WARN: verify current Priority Tier availability before relying on it.

### The `service_tier` request parameter

| Value | Behavior |
|---|---|
| `"auto"` (**default**) | Use Priority Tier capacity if available, else fall back to standard. |
| `"standard_only"` | Force standard tier — skip Priority capacity even if you have it (e.g. to preserve commitment for other traffic). |

> Note: `priority` is **not** a request value you send — it is what the API *reports back* in `usage.service_tier` when a request was served by Priority capacity. You request `auto` or `standard_only`; the server decides and tells you which tier it used. (Some older references describe a `priority` request value; the current parameter accepts only `auto` / `standard_only`. WARN: verify against the [service tiers docs](https://platform.claude.com/docs/en/api/service-tiers).)

```python
msg = client.messages.create(
    model="claude-opus-4-8", max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude!"}],
    service_tier="auto",          # or "standard_only"
)
print(msg.usage.service_tier)     # "priority" or "standard" — which pool served it
```

### How a request is assigned Priority

A request is served by Priority Tier when the org has sufficient Priority **input** *and* **output** tokens-per-minute capacity remaining; otherwise it proceeds at standard. Priority capacity burns down at **rates that mirror pricing**: cache reads 0.1×, 5-min cache writes 1.25×, 1-hour cache writes 2×, and `inference_geo: "us"` requests on Opus 4.6 / Sonnet 4.6+ at 1.1× (both input and output). A Priority-assigned request **also draws from your normal rate limits** — if it would exceed those, it is declined regardless of Priority capacity.

### Priority headers & pricing implication

On a Priority-eligible org, `service_tier="auto"` responses carry `anthropic-priority-input-tokens-*` and `anthropic-priority-output-tokens-*` headers (limit / remaining / reset) — their **presence alone** tells you the request was Priority-eligible, even if it was over the limit. A commitment specifies input-TPM, output-TPM, a duration (1/3/6/12 months), and a **specific model version**; requests beyond committed capacity fall back to standard. Priority Tier is supported on all current models **including Fable 5 and Opus 4.8**, except Mythos Preview / Mythos 5. For the per-model pricing that the burndown rates mirror, see [./model-families.md](./model-families.md). (Fast mode is **not** available on Priority Tier — see §3.)

---

## 19. Consolidated `stop_reason` table

Every Messages response carries a `stop_reason`. **Always branch on it before reading `content[0]`** — a refused or context-exceeded response may have empty or partial content. Only `refusal` populates `stop_details` (it is `null` for all others — guard before reading `.category`).

| `stop_reason` | Meaning | How to handle |
|---|---|---|
| `end_turn` | Claude finished naturally. | Read the content; done. |
| `max_tokens` | Hit the `max_tokens` cap mid-generation. | Output is truncated — raise `max_tokens` (stream above ~16K) and/or continue. |
| `stop_sequence` | Hit one of your custom `stop_sequences`. | The matched sequence is in `stop_sequence`; output ends before it. |
| `tool_use` | Claude emitted one or more `tool_use` blocks. | Execute the tools, return `tool_result` blocks in **one** user message, loop. |
| `pause_turn` | A **server-side** tool loop (web search, code exec) hit its iteration cap. | Re-send the conversation (user msg + assistant `response.content`) to resume — do **not** add a "continue" message. |
| `refusal` | Safety classifiers or the model declined (HTTP 200). | Check `stop_details` (`category`, `explanation`); pre-output refusals have empty `content` and aren't billed; mid-stream ones bill the partial — discard it. See §15. On Fable 5, opt into `fallbacks`. |
| `model_context_window_exceeded` | Generation hit the **context window** (not the `max_tokens` cap) on Claude 4.5+ models. | Distinct from `max_tokens` — compact/edit context or split the conversation; raising `max_tokens` won't help. |

> **Forward-compatibility:** treat the set as open — handle unknown `stop_reason` values as "stop and inspect" rather than asserting on a fixed list, since new reasons appear with new models. WARN: verify any additional stop reasons (e.g. fallback-related) on the [handling-stop-reasons](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons) docs.

---

## 20. Beta-header reference table

A consolidated index of the `anthropic-beta` headers referenced across this page. Pass them via the SDK `betas=[...]` argument on `client.beta.messages.*` (or the raw `anthropic-beta` header). **Beta headers change** — treat dates as load-bearing and verify before pinning; a feature graduating to GA means dropping the header and the `client.beta.*` path.

| Feature (section) | Beta header | Status |
|---|---|---|
| Fast mode (§3) | `fast-mode-2026-02-01` | Beta (research preview); first-party API only, Opus 4.8/4.7/4.6 |
| Task budgets (§2) | `task-budgets-2026-03-13` | Beta; Fable 5 / Opus 4.8 / 4.7 |
| Computer use (§11) | `computer-use-2025-11-24` | Beta (GA-track); current models. Older: `computer-use-2025-01-24` |
| Files API (§5) | `files-api-2025-04-14` | Beta; required on **both** upload and the referencing `messages.create` |
| Server-side fallback (§15) | `server-side-fallback-2026-06-01` | Beta; Fable 5 refusal recovery, first-party + Claude Platform on AWS |
| Compaction (frontmatter / §4) | `compact-2026-01-12` | Beta; server-side history summarization, Fable 5 / Opus 4.8/4.7/4.6 / Sonnet 4.6 |
| Context editing (frontmatter / §4) | `context-management-2025-06-27` | Beta; clears stale tool results / thinking (distinct from compaction) |

> **No `output-300k` (or `output-128k`) header exists.** 128K output is **native** on Fable 5 / Opus 4.6/4.7/4.8 — just set `max_tokens` and stream; there is no opt-in beta header for long output. (The legacy `output-128k-2025-02-19` header is a no-op on Claude 4+.) WARN: verify if any larger-output beta ships later. Likewise the **1M context window needs no beta header** on current models (the legacy `context-1m-2025-08-07` header is not required — see §4).

---

## Related pages

- [./model-families.md](./model-families.md) — every model, its IDs, pricing, and per-model capability support
- [../claude-code/slash-commands.md](../claude-code/slash-commands.md) — `/fast`, `/model`, `/model-config`, `/context`, `/goal`
- [../claude-code/cli-and-shortcuts.md](../claude-code/cli-and-shortcuts.md) — `--model` flag, effort selection, keyboard shortcuts
- [../claude-code/settings.md](../claude-code/settings.md) — `fastMode`, `fastModePerSessionOptIn`, permissions, env vars
- [../platform/claude-ai.md](../platform/claude-ai.md) — Claude.ai thinking toggle, model picker, context size by plan
- [../capabilities/mcp.md](../capabilities/mcp.md) — server tools via MCP, tool-list cache stability
- [../capabilities/connectors.md](../capabilities/connectors.md) — connectors as the Claude.ai tool surface
- [../capabilities/skills.md](../capabilities/skills.md) — Agent Skills (progressive-disclosure capability loading)
- [../capabilities/artifacts.md](../capabilities/artifacts.md) — where structured/code output is rendered in Claude.ai
- [../glossary.md](../glossary.md) — term definitions

## Open questions / to verify

- Exact Claude.ai menu label for the thinking toggle ("Extended" vs "Thinking") and whether it appears per-model. (Consumer-UI label — not in the API docs.)
- Current Claude.ai context-window sizes by plan, including the Enterprise figure (≈500K?). (Consumer-surface, changes frequently.)
- Current "think"/"ultrathink" trigger-word behavior in Claude Code on adaptive-thinking models.
- The official list of benchmarked supported languages for multilingual.
- Current **usage-tier rate-limit numbers** (RPM/ITPM/OTPM per model, §17) — these scale frequently; the table shows representative T1→T4 ranges. Re-verify against the [rate-limits docs](https://platform.claude.com/docs/en/api/rate-limits).
- Current **Priority Tier availability** (§18) — docs state new capacity commitments are no longer purchasable (existing ones honored). Confirm whether this has changed and whether `service_tier` accepts any value beyond `auto` / `standard_only`.
- Whether any **larger-output beta** (beyond native 128K) ships — there is currently **no** `output-300k`/`output-128k` opt-in header (§20); 128K output is native on Claude 4+.

**Resolved this pass (now stated as fact in-body):** 1M needs no beta header on current models · no long-context price premium on Opus 4.8/4.7 · computer-use tool is `computer_20251124` + header `computer-use-2025-11-24` · high-res vision = 2576 px / 4784 tokens on Fable 5/Mythos 5/Opus 4.8/4.7 · Claude.ai = 20 images/turn · Fast mode pricing & toggles · **rate limits, usage tiers 1–4, and the `anthropic-ratelimit-*` headers (§17)** · **`service_tier` parameter (`auto`/`standard_only`) and the `anthropic-priority-*` headers (§18)** · **consolidated `stop_reason` table (§19)** · **beta-header reference table (§20)**.

## Sources

- [Extended thinking](https://platform.claude.com/docs/en/build-with-claude/extended-thinking) · [Adaptive thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking) · [Effort](https://platform.claude.com/docs/en/build-with-claude/effort)
- [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows)
- [Vision](https://platform.claude.com/docs/en/build-with-claude/vision) · [Coordinates & bounding boxes](https://platform.claude.com/docs/en/build-with-claude/vision-coordinates) · [PDF support](https://platform.claude.com/docs/en/build-with-claude/pdf-support)
- [Tool use overview](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview) · [Computer use tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/computer-use-tool)
- [Structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)
- [Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) · [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing) · [Citations](https://platform.claude.com/docs/en/build-with-claude/citations)
- [Streaming](https://platform.claude.com/docs/en/build-with-claude/streaming) · [Token counting](https://platform.claude.com/docs/en/build-with-claude/token-counting)
- [Fast mode — Claude Code](https://code.claude.com/docs/en/fast-mode) · [Fast mode — API](https://platform.claude.com/docs/en/build-with-claude/fast-mode)
- [Using extended thinking (Help Center)](https://support.claude.com/en/articles/10574485-using-extended-thinking)
- [Rate limits](https://platform.claude.com/docs/en/api/rate-limits) · [Service tiers](https://platform.claude.com/docs/en/api/service-tiers) · [Handling stop reasons](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons)
