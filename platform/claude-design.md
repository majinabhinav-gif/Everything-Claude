---
type: Surface
title: Claude Design
description: Anthropic Labs' prompt-to-prototype visual workspace where you talk to Claude on a live canvas to build designs, prototypes, slides, and one-pagers, then hand them to Claude Code.
domain: platform
tags: [claude-design, anthropic-labs, prompt-to-prototype, design, prototyping, slides, canvas, design-system, design-to-code, vision, opus-4.7]
related: [claude-code-overview, artifacts, claude-ai, cowork, capabilities-and-modes, connectors]
resource: https://www.anthropic.com/news/claude-design-anthropic-labs
timestamp: 2026-06-26T00:00:00Z
confidence: high
okf_version: "0.1"
sources:
  - https://www.anthropic.com/news/claude-design-anthropic-labs
  - https://support.claude.com/en/articles/14604416-get-started-with-claude-design
  - https://techcrunch.com/2026/04/17/anthropic-launches-claude-design-a-new-product-for-creating-quick-visuals/
  - https://venturebeat.com/technology/anthropic-just-launched-claude-design-an-ai-tool-that-turns-prompts-into-prototypes-and-challenges-figma
  - https://venturebeat.com/technology/anthropic-ships-major-claude-design-overhaul-with-design-system-imports-code-round-trips-and-a-fix-for-its-token-burning-problem
  - https://www.technobezz.com/news/anthropic-launches-claude-design-update-with-direct-pipeline-to-claude-code
  - https://www.datacamp.com/blog/claude-design
  - https://getpushtoprod.substack.com/p/everything-you-need-to-know-about
  - https://sagnikbhattacharya.com/blog/claude-design
---

# Claude Design

Claude Design is the **prompt-to-prototype visual workspace** from **Anthropic Labs**: instead of talking to Claude in a chat transcript, you talk to it on a **live canvas** and watch polished visual work — designs, interactive prototypes, slide decks, one-pagers, wireframes, landing pages — appear and update in real time. You describe what you want in one sentence, review a first version, then refine through chat, inline comments, direct drag-and-drop edits, and adjustment sliders that Claude generates for your layout. When a design is ready to ship, Claude packages it into a **handoff bundle** and passes it to **Claude Code** so the coding agent programs the real interface "exactly where the designer left off" — no screenshot, no rebuild from scratch.

