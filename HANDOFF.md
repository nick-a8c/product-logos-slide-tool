# Product Logos Slide Tool — Project Handoff

This document captures the state of the **Product Logos Slide Tool** (internal nickname: *Icon Stage*) as of **v2.3.0** so a fresh Claude session can pick up from here without re-deriving context. Read it top to bottom before making changes.

## Quick links

- **Live site:** https://nick-a8c.github.io/product-logos-slide-tool/
- **Source:** https://github.com/nick-a8c/product-logos-slide-tool
- **Owner:** Nebojsa Jurcic (`nick-a8c`)
- **Current version:** v2.3.0 (Panel release — Essentials/Advanced tiers, contextual Motion panel, Motion Feel, first-run intro backdrop)
- **Previous:** v2.2.0 (Onboarding — Setup Assistant, templates, tutorial, undo/redo)

## What this project is

A single-file HTML tool that composes brand animations from Automattic product icons and the AUTOMATTIC wordmark, then exports them. Two layout modes: a **four-segment Line timeline** (`Logos Intro → Logos Outro → A8C Intro → A8C Outro`) and a continuous 3D **Ring** orbit. Exports: **MP4, WebM, PNG, still SVG, and a self-contained interactive HTML page** (Ring). Built for a motion designer at Automattic (Nebojsa); originally scoped as a self-serve alternative to MOGRT for non-Adobe users (sales/business folks).

Everything lives in **one HTML file** with all dependencies inlined — drop it on any disk, double-click, works offline on `file://`.

The deliverable is `product-logos-slide-tool.html` (~444 KB). Plain HTML/CSS/JS, no build step, no framework. The repo also contains an identical `index.html` for GitHub Pages — **keep these two byte-identical** after every edit (`cp index.html product-logos-slide-tool.html`).

> **GIF export was removed** in v2.1.x. The `gifenc` library is still inlined but is now **dead code** — no `exportGIF` exists. Removing it is a safe future cleanup (~9 KB).

## Audience and tone

- User is **Nebojsa**, motion designer at Automattic, based in Subotica
- Communication style: informal, direct, action-oriented; prefers concise practical guidance
- He's the designer — **trust his visual judgement over your own assumptions**
- He prefers GitHub UI clicks over CLI for repo tasks; walk through screen-by-screen when needed

## Architecture

### Single-file, three script blocks

1. **Inlined `gifenc`** (~9 KB IIFE) — GIF encoder, **now unused**
2. **Inlined `mp4-muxer`** (~73 KB IIFE, exposes `window.Mp4Muxer`) — MP4 container muxer
3. **Main app script** — everything else

> A Lottie player was briefly inlined in v2.2 development for the layout thumbnails, then **removed** — both thumbnails are now pure CSS/SVG code. Don't re-add Lottie unless a feature genuinely needs it (it costs ~170 KB).

### State

