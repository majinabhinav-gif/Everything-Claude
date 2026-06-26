---
type: Model Reference
title: Claude Model Families
description: Exhaustive reference for every current Claude model — IDs, context windows, output limits, knowledge cutoffs, pricing (with cache and batch levers), and availability surfaces. Prices/specs verified against platform.claude.com on 2026-06-26.
domain: models
tags: [models, model-ids, pricing, context-window, fable-5, opus-4-8, sonnet-4-6, haiku-4-5, prompt-caching, batch-api, bedrock, vertex]
related: [capabilities-and-modes, claude-ai, claude-code-overview, projects]
resource: https://platform.claude.com/docs/en/about-claude/models/overview
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources: [https://platform.claude.com/docs/en/about-claude/models/overview, https://platform.claude.com/docs/en/about-claude/pricing, https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5, https://www.anthropic.com/news/claude-fable-5-mythos-5, https://www.anthropic.com/news/claude-opus-4-8, https://platform.claude.com/docs/en/about-claude/models/migration-guide, https://platform.claude.com/docs/en/about-claude/model-deprecations, https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions, https://platform.claude.com/docs/en/api/models/list, https://platform.claude.com/docs/en/api/service-tiers, https://platform.claude.com/docs/en/build-with-claude/embeddings]
---

# Claude Model Families

This is the authoritative table of every Claude model you can call today: the exact model-ID string (these are known and stable — use them verbatim), the context window and max output, the knowledge cutoff, the per-million-token pricing with cache and batch cost levers, and which platforms serve each one. Anthropic's lineup spans four tiers — **Fable/Mythos** (most capable, "Mythos-class"), **Opus** (flagship reasoning/agentic), **Sonnet** (speed + intelligence balance), and **Haiku** (fastest/cheapest) — plus a long tail of legacy and retired snapshots. Model IDs are precise and reliable; prices and limits below were verified against [platform.claude.com](https://platform.claude.com/docs/en/about-claude/pricing) on **2026-06-26** but are still worth re-confirming before you build a cost model on them.

> **At a glance**
> - **What it is:** The complete catalog of callable Claude models — IDs, windows, limits, cutoffs, prices, and surfaces.
> - **Where you find it:** `model=` in any Messages API request; the model picker in Claude apps and Claude Code; `GET /v1/models` for live capability data.
> - **Who can use it by plan:** API/Console (pay-as-you-go + tiers); Claude apps (Free/Pro/Max/Team/Enterprise gate which models appear); Bedrock / Vertex AI / Microsoft Foundry / Claude Platform on AWS via those accounts.
> - **Status:** Verified 2026-06-26. Fable 5 went GA 2026-06-09. Opus 4.8 is the recommended flagship default. Pricing/limits change — re-verify the specifics.

---

## The current lineup at a glance

| Model | Model ID (use verbatim) | Alias | Context | Max output | Knowledge cutoff (reliable / training) | Tier |
|---|---|---|---|---|---|---|
| Claude Fable 5 | `claude-fable-5` | (ID is dateless; it is itself a pinned snapshot) | 1M | 128K | not published — WARN: verify | Fable / "Mythos-class" |
| Claude Mythos 5 | `claude-mythos-5` | — (Project Glasswing only) | 1M | 128K | not published — WARN: verify | Mythos-class |
| Claude Opus 4.8 | `claude-opus-4-8` | dateless ID = pinned snapshot | 1M (200K on Microsoft Foundry) | 128K | Jan 2026 / Jan 2026 | Opus (flagship) |
| Claude Opus 4.7 | `claude-opus-4-7` | dateless ID = pinned snapshot | 1M | 128K | Jan 2026 / Jan 2026 | Opus (legacy) |
| Claude Opus 4.6 | `claude-opus-4-6` | dateless ID = pinned snapshot | 1M | 128K | May 2025 / Aug 2025 | Opus (legacy) |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | dateless ID = pinned snapshot | 1M | 64K (300K via batch beta) | Aug 2025 / Jan 2026 | Sonnet |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | `claude-haiku-4-5` | 200K | 64K | Feb 2025 / Jul 2025 | Haiku |

> **Addressing the 1M context window — two surfaces, one model.** On the **public Claude API**, Opus 4.8 has exactly one model ID — `claude-opus-4-8` (per the [model IDs & versioning](https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions) and [overview](https://platform.claude.com/docs/en/about-claude/models/overview) docs) — and it **already serves the full 1M-token context window at standard pricing** (no long-context premium; see [Pricing levers](#pricing-levers-the-cost-knobs)). There is no separate `[1m]` *API SKU*; the public dateless scheme is `claude-{name}-{major}-{minor}` with no bracketed suffix. **In Claude Code / the agent runtime, however, `claude-opus-4-8[1m]` IS a real, addressable model handle** — the explicit selector for the 1M-context variant of Opus 4.8 (it is the exact model string this environment reports running). So: treat `[1m]` as a **runtime / Claude Code variant handle**, not a public-API SKU. Pass `claude-opus-4-8` to `client.messages.create()` (you get the full 1M window either way); expect `claude-opus-4-8[1m]` wherever Claude Code surfaces the explicit long-context variant. (Historically, the 1M opt-in on *older* models was a `context-1m-2025-08-07` beta *header*, not an ID suffix.)

> **Naming convention (verified):** Starting with the **4.6 generation**, model IDs use a **dateless format** (`claude-opus-4-8`, `claude-sonnet-4-6`) that is itself a **pinned snapshot**, not an evergreen pointer — Anthropic never updates the weights behind an existing ID; a new version ships under a new ID. Older models (pre-4.6) use dated IDs (`claude-haiku-4-5-20251001`) and expose a dateless **alias** (`claude-haiku-4-5`) that resolves to the most recent dated snapshot. The dateless 4.6+ IDs are **not** aliases — they are the snapshot. The docs do not define a dated form of a dateless ID, so do not invent one (e.g., `claude-opus-4-8-20260601` is not a documented ID). Note: serving infrastructure (router, safety classifiers, sampling logic) can change under a fixed ID, occasionally producing minor behavioral drift even though the weights are unchanged.

---

## Per-model detail tables

### Claude Fable 5 — `claude-fable-5`

Anthropic's **most capable widely released model** and the first publicly available "Mythos-class" model — a tier *above* Opus. Built for the most demanding reasoning and long-horizon agentic work (overnight coding runs, end-to-end enterprise deliverables, deep research). GA on **2026-06-09**. Priced at 2× Opus-tier ($10/$50), so it is **not** the default "upgrade to the latest" target — reach for it only when the task genuinely needs frontier capability.

| Attribute | Value |
|---|---|
| Model ID | `claude-fable-5` (dateless; itself a pinned snapshot) |
| Context window | 1M tokens (default; 1M is also the maximum) |
| Max output | 128K tokens per request |
| Tokenizer | Same as Opus 4.7/4.8 — vs pre-4.7 models, the same text is **~30–35% more tokens** (overview: ~30%; pricing page: up to 35%) |
| Thinking | **Adaptive only, always on.** Omit the `thinking` param; `thinking:{type:"disabled"}` is **not supported**. Use `effort` to control depth. Raw chain-of-thought is never returned — `thinking.display` is `"omitted"` (default, empty `thinking` field) or `"summarized"` (readable summary). Pass thinking blocks back unchanged on the same model |
| Effort | The `effort` parameter is supported (control thinking depth/cost). WARN: verify the exact level set for Fable — the Models API exposes `low/medium/high/xhigh/max` for Opus-tier, but the Fable launch doc does not enumerate Fable's levels |
| Vision | Yes |
| Other supported features (at launch) | Task budgets (beta `task-budgets-2026-03-13`), the memory tool, code execution, programmatic tool calling, tool-result clearing via context editing (beta `context-management-2025-06-27`), and compaction |
| Prefill / sampling params | WARN: verify — the Fable launch doc does not document last-turn prefill or `temperature`/`top_p`/`top_k`. (The deprecation table documents `temperature`/`top_p`/`top_k` returning 400 on **Opus 4.7 and later**; it does not name Fable, so do not assume the same 400 behavior without checking) |
| Refusals | **Fable 5 includes safety classifiers** that can decline a request: the API returns HTTP 200 with `stop_reason:"refusal"` (not an error) and reports which classifier declined. Check `stop_reason` before reading `content`. **Mythos 5 has no such classifiers** |
| Refusal billing | You are **not billed** for a request refused **before any output** is generated. On retry to another model, [fallback credit](https://platform.claude.com/docs/en/build-with-claude/fallback-credit) refunds the prompt-cache cost so you don't pay it twice |
| Data retention | **30-day retention; not available under ZDR** (both Fable 5 and Mythos 5 are designated Covered Models). ZDR orgs cannot use them |
| Pricing (per MTok) | **$10 input / $50 output** |
| Cache writes | $12.50 (5-min) / $20 (1-hour) per MTok |
| Cache read (hit) | $1.00 per MTok |
| Batch (50% off) | $5 input / $25 output per MTok |
| Availability | GA on Claude API, Claude Platform on AWS, Amazon Bedrock (`anthropic.claude-fable-5`), Google Cloud/Vertex (`claude-fable-5`), Microsoft Foundry |

WARN: verify Fable 5's knowledge/training cutoff against the official model page — the overview comparison table publishes cutoff rows for Opus 4.8 / Sonnet 4.6 / Haiku 4.5 but **omits them for Fable 5 and Mythos 5**.

```python
# Fable 5: adaptive thinking is implicit (always on); control depth via `effort`.
# Ship a server-side fallback so a classifier decline can be retried on another model.
# (server-side fallback is in beta on the Claude API and Claude Platform on AWS.)
resp = client.beta.messages.create(
    model="claude-fable-5",
    max_tokens=16000,
    effort="high",                                   # control thinking depth/cost
    betas=["server-side-fallback-2026-06-01"],       # verified beta header
    fallbacks=[{"model": "claude-opus-4-8"}],        # API retries here on a refusal
    messages=[{"role": "user", "content": "Plan and implement the migration."}],
)
if resp.stop_reason == "refusal":
    handle_refusal()      # no output was billed; `fallbacks` may have already retried
else:
    print(resp.content[0].text)
```

### Claude Mythos 5 — `claude-mythos-5`

Same capabilities, pricing, and limits as Fable 5 — but **Mythos 5 omits Fable 5's safety classifiers** (so it does not return classifier-driven `stop_reason:"refusal"`). Available **exclusively through [Project Glasswing](https://anthropic.com/glasswing)** (limited availability to approved customers; participation is the only way to access it). It joins and succeeds the invitation-only **Claude Mythos Preview** (`claude-mythos-preview`), a research-preview model for defensive cybersecurity. Use `claude-mythos-5` only if your org participates in Project Glasswing; otherwise use `claude-fable-5` (same capabilities, GA).

| Attribute | Value |
|---|---|
| Model ID | `claude-mythos-5` (Project Glasswing limited availability only) |
| Context / output | 1M / 128K (same as Fable 5) |
| Classifiers | **None** — no classifier-driven refusals (the one capability difference vs Fable 5) |
| Pricing | $10 input / $50 output per MTok; cache & batch identical to Fable 5 |
| Data retention | 30-day; not available under ZDR (Covered Model) |
| Predecessor | `claude-mythos-preview` (invitation-only; **retires 2026-06-30** → migrate to `claude-mythos-5` or `claude-fable-5`) |
| Availability | Project Glasswing only (limited) — contact your Anthropic / AWS / Google Cloud account team |
| Source (verified 2026-06-26) | Confirmed on the official launch doc — [Introducing Claude Fable 5 and Claude Mythos 5](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5) lists `claude-mythos-5` as "Shares Claude Fable 5's capabilities without the safety classifiers. Available through Project Glasswing. Successor to Claude Mythos Preview." (also: [anthropic.com/news/claude-fable-5-mythos-5](https://www.anthropic.com/news/claude-fable-5-mythos-5)) |

### Claude Opus 4.8 — `claude-opus-4-8` (recommended default)

The **most capable Opus-tier model** and the recommended default for complex reasoning, long-horizon agentic coding, and high-autonomy work. Around **4× less likely** than its predecessor to miss a code flaw. Same request surface as Opus 4.7 (no new breaking changes) — a 4.7→4.8 move is a model-ID swap plus prompt re-tuning.

| Attribute | Value |
|---|---|
| Model ID | **Public API:** `claude-opus-4-8` (dateless; pinned snapshot) — already serves the full 1M window at standard pricing. **Claude Code / runtime:** also exposes `claude-opus-4-8[1m]` as an explicit 1M-context variant handle for the same model (see the [note above](#the-current-lineup-at-a-glance)) |
| Context window | 1M tokens (**200K on Microsoft Foundry** — verified footnote on the overview page) |
| Max output | 128K tokens (300K via Message Batches API beta `output-300k-2026-03-24`) |
| Knowledge cutoff | Reliable: **Jan 2026** · Training data: **Jan 2026** |
| Thinking | **Adaptive thinking only** (extended thinking: No). `effort` controls depth |
| Effort | `effort` **defaults to `high` on all surfaces** (Claude API and Claude Code); set it explicitly for a different level. Supported levels (per Models API): `low` / `medium` / `high` / `xhigh` (Claude Code labels this "extra") / `max` |
| Sampling params | `temperature` / `top_p` / `top_k` return **400** when set to a non-default value (deprecated on Opus 4.7+); omit them and steer via prompting |
| Vision | Yes (high-resolution) |
| Fast mode | Yes (research preview) — 2.5× faster output; see [Pricing levers](#pricing-levers-the-cost-knobs). Not on Claude Platform on AWS |
| Mid-conversation system messages | **Opus 4.8 only** — the Messages API accepts `role:"system"` entries inside `messages[]` to update instructions mid-task **without breaking the prompt cache** |
| Pricing (per MTok) | **$5 input / $25 output** |
| Cache writes | $6.25 (5-min) / $10 (1-hour) per MTok |
| Cache read (hit) | $0.50 per MTok |
| Batch (50% off) | $2.50 input / $12.50 output per MTok |
| Lifecycle | Active. Tentative retirement: **not sooner than May 28, 2027** |
| Availability | Claude API, Claude Platform on AWS (`claude-opus-4-8`), Amazon Bedrock (`anthropic.claude-opus-4-8`), Google Cloud/Vertex (`claude-opus-4-8`), Microsoft Foundry (200K context) |

### Claude Opus 4.7 — `claude-opus-4-7`

Previous-generation Opus. Highly autonomous; strong on long-horizon agentic work, knowledge work, vision, and memory. Introduced the current tokenizer (~30–35% more tokens), the adaptive-thinking-only surface, the `xhigh` effort level, and the removal of sampling params. No Fast Mode price break vs 4.6 (still $30/$150 fast).

| Attribute | Value |
|---|---|
| Model ID | `claude-opus-4-7` (dateless; pinned snapshot) |
| Context / Max output | 1M / 128K tokens (300K output via batch beta) |
| Knowledge cutoff | Reliable: Jan 2026 · Training: Jan 2026 |
| Thinking | Adaptive only (extended thinking: No); `temperature`/`top_p`/`top_k` return 400 when non-default |
| Pricing (per MTok) | $5 input / $25 output; cache $6.25/$10 write, $0.50 read; batch $2.50/$12.50 |
| Fast mode | Yes — Opus 4.6/4.7 fast: **$30 input / $150 output** per MTok |
| Lifecycle | Active. Tentative retirement: not sooner than April 16, 2027 |
| Availability | Claude API, Claude Platform on AWS, Amazon Bedrock (`anthropic.claude-opus-4-7`), Google Cloud (`claude-opus-4-7`), Microsoft Foundry |

### Claude Opus 4.6 — `claude-opus-4-6`

Older Opus. Supports **both extended thinking and adaptive thinking** (the last Opus to keep extended thinking). 128K max output. Last Bedrock ID to carry the `-v1` suffix.

| Attribute | Value |
|---|---|
| Model ID | `claude-opus-4-6` (dateless; pinned snapshot) |
| Context / Max output | 1M / 128K tokens (300K output via batch beta) |
| Knowledge cutoff | Reliable: **May 2025** · Training: **Aug 2025** |
| Thinking | Adaptive (recommended) **or** extended thinking (`thinking:{type:"enabled"}`) |
| Pricing (per MTok) | $5 input / $25 output; cache $6.25/$10 write, $0.50 read; batch $2.50/$12.50 |
| Fast mode | Yes — $30 input / $150 output per MTok |
| Lifecycle | Active. Tentative retirement: not sooner than February 5, 2027 |
| Availability | Claude API, Claude Platform on AWS, Amazon Bedrock (`anthropic.claude-opus-4-6-v1` — **last `-v1` Bedrock ID**), Google Cloud (`claude-opus-4-6`), Microsoft Foundry |

### Claude Sonnet 4.6 — `claude-sonnet-4-6`

The best **combination of speed and intelligence** — the workhorse for most production workloads. Supports **both adaptive and extended thinking**; 1M context; 64K output. `effort` defaults to `high`, so set it explicitly (often `medium` or `low`) when migrating from Sonnet 4.5 to control latency/cost.

| Attribute | Value |
|---|---|
| Model ID | `claude-sonnet-4-6` (dateless; pinned snapshot) |
| Context window | 1M tokens |
| Max output | 64K tokens (300K via Message Batches API beta `output-300k-2026-03-24`) |
| Knowledge cutoff | Reliable: **Aug 2025** · Training: **Jan 2026** |
| Thinking | Adaptive (recommended) **or** extended thinking; supports `effort` up to `max` |
| Sampling params | `temperature`/`top_p`/`top_k` 400 on non-default (4.7+ behavior also applies here per the parameter-deprecation table) |
| Pricing (per MTok) | **$3 input / $15 output** |
| Cache writes | $3.75 (5-min) / $6 (1-hour) per MTok |
| Cache read (hit) | $0.30 per MTok |
| Batch (50% off) | $1.50 input / $7.50 output per MTok |
| Lifecycle | Active. Tentative retirement: not sooner than February 17, 2027 |
| Availability | Claude API, Claude Platform on AWS, Amazon Bedrock (`anthropic.claude-sonnet-4-6` — no `-v1`), Google Cloud (`claude-sonnet-4-6`), Microsoft Foundry |

### Claude Haiku 4.5 — `claude-haiku-4-5-20251001`

The **fastest model with near-frontier intelligence** — the cost-effective choice for simple/high-volume tasks (classification, extraction, routing, sub-agents). The only current model with a **200K** (not 1M) context window, and the only current model whose canonical ID is **dated** (`-20251001`) with a separate dateless alias.

| Attribute | Value |
|---|---|
| Model ID (full) | `claude-haiku-4-5-20251001` |
| Alias | `claude-haiku-4-5` |
| Context window | 200K tokens |
| Max output | 64K tokens |
| Knowledge cutoff | Reliable: **Feb 2025** · Training: **Jul 2025** |
| Thinking | **Extended thinking: Yes; adaptive thinking: No.** `effort` errors above `high` (no `xhigh`/`max`) |
| Pricing (per MTok) | **$1 input / $5 output** |
| Cache writes | $1.25 (5-min) / $2 (1-hour) per MTok |
| Cache read (hit) | $0.10 per MTok |
| Batch (50% off) | $0.50 input / $2.50 output per MTok |
| Lifecycle | Active. Tentative retirement: not sooner than October 15, 2026 |
| Availability | Claude API, Claude Platform on AWS, Amazon Bedrock (`anthropic.claude-haiku-4-5-20251001-v1:0`), Google Cloud (`claude-haiku-4-5@20251001`), Microsoft Foundry |

> **Vertex ID format quirk (verified):** dated-snapshot models use an `@` version separator on Google Cloud — `claude-haiku-4-5@20251001`, **not** `claude-haiku-4-5-20251001`. Current-generation dateless models (`claude-opus-4-8`, `claude-sonnet-4-6`, `claude-fable-5`) use the bare first-party ID on Vertex. **Bedrock** prefixes everything with `anthropic.`; dated models add `-v1:0` (`anthropic.claude-haiku-4-5-20251001-v1:0`), and Opus 4.6 is the **last** Bedrock ID with a bare `-v1` (`anthropic.claude-opus-4-6-v1`) — Anthropic dropped the suffix from Sonnet 4.6 onward.

---

## Pricing summary (per million tokens, USD)

All numbers verified against the official [pricing page](https://platform.claude.com/docs/en/about-claude/pricing) on 2026-06-26. **Re-verify before relying on them — pricing changes.**

| Model | Input | Output | 5m cache write | 1h cache write | Cache read | Batch in | Batch out |
|---|---|---|---|---|---|---|---|
| Fable 5 | $10 | $50 | $12.50 | $20 | $1.00 | $5 | $25 |
| Mythos 5 | $10 | $50 | $12.50 | $20 | $1.00 | $5 | $25 |
| Opus 4.8 | $5 | $25 | $6.25 | $10 | $0.50 | $2.50 | $12.50 |
| Opus 4.7 | $5 | $25 | $6.25 | $10 | $0.50 | $2.50 | $12.50 |
| Opus 4.6 | $5 | $25 | $6.25 | $10 | $0.50 | $2.50 | $12.50 |
| Sonnet 4.6 | $3 | $15 | $3.75 | $6 | $0.30 | $1.50 | $7.50 |
| Haiku 4.5 | $1 | $5 | $1.25 | $2 | $0.10 | $0.50 | $2.50 |

Cache and batch multipliers are uniform across models: **5-min cache write = 1.25× base input**, **1-hour cache write = 2× base input**, **cache read = 0.1× base input**, **batch = 0.5×** on both input and output. You can derive any cell from the base input/output price.

For reference (legacy/deprecated, still callable on some surfaces): **Opus 4.5** = $5/$25; **Sonnet 4.5** = $3/$15; **Opus 4.1** (deprecated, retires 2026-08-05) = $15/$75.

---

## Pricing levers: the cost knobs

These mechanisms move real money and stack with each other (cache + batch + data-residency multipliers compound).

### 1M context — no long-context premium

Fable 5, Mythos 5 (and Mythos Preview), Opus 4.8, Opus 4.7, Opus 4.6, and Sonnet 4.6 include the **full 1M token context window at standard per-token pricing** — there is **no separate long-context tier or premium**. The docs put it plainly: a 900K-token request is billed at the same per-token rate as a 9K-token request. Prompt-caching and batch discounts apply at standard rates across the entire 1M window. (Haiku 4.5 caps at 200K; Opus 4.8 caps at 200K specifically on Microsoft Foundry.)

### Prompt caching — up to ~90% off repeated context

A cache **read** costs 0.1× base input. With the 5-min TTL (1.25× write) you break even after a single read; with the 1-hour TTL (2× write), after two reads. Two ways to enable: **automatic caching** (one top-level `cache_control` field; the system manages breakpoints as the conversation grows) or **explicit breakpoints** (`cache_control` on individual content blocks).

```python
# Cache a large stable prefix (system prompt / docs) once, read it cheaply thereafter.
resp = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=4096,
    system=[{"type": "text", "text": LARGE_SHARED_PROMPT,
             "cache_control": {"type": "ephemeral"}}],   # add "ttl":"1h" for bursty traffic
    messages=[{"role": "user", "content": "Answer from the cached context."}],
)
# Verify it's working:
resp.usage.cache_creation_input_tokens   # written this request (~1.25x)
resp.usage.cache_read_input_tokens       # served from cache (~0.1x) — should be > 0 on repeats
```

See [../capabilities/projects.md](../capabilities/projects.md) and the modes page for how caching interacts with long-running agents.

### Batch API — 50% off, asynchronous

The [Batch API](https://platform.claude.com/docs/en/build-with-claude/batch-processing) processes large volumes of requests asynchronously at **half price** on both input and output. Not compatible with Fast Mode; results arrive unordered (key by `custom_id`). The same `output-300k-2026-03-24` beta header that unlocks 300K output applies on Opus 4.8/4.7/4.6 and Sonnet 4.6.

### Fast Mode — premium for speed (Opus only)

Fast mode (research preview, beta `fast-mode-2026-02-01`) runs the same Opus model **~2.5× faster** at premium pricing, across the full context window (including >200K input):

| Model | Fast input / MTok | Fast output / MTok |
|---|---|---|
| Opus 4.6 / Opus 4.7 | $30 | $150 |
| Opus 4.8 | $10 | $50 |

Fast mode is **not available on Claude Platform on AWS** and **not available with the Batch API**. Prompt-caching and data-residency multipliers stack on top of fast-mode pricing.

### Priority Tier (`service_tier`) — guaranteed capacity at its own pricing

Anthropic offers a **Priority Tier** that prioritizes your requests over all others to minimize "server overloaded" errors during peak times, targeting **99.5% uptime** — sold as a committed-capacity offering (input + output tokens/min, a 1/3/6/12-month duration, and a specific model version) with its **own pricing** separate from standard pay-as-you-go. You opt in per request via the **`service_tier`** parameter: `"auto"` (default — uses Priority Tier capacity when available, else falls back to standard) or `"standard_only"`; the response `usage.service_tier` reports which tier actually served the request (`"priority"`, `"standard"`, or `"batch"`). See [./capabilities-and-modes.md](./capabilities-and-modes.md) for the request param. **Note (verified 2026-06-26):** new Priority Tier capacity commitments are **no longer available for purchase** — only orgs with an existing commitment can use it (contact your account team for guaranteed capacity). Priority Tier is supported on all current models **except Mythos Preview and Mythos 5**.

### Data residency (`inference_geo`) — 1.1× for US-only

On **Opus 4.6 / Sonnet 4.6 and later**, `inference_geo:"us"` applies a **1.1× multiplier** on all token categories (input, output, cache writes, cache reads). `inference_geo:"global"` (default) is standard pricing. Applies to the **Claude API and Claude Platform on AWS only**; partner clouds (Bedrock, Vertex) use their own regional pricing. Earlier models **400** on the parameter.

---

## Availability surfaces

Every current model runs on five surfaces, with per-surface ID conventions:

| Surface | ID convention | Example (Opus 4.8) | Notes |
|---|---|---|---|
| **Claude API** (first-party) | bare ID | `claude-opus-4-8` | Global by default; full feature surface |
| **Claude apps** (claude.ai / desktop / mobile) | model picker, not an ID | "Opus 4.8" | Which models appear is gated by plan — see [../platform/claude-ai.md](../platform/claude-ai.md) |
| **Claude Platform on AWS** | same IDs as Claude API (no `anthropic.` prefix) | `claude-opus-4-8` | Anthropic-operated; bills in CCUs ($0.01/CCU) via AWS Marketplace; follows Anthropic's first-party deprecation schedule |
| **Amazon Bedrock** | `anthropic.`-prefixed | `anthropic.claude-opus-4-8` | Partner-operated; global vs regional endpoints (regional = +10%); sets its own retirement dates |
| **Google Cloud / Vertex AI** | bare (dateless) or `@`-dated | `claude-opus-4-8` | Global / multi-region / regional endpoints (non-global = +10%); sets its own retirement dates |
| **Microsoft Foundry** | per-Foundry catalog | `claude-opus-4-8` | Beta for several features; **Opus 4.8 context = 200K** on Foundry |

Capabilities differ by surface — for example, Fast Mode and mid-conversation system messages are first-party (and partly Platform-on-AWS) only; partner clouds set their own model lifecycles. Consult Anthropic's per-feature availability matrix before assuming a feature exists on a partner cloud.

### Querying capabilities live (Models API)

Don't hard-code capabilities — ask the API. `GET /v1/models` and `GET /v1/models/{id}` return `max_input_tokens` (context window), `max_tokens` (output cap), and a `capabilities` object with `supported: true/false` leaves. Verified shape (from the [list-models reference](https://platform.claude.com/docs/en/api/models/list)):

```python
m = client.models.retrieve("claude-opus-4-8")
m.max_input_tokens                                       # e.g. 1000000
m.max_tokens                                             # e.g. 128000
m.capabilities["thinking"]["supported"]                  # True
m.capabilities["thinking"]["types"]["adaptive"]["supported"]   # True
m.capabilities["thinking"]["types"]["enabled"]["supported"]    # extended thinking (False on Opus 4.7/4.8)
m.capabilities["effort"]["xhigh"]["supported"]           # True on Opus-tier
m.capabilities["effort"]["max"]["supported"]             # True
m.capabilities["image_input"]["supported"]               # True
m.capabilities["batch"]["supported"]                     # True
m.capabilities["context_management"]["compact_20260112"]["supported"]  # compaction
```

The `capabilities` object also exposes `citations`, `code_execution`, `pdf_input`, `structured_outputs`, and a `context_management` sub-tree (`clear_thinking_20251015`, `clear_tool_uses_20250919`, `compact_20260112`).

---

## Lineage and deprecations (high level)

Claude's family tree, oldest to newest. The headline trend across generations: adaptive thinking replaced fixed `budget_tokens`; sampling parameters (`temperature`/`top_p`/`top_k`) were removed (return 400 on **Opus 4.7+**); the tokenizer changed (4.7+, ~30–35% more tokens for the same text); and the 1M context window moved to standard pricing.

| Generation | Examples | Status (as of 2026-06-26) |
|---|---|---|
| Claude 1.x / 2.x / 3.x | Claude 1/Instant, 2.0/2.1, Opus 3, Sonnet 3/3.5/3.7, Haiku 3/3.5 | **Retired on first-party** (Sonnet 3.7 + Haiku 3.5 retired 2026-02-19; Haiku 3 retired 2026-04-20; Opus 3 retired 2026-01-05). Some still on Bedrock/Vertex |
| Claude 4.0 / 4.1 | Opus 4, Sonnet 4, Opus 4.1 | **Opus 4 & Sonnet 4 retired 2026-06-15.** **Opus 4.1 deprecated** (2026-06-05), retires **2026-08-05** → migrate to Opus 4.8 |
| Claude 4.5 | Opus 4.5, Sonnet 4.5, Haiku 4.5 | Opus 4.5 / Sonnet 4.5 **active (legacy)**; **Haiku 4.5 is the current Haiku** |
| Claude 4.6 | Opus 4.6, Sonnet 4.6 | **Active.** Sonnet 4.6 is the current Sonnet; Opus 4.6 is legacy Opus |
| Claude 4.7 / 4.8 | Opus 4.7, Opus 4.8 | **Active.** Opus 4.8 is the recommended flagship |
| Fable / Mythos 5 | Fable 5, Mythos 5, Mythos Preview | **Active.** Fable 5 GA; Mythos 5 = Project Glasswing only; **Mythos Preview retires 2026-06-30** |

**Retired models fail (their IDs no longer serve).** Common drop-in replacements (per the deprecation tables): Sonnet 3.x / Sonnet 4 → `claude-sonnet-4-6`; Opus 3 / Opus 4 / Opus 4.1 → `claude-opus-4-8`; Haiku 3 / 3.5 → `claude-haiku-4-5-20251001`. Anthropic gives **at least 60 days' notice** before retiring a publicly released model. For the full retirement schedule and per-model replacement table, see the [migration guide](https://platform.claude.com/docs/en/about-claude/models/migration-guide) and [model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations).

### Picking a model (quick heuristic)

- **Hardest reasoning / overnight autonomous agents, cost no object** → `claude-fable-5`
- **Default flagship — complex reasoning, agentic coding, high autonomy** → `claude-opus-4-8`
- **Most production workloads — speed + intelligence balance** → `claude-sonnet-4-6`
- **High-volume, latency-sensitive, simple tasks / sub-agents** → `claude-haiku-4-5`

### Out of scope here: adjacent (non-chat) models

This table covers callable **chat/completion** models only. Two adjacent capabilities live outside it but are worth knowing:

- **Embeddings** — Anthropic **does not offer its own embedding model**; the official docs recommend **[Voyage AI](https://platform.claude.com/docs/en/build-with-claude/embeddings)** as the embeddings provider (e.g. `voyage-4-large` / `voyage-4` / `voyage-4-lite`, plus domain-specific `voyage-code-3`, `voyage-finance-2`, `voyage-law-2`, and multimodal `voyage-multimodal-3.5`). Used for semantic search, RAG retrieval, recommendations, clustering.
- **Reranking** — Voyage AI also provides **[rerankers](https://docs.voyageai.com/docs/reranker)** (cross-encoders that re-score query↔document pairs to refine retrieval results); pair them with embeddings to cut retrieval-failure rates. Not a Claude Messages-API model — a separate Voyage endpoint.

---

## Related pages

- [./capabilities-and-modes.md](./capabilities-and-modes.md) — extended/adaptive thinking, effort, fast mode, 1M context, vision, tools (how the per-model capability rows behave)
- [../platform/claude-ai.md](../platform/claude-ai.md) — which models appear in Claude apps by plan, and the in-app model picker
- [../platform/claude-code.md](../platform/claude-code.md) — model selection and effort defaults in Claude Code
- [../capabilities/projects.md](../capabilities/projects.md) — knowledge base + caching interactions

## Open questions / to verify

- Exact knowledge/training **cutoff dates for Fable 5 and Mythos 5** — not published in the overview comparison table.
- Fable 5's exact **effort level set** and whether **prefill** / **sampling params** behave as on Opus (the Fable launch doc does not enumerate effort levels and does not document prefill/sampling-param 400s for Fable specifically).
- Whether any model's **context window narrows on another partner cloud** beyond the documented Opus 4.8 = 200K on Microsoft Foundry.
- All **prices** — re-verify against [platform.claude.com/docs](https://platform.claude.com/docs/en/about-claude/pricing) before building cost models.
- **Beta header lifecycles** — `output-300k-2026-03-24`, `fast-mode-2026-02-01`, `server-side-fallback-2026-06-01`, `task-budgets-2026-03-13`, `context-management-2025-06-27` may change names/availability; confirm current flags before shipping.
- **Priority Tier purchasability** — as of 2026-06-26 the [service tiers doc](https://platform.claude.com/docs/en/api/service-tiers) states new Priority Tier capacity commitments are **no longer available for purchase** (existing-commitment orgs only). Re-check whether this is permanent or whether a successor guaranteed-capacity offering (e.g. provisioned throughput) replaces it before quoting availability to a customer.

> **Mythos 5 verification (resolved 2026-06-26):** **CONFIRMED.** `claude-mythos-5` is an official model — it appears in the model table of the [Fable 5 / Mythos 5 launch doc](https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5) and on [anthropic.com/news/claude-fable-5-mythos-5](https://www.anthropic.com/news/claude-fable-5-mythos-5): same specs/pricing as Fable 5, no safety classifiers, Project Glasswing limited availability, successor to Mythos Preview. (It is newer than early-2026 session context, which predates the 2026-06-09 launch — not an error.)

## Sources

- Models overview — https://platform.claude.com/docs/en/about-claude/models/overview
- Pricing — https://platform.claude.com/docs/en/about-claude/pricing
- Introducing Claude Fable 5 and Claude Mythos 5 — https://platform.claude.com/docs/en/about-claude/models/introducing-claude-fable-5-and-claude-mythos-5
- Introducing Claude Opus 4.8 — https://www.anthropic.com/news/claude-opus-4-8
- Model IDs and versioning — https://platform.claude.com/docs/en/about-claude/models/model-ids-and-versions
- Model migration guide — https://platform.claude.com/docs/en/about-claude/models/migration-guide
- Model deprecations — https://platform.claude.com/docs/en/about-claude/model-deprecations
- Models API (list/retrieve) — https://platform.claude.com/docs/en/api/models/list
- Service tiers (`service_tier`, Priority Tier) — https://platform.claude.com/docs/en/api/service-tiers
- Embeddings (Voyage AI recommendation) — https://platform.claude.com/docs/en/build-with-claude/embeddings
- Reranking (Voyage AI) — https://docs.voyageai.com/docs/reranker
- Claude Fable 5 and Claude Mythos 5 (Anthropic news) — https://www.anthropic.com/news/claude-fable-5-mythos-5