> At a glance
> - **What it is:** An Anthropic Labs visual-work surface (in **beta**) that gives Claude a canvas to produce and iterate on designs, prototypes, slides, and one-pagers by conversation, with a direct pipeline to Claude Code for implementation.
> - **Where you find it:** `claude.ai/design` on the web, or from the **Claude Desktop** app sidebar. **Not available on mobile** (per the Claude Help Center). Companion entry points in Claude Code via the `/design-sync` and `/design` commands (surfaced by the Claude Design integration, not in the core commands reference; reported to require **Claude Code v2.1.181+**).
> - **Who can use it by plan:** Claude **Pro, Max, Team, and Enterprise**. **No Free tier.** On **Enterprise** it is **off by default** — an admin must enable it in **Organization settings**.
> - **Status:** Launched **2026-04-17** in beta (gradual rollout that day, alongside the Opus 4.7 model upgrade). A **major overhaul on 2026-06-17** added design-system imports, two-way code round-trips (`/design-sync`), enterprise brand governance, a local canvas editor, and shared (pooled) usage limits. Still labeled **beta** as of 2026-06-26. Powered by **Claude Opus 4.7** (Anthropic's most capable vision model at the time of launch). *Note: Anthropic later shipped **Opus 4.8** (May 2026) as its newest Opus-tier model; the Design announcement still names Opus 4.7 and Anthropic has not published whether the surface auto-upgrades — treat the exact serving model as unconfirmed.*

---

## What Claude Design is (and is not)

Claude Design is positioned for **rapid ideation-to-visual workflows** — Anthropic explicitly aimed it at "founders and product managers without a design background" who want to get from an idea to something clickable fast, as well as designers who want to explore variations quickly. Anthropic has called it **complementary** to tools like Canva rather than a replacement: ideate and prototype in Claude Design, then export to a dedicated design tool (or to code) for professional refinement and collaboration.

It is **not** a full production design suite, and it does **not** reliably simulate real backend behavior. Interactive prototypes can wire up navigation, toggles, modals, form validation, and tab bars, but real authentication, live API calls, and persisted server data are out of scope for the prototype itself (see [Live data and interactivity](#live-data-and-interactivity)).

| Claude Design is good for | Reach for something else when |
|---|---|
| One-prompt clickable prototypes for user testing | You need pixel-locked, hand-tuned production design files (Figma) |
| Pitch decks, one-pagers, marketing landing pages | You need real backend logic, auth, and persisted data (use Claude Code) |
| Exploring 2–3 layout variations fast | Long-form collaborative editing with many stakeholders (export to Canva) |
| Handing a design to engineering to build | Print/brand production at agency fidelity |

---

## The model and capabilities behind it

Claude Design is **powered by Claude Opus 4.7**, described at launch as Anthropic's most capable **vision** model — the vision strength is what lets it read screenshots, design files, and web captures and reason about layout, spacing, and visual hierarchy. See [../models/model-families.md](../models/model-families.md) for model IDs and pricing, and [../models/capabilities-and-modes.md](../models/capabilities-and-modes.md) for vision and extended-thinking details.

> **WARN: verify (model version).** The official announcement and 9to5Mac both tie the launch to the **Opus 4.7** upgrade. Anthropic subsequently released **Opus 4.8** (May 2026) as its newest Opus-tier model. Anthropic has **not** published whether Claude Design has been re-pointed to a newer Opus release, so the currently-serving model on the surface is unconfirmed — the page intentionally keeps the sourced "Opus 4.7" claim while flagging that it may have advanced.

---

## Prompt-to-prototype: the core loop

The defining behavior is **one sentence in, clickable prototype out**. You write a plain-language description and Claude builds a first version on the canvas; everything after that is refinement.

```text
# Example seed prompts (typed into the chat pane)

"Build a clickable prototype for a habit-tracking mobile app: onboarding,
 a home dashboard with streaks, and an 'add habit' flow. Friendly, rounded,
 mint-green accent."

"Design a SaaS pricing landing page with a hero, three pricing tiers
 (monthly/annual toggle), a feature comparison table, and an FAQ accordion."

"Make a 10-slide pitch deck for a B2B logistics startup: problem, solution,
 market, product demo, traction, team, ask. Use our brand design system."

"Turn this PRD (attached DOCX) into a one-pager I can send to leadership."
```

After the first render you iterate conversationally:

```text
"Make the color scheme darker and more minimal."
"Show me 2–3 alternative layouts for this page."
"Add a dark mode toggle to the header."
"Tighten the vertical spacing in the pricing cards."
```

---

## Starting a project: the four input paths

You can seed a Claude Design project from any of four sources — and combine them:

| Input path | What it does | Example |
|---|---|---|
| **Text prompt** | Describe the visual in natural language | "A clickable dashboard for a fitness app." |
| **Upload images / docs / assets** | Drop in screenshots, images, existing assets, slide decks, and documents as references | Upload an existing deck to restyle, or a doc to turn into a one-pager |
| **Point at your codebase / design files** | Link or upload a GitHub repo, local folder, or design files so Claude reads real components/tokens | "Use the components in `apps/web/src/ui`." |
| **Web capture tool** | Grab elements directly from a live website so prototypes mirror the real product | Capture your marketing site's nav and reuse it 1:1 |

The Help Center groups inputs as: **text prompts**; **screenshots, images, and existing assets**; **codebases and design files** (link or upload); and **existing slide decks / documents**. The **web capture** tool is what makes prototypes "look like the real product" — you pull existing interface patterns from your own site rather than re-describing them. (Note: a specific list of accepted upload file extensions — e.g. DOCX/PPTX/XLSX — is **not** enumerated in official docs; the surface does ingest decks and documents, but treat exact extension support as **WARN: verify**.)

---

## The canvas and editing experience

Claude Design uses a **two-pane interface**: **chat on the left, a live canvas on the right.** WARN: verify exact pane layout in the current build. Beyond chatting, you have four hands-on ways to change a design without re-explaining the whole thing:

### 1. Inline comments
Click a specific element on the canvas and leave a targeted comment ("this CTA should be bigger"). Claude reads the comment and applies just that change.
- **Known issue / workaround:** comments occasionally disappear before Claude reads them; the documented fallback is to **paste the feedback directly into the chat** instead.

### 2. Direct text edits
Edit copy in place — click the text and type — without sending a prompt.

### 3. Direct layout manipulation (rich layout controls)
The rebuilt canvas editor lets you **drag, resize, and align** elements directly. This is also the **token-saving** path: small visual tweaks happen locally in the editor "without burning a model turn for every tweak."

### 4. Adjustment sliders / "knobs"
For a given design, Claude **generates custom sliders** ("adjustment knobs") that let you tune **spacing, color, and layout live**, with changes applied across the full design.

```text
# Refinement toolkit, by intent
Big conceptual change ........ chat ("redesign the hero, more editorial")
Target one element ........... inline comment on that element
Fix copy ..................... direct text edit
Reposition / resize .......... drag/resize/align on canvas (no model turn)
Dial in spacing or color ..... adjustment slider Claude created
```

---

## Canvas toolbar, navigation, and viewport controls

Beyond the four editing modes above, the canvas has its own chrome for moving around a design and switching how you view it. **WARN: verify** — Anthropic does not publish a dedicated "canvas controls" or keyboard-shortcut reference, so the labels below come from beta walkthroughs (getpushtoprod, sagnikbhattacharya) and should be treated as reported-not-official.

### Zoom and pan
- **Zoom** runs from **50% to 200%** (reported range). You **zoom in/out** to inspect detail or see a whole flow at once.
- **Pan / scroll between screens:** you **scroll between slides or screens** on the canvas — a multi-screen prototype or multi-slide deck lays its frames out so you can move from one to the next without leaving the project.
- **Selection:** **click any element** to select it; **click-and-drag** to marquee-select multiple elements at once.

### Canvas modes (top-bar toolbar)
Reported beta builds expose a set of named **modes** in the canvas toolbar, layered on top of the four editing affordances in [The canvas and editing experience](#the-canvas-and-editing-experience):

| Mode | What it does (reported) |
|---|---|
| **Tweaks** | Toggles Claude's generated **sliders and color pickers** on, surfacing live controls (theme color, accent color, timing values) for whatever Claude flagged as adjustable |
| **Edit** | Opens a **property inspector on the right side** for background color, font family, and per-element properties |
| **Comments** | Inline-comment mode, with checkboxes to **"Select for Send to Claude"** so you can batch which notes Claude acts on |
| **Draw** | Sketch/annotate directly on the canvas; supports **mic input** for spoken instructions alongside the sketch |
| **Present** | View the prototype **in the current tab, fullscreen, or a new tab** |

The **floating element toolbar** that appears when you click an element offers the per-element trio **Comment / Edit text directly / Adjust** (the same Comment, direct-text-edit, and slider affordances documented above, scoped to the selection).

### Viewport / breakpoint presets
The live **preview** historically offered only a basic **mobile device-simulation toggle**; full **tablet / desktop / wide** presets in the canvas are a known gap, even though the generated CSS can be fully responsive (see [Responsive breakpoints](#responsive-breakpoints-mobile--tablet--desktop) below). **WARN: verify** the current set of in-canvas viewport presets.

### Layers, frames, and artboards
A right-hand panel surfaces **layer-style controls** and an **"Examples" tab** in reported builds, but Anthropic publishes **no** documented **layers panel, frame/artboard manager, or named-screen organizer** the way a dedicated design tool (Figma) would. Multi-screen structure is expressed as the **scrollable sequence of screens/slides** on the canvas rather than an explicit artboard hierarchy. **WARN: verify** whether a formal layers/frames panel exists in the current build.

### Undo / redo and keyboard shortcuts
Reported top-bar controls include **Undo** and **Redo** alongside **Share** and **Export**, so direct-manipulation edits on the canvas are reversible without a chat turn. Anthropic has **not** published a Claude Design canvas keyboard-shortcut reference — standard browser/OS combos (e.g. `Ctrl/Cmd+Z` undo, `Ctrl/Cmd+Shift+Z` redo) are **plausible but unconfirmed**, and there is **no** documented spacebar-pan or zoom-to-fit shortcut. **WARN: verify** all canvas keyboard shortcuts against the live app.

```text
# Canvas controls cheat-sheet (reported, not officially documented)
Zoom ......................... 50%–200%, zoom in/out
Move between screens ......... scroll between slides/screens
Select ....................... click element; click-drag = marquee multi-select
Reversible edits ............. Undo / Redo in the top bar
Modes ........................ Tweaks · Edit · Comments · Draw · Present
Per-element toolbar .......... Comment · Edit text directly · Adjust
Right panel .................. property inspector (Edit) · "Examples" tab
```

---

## Project & version management

Claude Design organizes work into **design projects** (the same "project" concept used elsewhere in Claude — a project carries its own conversation, canvas, attached design system, and context). The reported workspace surfaces:

- A **"New project"** button on the home screen to start a fresh design.
- A **list of recent projects in the left sidebar** so you can jump back into earlier work. (This is the reported project list; a richer gallery/grid view is not separately documented — **WARN: verify**.)
- **Share ▸ "Duplicate as Template"** to clone a design as a reusable starting point (reported). A plain **Duplicate**, **Rename**, and **Delete** per project are expected workspace operations but are **not** enumerated in official docs — **WARN: verify** the exact menu labels and locations.

### Saving a direction and returning to past work
The **officially documented** way to preserve an iteration before exploring a new direction is **conversational**, not a version panel. The Help Center's pattern:

```text
# In the chat pane, before trying something risky:
"Save what we have and try a completely different approach."
# → Claude saves your current project and confirms where it's saved,
#   so you can reference earlier iterations later in the conversation.
```

This means a project's history is partly carried **in the conversation itself** — earlier states are recoverable by referring back to a saved point ("go back to the version before the dark redesign") rather than by scrubbing a timeline. **WARN: verify** whether the canvas exposes a discrete **version-history timeline / restore-to-version** control (as Claude **artifacts** do — see [../capabilities/artifacts.md](../capabilities/artifacts.md)); reporting on the Design canvas does not confirm a dedicated restore-a-prior-version UI, and the documented mechanism is the save-and-reference conversational flow plus canvas **Undo/Redo** for fine-grained reversal.

| Need | Reported / documented path |
|---|---|
| Start fresh | **"New project"** (home screen) |
| Reopen earlier work | **Recent projects** in the left sidebar |
| Reuse a design as a base | **Share ▸ "Duplicate as Template"** |
| Preserve before pivoting | Chat: *"Save what we have and try a different approach"* (Claude confirms where it's saved) |
| Undo a manipulation | **Undo / Redo** in the canvas top bar |
| Restore a much-earlier state | Refer back to a **saved point in the conversation** (no confirmed timeline-scrub UI — **WARN: verify**) |
| Rename / Delete a project | Expected workspace actions — **WARN: verify** labels |

---

## Toggles and outputs Claude Design can produce or control

A power-user's view of the switches Claude Design generates **inside** a prototype, plus the viewport controls in the **workspace**:

### Dark-mode toggle
Claude can build a **dark-mode toggle** into a prototype's UI; dark-mode variants can switch dynamically based on **system preference** or a **user toggle**, and a full design system can ship light + dark variants. You add one by prompting, e.g. `"Add a dark mode toggle to the header."`

### Responsive breakpoints (mobile / tablet / desktop)
Claude designs responsively across the standard breakpoints. State which targets you care about in the prompt.

| Tier | Common widths cited | Notes |
|---|---|---|
| Mobile | ~320–375px | Mobile-first media queries |
| Tablet | ~768px | |
| Desktop | ~1200–1440px | |

> **WARN: verify** — reporting indicates the in-workspace **preview** historically offered only a basic **mobile device simulation toggle**, with tablet/wide-screen presets a known gap. The generated CSS can still be fully responsive even if the live preview lacks every viewport preset. Confirm the current set of viewport presets in the canvas.

### Interactivity / clickable flows
Claude produces **multi-screen flows** with navigation logic between views, plus interactive components: **toggles, modals, form validation, tab bars**, accordions, monthly/annual price switches, etc. These are meant to be clickable enough for **user testing without a code review**.

### Live data and interactivity
Prototypes can *look* data-driven and respond to interaction, but Claude **can't reliably simulate backend behavior** (real authentication, real API calls, persisted live data) in the prototype itself. For genuinely live data you take the **handoff to Claude Code** and wire it to a real backend (e.g., a database via [../capabilities/connectors.md](../capabilities/connectors.md) or MCP — see [../capabilities/mcp.md](../capabilities/mcp.md)). "Code-powered prototypes" can additionally include **voice, video, shaders, 3D, and built-in AI**.

---

## Design systems: onboarding and governance

During **onboarding**, Claude builds a **design system for your team** by reading your codebase and design files; every project afterward automatically uses your **colors, typography, and components**. Teams can maintain **multiple** design systems.

The **2026-06-17 overhaul** made systems first-class and governable:

- **Import sources:** bring one or multiple design systems in from a **GitHub repository, a design file, or a raw upload** rather than letting Claude invent its own buttons/spacing.
- **Validation:** Claude **checks its own output against your active design system** before delivering results, so generated screens use your real components, colors, and typography.
- **Inheritance:** new projects automatically **inherit the organization's design system**, so brand assets don't need re-uploading.
- **Enterprise / brand governance:** the update added controls so an organization can **enforce brand rules** and standardize on an approved system. (The general thrust — admin/brand control over what the surface produces — is confirmed by the overhaul reporting; treat any specific "lock-down / single-standard-system" admin toggle wording as **WARN: verify** against official docs.)

### `/design-sync` and `/design` (Claude Code integration)

These commands are surfaced by the **Claude Design integration inside Claude Code** — they are **not** listed in the core Claude Code [commands reference](../claude-code/slash-commands.md). Reporting indicates they require **Claude Code v2.1.181 or later**, and they lean on your **claude.ai login** to find design-system projects you have write permission on (no extra setup).

```text
# In Claude Code, inside your project's terminal:

/design-sync          # Two-way sync between your repo and Claude Design:
                      #   pull  → import the repo's design system/tokens/components
                      #           into Claude Design, so generated screens use them
                      #   push  → after implementing a design in code, push the
                      #           current state back so the canvas matches reality
                      # Builds a PLAN (files to write/delete, starting folder) and
                      # returns a planId you must approve before anything changes.
                      # Does NOT watch the repo — re-run it after token/component edits.

/design               # Create, edit, and sync design projects without leaving
                      # the terminal (companion to /design-sync).
```

The Claude Help Center confirms `/design-sync` is the path to **import your design system into Claude Design**. **WARN: verify** the exact arguments, the precise version floor, and whether these are formally documented commands versus integration-provided ones, against official docs.

---

## Design-to-code handoff to Claude Code

This is the headline workflow and the reason Claude Design sits next to a coding agent. When a design is ready to build, **Claude packages everything into a handoff bundle** that you pass to **Claude Code with a single instruction**. Claude Code then "picks up **exactly where the designer left off** — no screenshot, no rebuild."

Crucially, the handoff is **codebase-aware**: because Claude Design can read your **local codebase** during the project, it hands the design to the coding agent so it **programs the interface without starting from scratch**, reusing your existing components instead of regenerating them.

```text
# Conceptual round-trip
1. Designer prototypes in Claude Design (reads repo's design system).
2. Export ▸ "Handoff to Claude Code" → produces a handoff bundle.
3. In Claude Code (local agent or web), one instruction:
   "Implement this handoff in apps/web, using our existing components."
4. Claude Code builds the real, wired-up interface in the actual codebase.
```

The handoff targets either the **local Claude Code agent** or **Claude Code on the web**. See [./claude-code.md](./claude-code.md) and [../claude-code/dispatch-remote-routines.md](../claude-code/dispatch-remote-routines.md). A common end-to-end pattern Anthropic describes: explore in Claude Design → hand to Claude Code to implement → let [./cowork.md](./cowork.md) manage the review cycle.

---

## Export, share, and collaboration

### Export destinations
The Export menu covers downloadable files, decks, and a growing set of partner integrations.

| Category | Targets |
|---|---|
| **Files / URL** | Internal share URL, download to a **folder**, **.zip**, **PDF**, **PPTX**, standalone **HTML** |
| **Design tools / decks** | **Canva** (editable + collaborative), **Gamma**, **Miro** |
| **Build / deploy partners** | **Vercel**, **Replit**, **Lovable**, **Base44**, **Wix**, **Adobe** |
| **To code** | **Handoff to Claude Code** (local agent or web) |

The confirmed partner roster (announcement + Help Center + overhaul reporting) is: **Adobe, Base44, Canva, Gamma, Lovable, Miro, Replit, Vercel, Wix**, plus **PDF/PPTX/HTML** file exports and the **Claude Code** handoff. **WARN: verify** the exact export-menu button labels; the integration list has grown over time and may keep changing.

### Sharing and collaboration
Sharing is **organization-scoped**. By default a project is **private to you**; you share it via a **shareable link** within your org. The Help Center lists three share access levels:

| Access level | Capability |
|---|---|
| **View-only** | See the design |
| **Comment** | See the design and leave inline comments |
| **Edit** | Modify the design and chat with Claude in the same conversation |

> **Known limitation (from the Help Center):** **multi-person editing** — two or more people editing the same design project at the same time — is **"still basic and may not work reliably."** So while collaborators can be brought into the same project to steer Claude together, simultaneous live co-editing is not yet robust. **WARN: verify** whether external (non-org) sharing is supported — official docs describe sharing within your organization only.

---

## Availability, plans, and usage limits

- **Plans:** Claude **Pro, Max, Team, Enterprise**. **No Free tier** (confirmed: announcement + Help Center).
- **Enterprise default:** **off by default**; an org admin enables it in **Organization settings** (confirmed wording from both the announcement and Help Center). The exact in-console toggle label/path is not separately documented.
- **Surfaces:** **web** at `claude.ai/design` and the **Claude Desktop** app sidebar. **Not available on mobile** (confirmed by the Help Center) — the canvas is web/desktop only.
- **Usage / metering:** **Claude Design counts toward the same usage limits as the rest of Claude** (confirmed Help Center wording) — pooled with chat, [./cowork.md](./cowork.md) (Cowork), and [./claude-code.md](./claude-code.md), rather than drawing from a separate, smaller pool. Complex projects with large codebases consume more usage. When limits are hit, Claude Design becomes unavailable until reset; you can **enable "extra usage"** to keep working beyond plan limits. Specific per-plan numeric allowances are **not** published in official docs (any number you see elsewhere is **WARN: verify**).

> Note on the "token-burning" history: the first research preview was criticized for consuming usage quickly because every tweak cost a model turn. The overhaul fixed this two ways: (1) the **canvas editor** handles small drag/resize/color/spacing edits **locally** without a model call, and (2) usage was **pooled** with the rest of Claude rather than a small dedicated bucket.

---

## Worked examples

### Example A — Landing page (prompt → export to Vercel)
```text
Prompt: "SaaS landing page for an AI meeting-notes app. Hero with product
 screenshot, logo cloud, 3 feature cards, a monthly/annual pricing toggle,
 testimonial carousel, footer. Use our brand design system. Responsive,
 mobile + desktop, with a dark mode toggle."
Iterate:  inline-comment the hero → "make the headline 2 lines, larger"
          slider → reduce section spacing 20%
Export:   ▸ Vercel (deploy preview)  OR  ▸ Handoff to Claude Code
```

### Example B — Product dashboard (codebase-aware → handoff)
```text
Seed:     point Claude at the GitHub repo so it reads existing UI components
Prompt:   "Design an analytics dashboard: KPI row, a line chart, a sortable
           table, and a filter sidebar — using our component library."
Live data: prototype shows sample data; real data comes after handoff
Export:   ▸ Handoff to Claude Code → "Implement in apps/web with our components"
```

### Example C — Slide deck / one-pager
```text
Prompt:   "10-slide investor deck from this PRD (attached DOCX). Brand system.
           Then also produce a single one-pager summary."
Refine:   "Show 2–3 layout alternatives for the traction slide."
Export:   ▸ PPTX (editable deck)  ▸ PDF (one-pager)  ▸ Send to Canva
```

### Example D — Mobile app prototype (clickable flow)
```text
Prompt:   "Clickable iOS prototype: onboarding (3 screens) → home with streaks
           → add-habit modal → settings with a dark mode toggle. Tab bar nav."
Test:     share view/comment link with PM; collect inline comments
Note:     navigation + modals are clickable; no real auth/backend in prototype
```

---

## Tips for power users

- **Be explicit about viewport targets** ("must work on mobile, tablet, and desktop") — Claude tailors layout and breakpoints to what you name.
- **Use the canvas editor for nudges, chat for concepts.** Drag/resize/align and sliders don't spend a model turn; save chat for substantive changes.
- **Seed from your codebase early** so the eventual Claude Code handoff reuses real components instead of regenerating UI.
- **If inline comments vanish, paste them into chat** — documented workaround for a known bug.
- **For real data, plan the handoff.** Prototype the look in Claude Design; wire live data in Claude Code via [../capabilities/connectors.md](../capabilities/connectors.md) / [../capabilities/mcp.md](../capabilities/mcp.md).

---

## Related pages
- [./claude-code.md](./claude-code.md) — the coding agent that receives the design handoff bundle and implements it
- [../capabilities/artifacts.md](../capabilities/artifacts.md) — the in-chat sibling for self-contained interactive outputs; Design is the dedicated canvas surface
- [./claude-ai.md](./claude-ai.md) — the Claude.ai apps, plans, and settings where Design lives
- [./cowork.md](./cowork.md) — desktop agent that can manage the review cycle around a design build
- [../models/capabilities-and-modes.md](../models/capabilities-and-modes.md) — vision and modes powering the canvas
- [../claude-code/slash-commands.md](../claude-code/slash-commands.md) — `/design` and `/design-sync` commands
- [../capabilities/connectors.md](../capabilities/connectors.md) — wiring prototypes to real data after handoff
- [../capabilities/mcp.md](../capabilities/mcp.md) — Model Context Protocol for connecting handed-off builds to live systems

## Open questions / to verify
- Whether the surface still serves **Opus 4.7** or has been re-pointed to **Opus 4.8** (the newer Opus-tier model shipped May 2026); the announcement still names Opus 4.7 and Anthropic hasn't published an upgrade.
- Exact in-canvas **viewport presets** (does tablet/wide-screen simulation exist beyond a mobile toggle?).
- Precise **export-menu button labels** (the destination list itself is now confirmed).
- Exact **admin enablement toggle label/path** on Enterprise, and the precise **brand "lock-down / single standard system"** governance controls.
- Exact arguments and the precise **Claude Code version floor** for **`/design-sync`** / **`/design`**, and whether they are formally documented vs. integration-provided (the official commands reference does not list them).
- The **web capture** tool's exact UI label and invocation.
- Whether **external (non-org) sharing** is supported (docs describe org-scoped sharing only).
- Accepted **upload file extensions** (decks/documents are accepted; a specific extension list isn't published).
- Whether the canvas has a discrete **version-history timeline / restore-to-version** control (like Claude artifacts) versus only the documented "save and reference in conversation" flow + Undo/Redo.
- Exact per-project **Rename / Duplicate / Delete** menu labels and whether a richer **project gallery/grid** exists beyond the recent-projects sidebar (only **Share ▸ "Duplicate as Template"** is reported).
- The full set of **canvas modes** (Tweaks / Edit / Comments / Draw / Present) and the **"Examples" tab** — confirm names/behaviors against the live app.
- Whether a formal **layers panel / frame–artboard manager** exists, or whether multi-screen structure is only the scrollable screen/slide sequence.
- Any **Claude Design canvas keyboard shortcuts** (undo/redo, pan, zoom-to-fit) — none are officially published; the reported **zoom range is 50%–200%**.

### Resolved during verification (2026-06-26)
- **Plans:** Pro/Max/Team/Enterprise, no Free; Enterprise off by default, admin-enabled in Organization settings — confirmed.
- **Mobile:** **not available on mobile** — confirmed by the Help Center.
- **Usage:** pooled with the rest of Claude (chat, Claude Code, Cowork); "extra usage" available — confirmed.
- **Overhaul date:** **2026-06-17** — confirmed by overhaul reporting.
- **Multi-person editing:** "basic and may not work reliably" — confirmed by the Help Center.
- **Sharing levels:** view-only / comment / edit (private by default) — confirmed.

## Sources
- Anthropic — Introducing Claude Design by Anthropic Labs: https://www.anthropic.com/news/claude-design-anthropic-labs
- Claude Help Center — Get started with Claude Design: https://support.claude.com/en/articles/14604416-get-started-with-claude-design
- TechCrunch — Anthropic launches Claude Design (2026-04-17): https://techcrunch.com/2026/04/17/anthropic-launches-claude-design-a-new-product-for-creating-quick-visuals/
- VentureBeat — Anthropic just launched Claude Design (prompts → prototypes): https://venturebeat.com/technology/anthropic-just-launched-claude-design-an-ai-tool-that-turns-prompts-into-prototypes-and-challenges-figma
- VentureBeat — Major Claude Design overhaul (design-system imports, code round-trips, token fix): https://venturebeat.com/technology/anthropic-ships-major-claude-design-overhaul-with-design-system-imports-code-round-trips-and-a-fix-for-its-token-burning-problem
- Technobezz — Claude Design update with direct pipeline to Claude Code: https://www.technobezz.com/news/anthropic-launches-claude-design-update-with-direct-pipeline-to-claude-code
- DataCamp — What Is Claude Design? Anthropic's AI Design Tool Explained: https://www.datacamp.com/blog/claude-design
- Push to Prod — Everything You Need to Know About Claude Design (beta canvas walkthrough: zoom 50%–200%, Tweaks/Edit/Comments/Draw/Present modes, "Duplicate as Template"): https://getpushtoprod.substack.com/p/everything-you-need-to-know-about
- Sagnik Bhattacharya — How to Use Claude Design (canvas two-pane, scroll between screens, "New project", recent-projects sidebar, floating Comment/Edit text/Adjust toolbar): https://sagnikbhattacharya.com/blog/claude-design
- 9to5Mac — Anthropic launches Claude Design following Opus 4.7 model upgrade (2026-04-17): https://9to5mac.com/2026/04/17/anthropic-launches-claude-design-for-mac-following-opus-4-7-model-upgrade/
- Claude Platform Docs — Models overview / What's new in Opus 4.8 (newer Opus-tier model, May 2026): https://platform.claude.com/docs/en/about-claude/models/overview
- Claude Code Docs — Commands reference (confirms `/design`,`/design-sync` are not core built-ins): https://code.claude.com/docs/en/commands