`state` is a single global persisted to `localStorage` under key `icon-stage-v2` (don't rename without a migration).

**`serializeState()`** returns the persisted shape; **`persist()`** writes it. This is a **curated allow-list**, not a full state dump — *if you add a new state field you must add it to `serializeState()` or it will silently never save.* (This bit us once already.)

Key fields:
- **Composition:** `count`, `spacing`, `scale`, `stageZoom`; `a8cScale` (wordmark, separate)
- **Animation mirror:** top-level `duration`, `stagger`, `slideDistance`, `startScale`, `easing`, `auto` mirror the *active* segment. `syncTopFromActiveSegment()` / `syncActiveSegmentFromTop()` keep them in sync.
- **Per-segment params:** `state.segments = { logosIn, logosOut, a8cIn, a8cOut }`, each `{ duration, stagger, slideDistance, startScale, easing, auto:{…}, reverseOrder }`
- **Active segment:** `state.activeSegment`
- **Pauses:** `state.segmentPauses`
- **Layout:** `state.layout` (`'line'|'ring'`), `state.ring` (all ring params), `state.layoutLink`, `state.layoutShared`
- **Background:** `bgOn`, `bgMode` (`solid|linear|radial`), `bgColor`, `bgColorB`, `bgAngle`, `bgPos`, `bgSpread`
- **Export:** `aspectRatio`, `resolution`, `fps` (60), `videoQuality` (`web|high|max`), `exportRange`
- **Library:** `library` (SVGs), `assignments` (slot → icon id)
- **Chrome:** `theme`, `panelState`, `panelX/Y`
- **v2.2 onboarding:** `onboarded`, `tutorialSeen`
- **v2.3 panel:** `panelTier` (`'essentials'|'advanced'`), `motionFeel` (`'calm'|'balanced'|'snappy'|'custom'`)

### The four-segment model (Line)

| # | Segment | Content | Direction |
|---|---|---|---|
| 1 | `logosIn` | Logo icons | Intro |
| 2 | `logosOut` | Logo icons | Outro |
| 3 | `a8cIn` | AUTOMATTIC wordmark (10 letters) | Intro |
| 4 | `a8cOut` | AUTOMATTIC wordmark | Outro |

Helpers: `isA8cSeg()`, `isOutSeg()`, `getSegmentSlotCount()`, `getSegmentTotalDuration()`. Intros animate inner→outer (mirror stagger); outros reverse, with an **Order of movement** toggle (`reverseOrder`).

### Ring layout

`renderRingStage()` + `ringFrame()` (rAF) place icons on a tilted ellipse; depth drives scale, CSS blur, opacity, z-index. Live interactions (preview + Interactive HTML export only, never video): hover grow/shrink with gaussian spread, shift-click ripple, drag-to-spin. Optional staggered intro. Exports cover exactly one revolution for a seamless loop; video locks below speed 7 (`RING_MIN_VIDEO_SPEED`).

### AUTO systems (important)

Two parallel auto systems, both driven by **icon count**:

- **Line:** `AUTO_ANCHORS` (+ `AUTO_ANCHORS_RESTRICTED` for vertical aspects) → `getAutoValue(key, n)`. Recomputed by `recomputeAuto()` whenever count changes.
- **Ring (v2.2):** `RING_COUNT_ANCHORS_BY_ASPECT` → `ringAutoForCount(n, aspect)` → `applyRingCountAuto()`. **Radius + icon size are count-driven *per aspect*;** tilt + angle stay aspect-driven (`RING_AUTO_BY_ASPECT`). Also called from `recomputeAuto()` and `applyRingAutoFromAspect()`.

Ring anchor table (interpolates linearly between anchors; clamps at the ends, so counts below the lowest anchor **hold flat** rather than shrinking):

| Aspect | anchors (n → radius / size) |
|---|---|
| 16:9 & 21:9 (shared) | 3→20/18, 10→29/15, 16→40/14, 25→48/12 |
| 4:3 | 7→29/14, 25→57/9 |
| 9:16 | 7→60/11, 25→87/7 |
| 4:5 | 7→47/14, 25→81/10 |
| 1:1 | 7→61/17, 25→75/10 |

Note: 9:16 / 4:5 / 1:1 **cap icon count at 18** (`getMaxCountForAspect`), so the 25-anchor only defines the slope.

Ring AUTO is non-destructive: entering AUTO stashes the CUSTOM "playground" (`ringCustomStash` / `ringAutoPristine`); touching any ring control in AUTO burns the stash.

### Export pipeline

- `buildExportTimeline()` → `{ items:[{seg,startMs,durMs,endMs}], totalMs }`. Ring returns a single pseudo-segment covering one revolution.
- Every frame goes through `drawSequenceFrameSupersampled()` → `drawSegmentFrameSupersampled()` → **4× supersample + halving downsample** (Chrome only honors `imageSmoothingQuality` at 2:1 ratios). SVGs pre-rasterized at **6×** with a 6% transparent margin for AA fringe. Buffer capped at `MAX_SS_PIXELS` (64 MP).
- **Quality presets** (`QUALITY_PRESETS`, `state.videoQuality`): `web` (bpp 0.010, kf 2s) · `high` *(default)* (0.030, 2s) · `max` (1.000, every frame a keyframe). Bitrate = `max(2 Mbps, w·h·fps·videoBpp)`.
- **MP4** (`exportMP4`): WebCodecs + mp4-muxer, H.264, even dims, codec fallback chain. **WebM**: VP9 with alpha when bg off. **PNG / still SVG**: single frame (scrub-aware). Ring's still SVG bakes DoF as native `<feGaussianBlur>`. **Interactive HTML** (`exportRingHTML`): self-contained ring page.
- **Frame-scrub stills:** `scrubPaused` + wheel/drag over the stage (`RING_SCRUB_STEP` 2.5°, `LINE_SCRUB_STEP` 40 ms) → PNG/SVG capture that exact frame.

### Backgrounds

One source of truth: `getBgCSS()` (live stage + HTML embeds) and `fillBgCanvas()` (all canvas exporters), plus gradient defs in the SVG exporter.

---

## v2.3 — The panel (current release)

The panel had **39 controls** on screen in Line (35 in Ring), and **22 of those 39 were the same 5 knobs repeated** across four near-identical Motion sections. v2.3 gets the default view down to about **7 controls without removing anything**.

### Essentials / Advanced tiers

`state.panelTier` = `'essentials'` (default) | `'advanced'`. Toggle buttons at the top of the panel; `applyTierUI()` puts `.adv` on `.panel-content`. The whole mechanism is **one CSS rule**:

```css
.panel-content:not(.adv) [data-tier="advanced"] { display: none !important; }
```

29 sections/controls carry `data-tier="advanced"`. **Nothing is removed, unbound, or moved** — same DOM, same listeners, just filtered. An Advanced user sees exactly the v2.2 panel.

> When adding a control, decide its tier. Untagged = visible in Essentials. Tag it `data-tier="advanced"` on the `.ctrl` / `.ctrl-row` / `.row` / `.section` wrapper — **not** on an inner `.ctrl-label`, or you'll hide the label and orphan a visible slider.

### Contextual Motion panel

The four per-segment Motion sections collapsed into **one** `[data-section="animate-in"]` with a segment toggle (`#motionSeg`: In / Out / A8C in / A8C out). `applyMotionPanelUI()` writes `panel.dataset.segment = state.activeSegment`, which is what **retargets the per-control AUTO toggles**. `#motionOrderRow` ("Order of movement") shows only for `logosOut` / `a8cOut`, where it means something. `MOTION_SEG_LABEL` maps the keys to human names.

### Motion Feel (Essentials)

`FEEL_PRESETS` — Calm / Balanced / Snappy — sets all four segment durations at once. **Absolute values, not multipliers**, so re-picking a feel is idempotent (a multiplier would compound). Hand-tuning any `ANIM_FIELDS` slider calls `markMotionCustom()` → `state.motionFeel = 'custom'`, so the pill never lies about what you're on. **Line only** — Ring's motion is its Speed/Direction in the Ring section.

### AUTO note

While AUTO is on, Essentials shows `#autoNote` ("Spacing & scale sized automatically") + a **Customise** button instead of six greyed-out sliders. Customise flips to Advanced and turns AUTO off.

### First-run intro backdrop

`#introBackdrop` — a fixed `z-index: 99` layer (modals are 100) with `background: var(--app-bg)`, a drifting SVG `<pattern>` grid (56px tile, `rotate(45)`, ~6px/s via `patternTransform`) in `--accent`, and a `--welcome-scrim` wash. It genuinely **hides the UI** rather than blurring it.

- `introFlow` flag: raised at boot when `!state.onboarded`, and it **survives Welcome → Setup Assistant → tutorial**. `startIntro()` / `endIntro()`.
- While `body.intro-backdrop` is set, the three onboarding modals **drop their own overlay + blur** (else they'd frost the pattern).
- `endIntro()` is a **no-op outside the intro flow** — so a *returning* user reopening the assistant gets the normal translucent overlay over their real composition, not an opaque pattern hiding their work. Don't regress this.
- Drift is parked under `prefers-reduced-motion`.

> **Verification gotcha:** the headless preview **pauses rAF entirely unless the page paints** — measured 0 ticks in 600 ms. The drift and the thumbnails look dead if you read state via JS alone. Read state *between screenshots* to confirm they're actually running.

---

## v2.2 — Onboarding

### Welcome fork (first run)

New visitors get one **Welcome** modal (never two stacked) with three choices: **Setup Assistant** / **Show me how it works** (tutorial) / **Just start**. Gated by `state.onboarded`.

**Migration gotcha (already fixed, don't regress):** the "don't re-onboard existing users" check must test the **saved `data`**, not `state` — `state.onboarded` already defaults to `false` (a boolean), so `typeof state.onboarded !== 'boolean'` would never fire and v2.1 users upgrading would wrongly be re-onboarded. See `restore()`.

### Setup Assistant (the wizard)

Entry: the holographic **"Setup Assistant"** button at the top of the panel (`#btn-open-wizard`), or the Welcome fork. Dismissible modal over the tool. 4 steps + an optional step 0 (`WIZ_STEPS = 4`):

| Step | Content |
|---|---|
| 0 | *Start from a template* — only shown if templates exist |
| 1 | **Use-case** — drives the real settings (see mapping) |
| 2 | **Line or Ring** — two **animated thumbnails**; pre-selected from the use-case but the user's pick wins |
| 3 | **Brands** — 9×3 grid + a **live preview** of the chosen layout that updates as icons toggle |
| 4 | **Review** → "Open in tool" + optional "Save as template" |

> **v2.3.1 removed the old step-1 "Role"** (Marketing / Design / …). It only pre-sorted the use-case cards and set nothing, so it and `WIZ_ROLES` are gone; the branches were renumbered down by one. The layout-thumbnail step is now step **2** (the `if (wiz.step !== 2) destroyWizThumbs()` guard).

`applyWizard()` writes state and runs the normal apply pipeline (`setLayout` → `applyAspectRatio` → resolution → background → brands → `applyFeel` → `recomputeAuto` → `bindUIFromState` → `renderStage` → `persist`).

**Use-case mapping** (`WIZ_USECASES`): Social Reel (line/9:16/MP4/snappy) · Social square (line/1:1) · Slide (line/16:9/large) · Website hero (ring/21:9/Interactive HTML/calm) · Logo loop (ring/1:1/WebM) · Email banner (line/16:9/PNG) · Wallpaper (line/16:9/large).

**Background is always solid white** for every guided-setup result, regardless of layout/use-case. (The `uc.bg` field and `wiz.bgMode`/`wiz.bgColor` are now dead code — safe to prune.)

`setLayout()` was **hoisted from a `bindEvents` closure to a global function** so the wizard can switch layout programmatically.

### Animated layout thumbnails (step 3)

Both are **pure code, no Lottie**, sharing one blue (`#1869FD`) so they read as a set:
- **Line** — CSS `@keyframes wizLineDot`: 5 dots grow `0.44→1×` + fade, staggered centre-outward over a 3.767s loop (a faithful re-creation of the owner's original Lottie comp).
- **Ring** — `startWizRing()`: rAF orbit of 8 dots on a tilted ellipse; front dots larger/saturated, back dots smaller/lighter.

Lifecycle: `initWizThumbs()` on entering step 3, `destroyWizThumbs()` on leaving/closing (cancels the rAF). Card clicks only toggle the class — **no re-render** — so the animations don't restart.

### Live layout preview (step 4)

`renderWizPreview()` — Line = a centred row; Ring = a tilted ellipse with depth (front bigger, back smaller + slightly faded, **no blur**), real brand colours. Ring/icon size scale with count (fewer icons → smaller ring + bigger icons). Includes breathing space at the edges and a small info note ("You will be able to edit icons and position later").

**Gotcha:** it guards against a premature `getBoundingClientRect()` — if the box measures `< 300px` wide it retries on the next frame, otherwise the ring gets sized against a stale width and clusters to one side.

### Templates

In-app named templates in `localStorage` under **`icon-stage-templates-v1`** (separate from the main state key). `loadTemplates` / `addTemplate` / `deleteTemplate` / `applyTemplate`. A template snapshots `serializeState()`; loading copies design fields only — `TEMPLATE_SKIP_KEYS` excludes theme/panel/onboarding. Surfaces as wizard step 0 + "Save as template" on the review step. The old JSON preset export/import still exists separately.

### Tutorial

5-step walkthrough (`TUTORIAL_STEPS`), opened from the Welcome fork or the persistent bright **"How it works"** button. Progress dots, Back/Next (→ "Done"), Skip, ×, Esc, backdrop, and ←/→ arrows. `tutorialSeen` persists on close.

When opened from the Welcome fork, `openTutorial(true)` sets `tutFromWelcome`, so the **first step shows Back** and it returns to Welcome (keeping the intro backdrop up — it deliberately does *not* call `closeTutorial()`, which would `endIntro()`). Opened from the persistent launcher, the first step has no Back.

**The clips are still placeholders.** Each step has a `media` field, currently `null`, rendering a dashed 16:9 placeholder. To drop a real clip in, set:
```js
media: { type: 'video', src: '<data: URI>' }   // inlined <video>, autoplay/loop/muted
media: { type: 'svg',   markup: '<svg …>' }    // inline SVG/CSS animation
```
`renderTutMedia()` handles both; nothing else changes. **Offline rule:** video must be a **`data:` URI** (inlined), not an external path — keep clips small, or go the SVG/CSS route like the thumbnails.

### Undo / Redo

In-memory, **in-session** history (resets on reload — snapshots carry the icon library and would blow the localStorage quota).

- `HISTORY_MAX = 50` steps; `HISTORY_DEBOUNCE = 220 ms`.
- Hooked into **`persist()` → `recordHistory()`**, debounced so a slider drag collapses to **one** undo step.
- `undo()` / `redo()` restore a serialized snapshot via `restoreHistorySnapshot()`. `HISTORY_SKIP_KEYS` excludes UI chrome, so undo never flips your theme or moves the panel.
- `histSuspend` guards against recording during a restore; `historyReady` guards the boot-time persists.
- UI: two circular buttons in the bottom bar right of PLAY ALL (`#btn-undo` / `#btn-redo`); `updateHistoryUI()` enables/disables them from stack depth. Shortcuts: `Ctrl/Cmd+Z`, `Ctrl/Cmd+Shift+Z`, `Ctrl+Y` (ignored while typing in an input).

---

## Migrations in `restore()`

- Legacy panel-state values → `'docked'`
- `stagger > 100` → ÷4; `slideDistance > 50` → 0; `spacing > 40` → remapped
- Missing `aspectRatio` → inferred from resolution
- Missing `segments` → built from defaults; legacy top-level params → `segments.logosIn`
- Missing `segmentPauses`, `reverseOrder` → defaulted; `a8cScale > 80` → clamped
- Invalid `videoQuality` → `'high'`
- **v2.2:** missing `data.onboarded` / `data.tutorialSeen` → set **true** (existing users are not re-onboarded). Check `data`, not `state` — see the gotcha above.
- **v2.3:** no migration needed — a v2.2 save with no `panelTier` / `motionFeel` falls back to the defaults (`'essentials'`, `'balanced'`). Existing users land in Essentials, which is intended; every control is one click away under Advanced.

## What's done

Everything in the Line + Ring feature set (see above), all export formats offline, the full v2.2 onboarding surface (Welcome fork, Setup Assistant wizard — use-case → layout → brands → review, animated layout thumbnails, live layout preview, in-app templates, 5-step tutorial shell, undo/redo, per-aspect count-driven Ring AUTO, holographic Setup Assistant button, Overall Control refresh + tooltip), and the v2.3 panel restructure (Essentials/Advanced tiers, contextual Motion panel, Motion Feel, AUTO note, first-run intro backdrop). v2.3.1 trimmed the wizard's role step, moved Control into Essentials, labelled the undo/redo buttons, and gave the first-run tutorial a Back-to-Welcome path.

## What's pending or open

- **Tutorial clips** — the 5 media slots are placeholders. Owner to supply; see the Tutorial section for the drop-in format.
- **Animated (multi-frame) SVG export** — the SVG exporter emits a static still. A full-sequence animated SVG (SMIL keyframe concatenation) remains unimplemented.
- **Mobile layout is a prototype** — functional bottom-sheet behind `@media (max-width: 640px)`, unpolished, needs real-device testing.
- **Dead code to prune:** inlined `gifenc` (~9 KB, unused); `uc.bg` + `wiz.bgMode` / `wiz.bgColor` (background is now always white).
- **Node.js 20 deprecation warning** in the GitHub Actions workflow — update `actions/checkout` / `actions/configure-pages` before June 2, 2026.
- **Known limits:** MP4 won't preserve transparency (H.264); WebCodecs MP4 is Chromium + Safari 16.4+ only (Firefox lacks it — use WebM).

## Conventions to follow when iterating

- **One change at a time, small diffs.** Use `Edit`, not full-file rewrites.
- **Always parse-check** after risky edits (one-liner below).
- **Always verify in a browser.** Don't claim "works" without inspecting state/DOM. Several real bugs this project hit (the `persist()` allow-list, the onboarding migration, the stale preview width) were **invisible to a code read** and only surfaced by driving the UI.
- **Keep the two HTML files byte-identical**: `cp index.html product-logos-slide-tool.html`.
- **Use the CSS variables** (`--accent`, `--text`, `--panel-bg`, …) so light/dark both work. Don't invent fallbacks like `var(--input-bg, #f6f6f6)` — that fallback applies in *both* themes.
- **Ask** when the spec is ambiguous or a change is cross-cutting. **Trust the owner's visual judgement.**
- **Save a fallback** before destructive changes: `cp index.html product-logos-slide-tool-vX.Y.Z-fallback.html`.

## Useful one-liners

Parse-check all script blocks:
```bash
node -e "
const fs=require('fs');const html=fs.readFileSync('index.html','utf8');
const re=/<script>([\s\S]*?)<\/script>/g;let m,i=0;
while((m=re.exec(html))){i++;try{new Function(m[1])}catch(e){console.error('Block',i,'ERR',e.message)}}
console.log('Parsed',i,'blocks OK');"
```

Keep deliverables synced + deploy:
```bash
cp index.html product-logos-slide-tool.html
git add -A && git commit -m "your message" && git push origin main
```

Serve locally (localStorage behaves better than `file://`):
```bash
python3 -m http.server 8931
```
