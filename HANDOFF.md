# Product Logos Slide Tool — Project Handoff

This document captures the state of the **Product Logos Slide Tool** (internal nickname: *Icon Stage*) so a fresh Claude session can pick up from here without re-deriving context. Read it top to bottom before making changes.

`APP_VERSION` in the code is still `'v2.1'` (the export filename suffix), but the codebase has landed several **post-v2.1.0, not-yet-released** changes on top of the tagged Ring release: an export-panel overhaul (Web/High/Max quality presets, frame-scrub stills, still SVG), animated aspect-ratio morphing, panel UI polish, and a mobile bottom-sheet prototype. Those are documented below and collected under **Unreleased** in `CHANGELOG.md`. Where this handoff and the older `v2.1.0` prose disagree, the **code + README are the source of truth**.

## Quick links

- **Live site:** https://nick-a8c.github.io/product-logos-slide-tool/
- **Source:** https://github.com/nick-a8c/product-logos-slide-tool
- **Owner:** Nebojsa Jurcic (`nick-a8c`)
- **Current version:** `v2.1` app-version tag; ring layout + unreleased export/morph/mobile work on `main`

## What this project is

A single-file HTML tool that composes brand animations from Automattic product icons and the AUTOMATTIC wordmark, then exports them. It ships **two layout modes**: a **four-segment Line timeline** (`Logos Intro → Logos Outro → A8C Intro → A8C Outro`) and a continuous 3D **Ring** orbit. Exports: **MP4, WebM, PNG, still SVG, and a self-contained interactive HTML page** (Ring). Built for a motion designer at Automattic (Nebojsa) for use during Radical Speed Month and beyond. Initially scoped as a self-serve alternative to MOGRT for non-Adobe users (sales/business folks). The whole thing lives in **one HTML file** with all dependencies inlined — drop it on any disk, double-click, it works offline.

> **GIF export was removed** in the post-v2.1.0 export overhaul. The `gifenc` library is still inlined (see below) but is now dead code — no `exportGIF` exists. Removing the library is a possible future cleanup.

The deliverable is `product-logos-slide-tool.html` (~387 KB currently). Plain HTML/CSS/JS, no build step, no framework. The repo also contains an identical `index.html` for GitHub Pages serving — **keep these two files in sync** after any edit (they are byte-identical today).

## Audience and tone

- User is **Nebojsa**, motion designer at Automattic, based in Subotica
- Communication style: informal, direct, action-oriented
- Prefers concise practical guidance over broad framing
- Will tell you what to fix in plain terms; assume good design taste, no need to over-explain
- Walk through GitHub clicks step-by-step when needed (he prefers UI workflow over CLI for repo tasks)

## Architecture

### Single-file, three script blocks

The HTML body has, in order:
1. **Inlined `gifenc`** (~9 KB IIFE-wrapped, exposes `window.gifenc`) — GIF encoder, **now unused** (GIF export removed post-v2.1.0; retained only because nothing references it and pulling it is low-priority)
2. **Inlined `mp4-muxer`** (~73 KB IIFE-wrapped, exposes `window.Mp4Muxer`) — MP4 container muxer
3. **Main app script** — everything else

Both libraries were inlined at v1.0.0 from the npm registry, wrapped in IIFEs to keep their internal short-named vars from leaking into global scope.

### State

`state` is a single global object persisted to `localStorage` under key `icon-stage-v2` (don't rename without a migration). Top-level fields:

- **Composition** (single global, applies to logo icons only): `count`, `spacing`, `scale`, `stageZoom`
- **A8C composition** (separate so the icon Spacing slider doesn't affect the wordmark): `a8cScale`
- **Animation mirror** (top-level fields that mirror the *active* segment's animation params): `duration`, `stagger`, `slideDistance`, `startScale`, `easing`, `auto`. These exist purely so legacy read sites (playAnimation, drawFrame, export functions) don't need per-call indirection. `syncTopFromActiveSegment()` / `syncActiveSegmentFromTop()` keep these in sync.
- **Per-segment animation params**: `state.segments = { logosIn, logosOut, a8cIn, a8cOut }`. Each contains `{ duration, stagger, slideDistance, startScale, easing, auto: {…}, reverseOrder }`. `reverseOrder` is outro-only.
- **Active segment**: `state.activeSegment` — one of `'logosIn' | 'logosOut' | 'a8cIn' | 'a8cOut'`. Drives stage rendering, the top-level mirror, and which animation panel is open.
- **Timeline pauses**: `state.segmentPauses = { logosInToOut, logosOutToA8cIn, a8cInToOut }` (seconds)
- **Export range**: `state.exportRange` — `'1' | '2' | '3' | '4' | 'all'`
- **Video quality**: `state.videoQuality` — `'web' | 'high' | 'max'` (default `'high'`), keys into `QUALITY_PRESETS`
- `bgOn`, `bgColor`, `aspectRatio`, `resolution`, `fps` — first-load defaults are Large (`3840x2160` for 16:9) and `fps: 60`
- `library` (uploaded SVGs), `assignments` (slot → icon id mapping)
- `theme`, `panelState`, `panelX`, `panelY`

### The four-segment model (v2.0.0)

Conceptually the animation is a sequence of four segments, played in this order:

| # | Segment | Content | Direction |
|---|---|---|---|
| 1 | `logosIn` | Logo icons | Intro (start → rest) |
| 2 | `logosOut` | Logo icons | Outro (rest → end) |
| 3 | `a8cIn` | AUTOMATTIC wordmark (10 letters) | Intro |
| 4 | `a8cOut` | AUTOMATTIC wordmark (10 letters) | Outro |

Helpers: `isA8cSeg(seg)`, `isOutSeg(seg)`, `getSegmentSlotCount(seg)` (10 for a8c, `state.count` for logos), `getSegmentTotalDuration(seg)` = `stagger * maxStep + duration`.

The active segment determines:
- Which content the stage shows (icons vs letters) via `renderStage()`'s `isA8cSegment()` branch
- Which animation params apply (via the top-level mirror)
- Which animation panel is auto-opened
- Which timeline pill is highlighted

### Animation engines

#### Intro engine (logosIn, a8cIn)

For each slot `i` of `n` total:
- `step = getMirrorStep(i, n)` — distance from center (0 for middle)
- `delay = step * stagger`
- Slot starts at `translateX(xOffset) scale(startScale) opacity(0)` and animates to `translateX(0) scale(1) opacity(1)` over `duration` ms
- `xOffset = getSlideOffset(i, n, slideDistance)` — signed pixel offset (outer slots move more)

#### Outro engine (logosOut, a8cOut)

Same mechanics, reversed direction: from rest → start state.
- `delay = (reverseOrder ? (maxStep - step) : step) * stagger`
- Slot starts at `translateX(0) scale(1) opacity(1)` and animates to `translateX(xOffset) scale(startScale) opacity(0)`

`reverseOrder` is the **Order of movement** toggle on the outro panels:
- **Inner first** (default, `reverseOrder=false`) — mirror-natural: middle leaves first
- **Outer first** (`reverseOrder=true`) — mirror-inverted: outermost leaves first

### A8C wordmark splitting

The Automattic logo SVG is embedded as `A8C_LOGO_SOURCE_SVG`. On first access, `getA8cLetters()`:
1. Appends the SVG into a hidden positioned `<div>` so `getBBox()` works
2. Groups the 11 source paths into 10 letter slots via `A8C_LETTER_GROUPS` (the O outline + the diagonal slash are combined into one "O" entry)
3. Computes each letter's bounding box (`w`, `h`, `srcX`, `srcY`)
4. Constructs a per-letter SVG with a viewBox cropped to the letter's bounding box

The letters are cached in `A8C_LETTERS_CACHE` after first access. `A8C_WORDMARK_REF_H = 43` is the reference line-height used to scale the wordmark — `scaleFactor = slotH / A8C_WORDMARK_REF_H`, where `slotH = getA8cSlotSize() = 80 * (state.a8cScale / 100)`. Inter-letter spacing comes from the source SVG positions (not the icon Spacing slider).

### Timeline UI

Sits inside `.stage-area` below the stage, above the floating bottom bar. Structure:

```
[Logos Intro pill | duration dropdown]
  [pause gap | pause dropdown]
[Logos Outro pill | duration dropdown]
  [pause gap | pause dropdown]
[A8C Intro pill | duration dropdown]
  [pause gap | pause dropdown]
[A8C Outro pill | duration dropdown]
```

Each pill is a `<button class="timeline-seg">` with a fill track + label + duration `<select>`. Each gap is a `<div class="timeline-gap">` with a short bar + pause `<select>`.

Dropdown options are 16 discrete values: `[0, 0.25, 0.5, 0.75, 1, 1.5, 2, 2.5, 3, 4, 5, 6, 7, 8, 9, 10]` seconds. The duration dropdown maps directly to `state.segments[seg].duration` (× 1000 ms); changing it also flips that segment's `auto.duration` to false. The Speed slider in the matching animation panel stays bidirectionally synced via `syncTimelineDropdowns()` (which is called inside `bindUIFromState` and at the end of the legacy slider handlers).

Clicking a timeline pill calls `setActiveSegment(seg)` AND immediately fires `playback.start('playing')` so the segment animates.

### Playback controller

`const playback = { mode, … }` is a tiny state machine over the play buttons. Modes:
- `'idle'` — at rest
- `'playing'` — single segment, auto-reverts after segment duration
- `'looping'` — single segment, repeats every cycle
- `'playing-all'` — async iteration through all 4 segments + pauses

PLAY ALL is `startPlayAll()` — an async loop that:
1. Calls `setActiveSegment(seg, { play: true })` for each segment in order
2. Waits `getSegmentTotalDuration(seg)` ms
3. Waits `getPauseSecondsBetween(seg, nextSeg) * 1000` ms before continuing
4. Checks `playback.mode !== 'playing-all'` between steps to allow cancellation

The `playback` controller's `updateUI()` paints all three buttons:
- PLAY SEGMENT: blue solid (active) / blue solid (idle)
- LOOP SEGMENT: outlined
- PLAY ALL: neutral fill (dark in light mode, light in dark mode — defined with explicit `:root[data-theme="dark"]` override so it stays visible in both themes)

### Aspect ratios + icon cap

Six ratios with fixed canvas dimensions (`ASPECT_RATIOS`): 16:9 / 21:9 / 4:3 / 9:16 / 4:5 / 1:1.

Vertical/square aspects (`9:16`, `4:5`, `1:1`) cap the icon count at **18** because more icons overflow the stage. `getMaxCountForAspect(ar)` returns 18 vs 25. `applyCountLimitForAspect()` updates the slider's `max` attribute and clamps `state.count` when switching.

A second auto-anchor table, `AUTO_ANCHORS_RESTRICTED`, provides tighter spacing/scale values for those aspects so 18 icons fit cleanly. `getAutoValue(key, n)` picks the restricted table when key ∈ {spacing, scale} and the active aspect is restricted.

### Export pipeline (v2.0.0 — heavily upgraded)

#### Sequencer

`buildExportTimeline()` returns `{ items: [{ seg, startMs, durMs, endMs }], totalMs }` for the selected range. `findSegmentForTime(timeline, t)` locates which segment owns a given global time. Inter-segment pauses come from `state.segmentPauses`.

Every per-frame export call goes through `drawSequenceFrameSupersampled(ctx, w, h, timeline, globalT, iconImages, a8cImages)`, which delegates to `drawSegmentFrameSupersampled(ctx, w, h, seg, localT, iconImages, a8cImages)`.

#### Per-segment frame renderer

`drawSegmentFrame()`:
1. Picks images (`iconImages` or `a8cImages`) and slot count from the segment
2. Computes per-slot widths + lead-gaps. For A8C: widths come from each letter's natural aspect at the A8C scale, gaps come from the source SVG positions scaled by `slotH / A8C_WORDMARK_REF_H`. For icons: uniform widths, gap = `state.spacing`.
3. Iterates slots: applies the segment's animation params (duration, stagger, easing, slideDistance, startScale) and direction (in vs out). For outros with `reverseOrder`, inverts the stagger delay.
4. Draws each slot's bitmap centered at its computed position with padding compensation (`padScaleX/Y`).

#### Supersampling

`drawSegmentFrameSupersampled()` allocates an offscreen canvas at `4× output resolution`, renders the segment frame into it, then downsamples to the output canvas via `halvingDownsample()` (each step is a 2:1 reduction, which Chrome reliably honors as proper averaging — single-shot large drawImage downsamples produce binary edges on transparent regions). SS factor is clamped by `MAX_SS_PIXELS = 64 MP` so 4K exports don't OOM.

#### SVG bitmap rasterization

`renderIconsToImages(renderSize)` and `renderA8cLettersToImages(renderSize)` pre-rasterize each SVG into a padded offscreen canvas:
- Target raster = `max(512, ceil(renderSize * 6))` so even tiny on-screen icons start from a high-res bitmap
- 6% transparent padding (`ICON_RASTER_PAD_RATIO`) on each side so the downsample filter has fringe room for AA
- Each bitmap carries `.padScaleX` / `.padScaleY` so `drawSegmentFrame` enlarges the draw rectangle to compensate

#### Quality presets (re-added post-v2.1.0)

The v2.0.0 single-preset design was **reverted** in the export overhaul — a three-way **Quality dropdown is back** for video, driven by `state.videoQuality` (`'web' | 'high' | 'max'`, default `'high'`). `QUALITY_PRESETS`:

| Key | Label | `videoBpp` | `keyframeIntervalSec` |
|---|---|---|---|
| `web` | Web (small) | `0.010` | `2` |
| `high` | High *(default)* | `0.030` | `2` |
| `max` | Max (lossless) | `1.000` | `0` (every frame a keyframe) |

`getQuality()` returns a copy of the active preset. Bitrate = `max(2_000_000, round(w * h * fps * videoBpp))`. `keyframeIntervalSec === 0` → every frame is a keyframe (Max); otherwise a keyframe every `fps * keyframeIntervalSec` frames. The old `gifMaxW`/`gifColors`/`gifKsize` fields are gone with GIF.

#### Export functions

- **WebM** (`generateWebMBlob`): VP9 with alpha when bg off, via `MediaRecorder` + `canvas.captureStream`. Routes through the sequencer. Uses the active quality preset's bitrate.
- **MP4** (`exportMP4`): H.264 via WebCodecs `VideoEncoder` + `mp4-muxer`. Even dimensions enforced. Tries codecs in order: `avc1.640034`/`640028` (High) → `4D0028` (Main) → `42E028` (Baseline). Routes through the sequencer. Uses the active quality preset's bitrate + keyframe interval.
- **PNG** (`exportPNG`): current still frame via `drawSegmentFrameSupersampled`. When paused-and-scrubbed it captures the scrubbed frame; otherwise the final frame of the *last* selected segment.
- **Still SVG** (`exportSVG`): a **single-frame vector still** (not the old SMIL animation). Places each icon as a nested static `<svg>` mapped from stage space to export space, with opacity and — in **Ring** mode — the depth-of-field blur **baked in as native `<feGaussianBlur>` filter defs**, so the still matches the on-screen DoF while staying scalable. Honors the current scrub frame.
- **Interactive HTML** (`exportRingHTML`, Ring only): self-contained page with the ring engine + interactions inlined. Replaces "Copy embed code" in ring mode.

The export panel is split into **Motion** (WebM / MP4 / Interactive HTML) and **Stills** (PNG / SVG). Animated SVG (SMIL full-sequence) is no longer produced — see What's pending.

### Auto mode

Animation autos are per-segment (in `state.segments[seg].auto`). Composition autos (spacing, scale) are top-level.

`recomputeAuto()` is called whenever `state.count` changes. It walks all 4 segments and recomputes any auto-enabled values from the `AUTO_ANCHORS` table interpolated at the current count.

`setSegmentAutoMode(seg, key, on)` is the per-segment version of `setAutoMode(key, on)`. The latter is for legacy top-level changes (active segment's auto via Overall Control buttons + the Animate IN panel's row toggles).

### Live preview crispness

Removed `will-change: transform, opacity` from `.icon-slot` in v2.0.0. With `will-change`, the browser baked each slot into a compositor-layer bitmap *before* the stage's `transform: scale()` ancestor downscale, causing visible pixelation. Without it, SVGs render vector-clean at the final composite scale.

Animation perf hasn't degraded — the GPU still handles transform/opacity transitions without the explicit hint.

### CSS variables for theming

All UI components must use `var(--button-secondary-bg)`, `var(--button-secondary-border)`, `var(--text)`, `var(--text-secondary)`, `var(--accent)` etc. — these flip correctly between light and dark modes.

Avoid hardcoded fallbacks like `var(--input-bg, #f6f6f6)` — those don't exist as variables, so the fallback applies in both themes. Use the actual defined vars instead.

For elements that need to look the same in both themes (e.g. the PLAY ALL button's distinct neutral pill), define an explicit `:root[data-theme="dark"]` override rather than relying on `--text` (which flips).

### Ring layout (v2.1.0)

A second stage layout alongside the classic line: a continuous 3D orbit with depth-of-field. Key facts:

- `state.layout` = `'line' | 'ring'`; ring params live in `state.ring` (defaults in `RING_DEFAULTS`, stage-level reset values in `RING_STAGE_DEFAULTS`). Ring mode hides the segment timeline, play buttons, segment panels, A8C section, and spacing/scale sliders via `.app[data-layout="ring"]` CSS.
- **Live engine**: `renderRingStage()` builds absolutely-positioned `.icon-slot`s; `ringFrame()` (rAF) places them on a tilted ellipse in canvas-px space. Depth (0=back, 1=front) drives scale, CSS blur, opacity, z-index.
- **Interactions** (preview + Interactive HTML export only, never video): hover grow/shrink with gaussian *spread* to neighbors; click ripple travelling both ways around the ring; drag-to-spin (grab/fling with Grip/Resistance/Weight physics, 720 deg/s cap, decays back into auto-rotation, 5px click-vs-drag threshold, pointer events so touch works).
- **Intro** (optional): per-icon staggered scale/fade-in at orbit positions, house easing; prepended to exports (`getRingIntroMs()`).
- **Exports**: `buildExportTimeline()` returns a single pseudo-segment `{seg:'ring'}` covering one revolution (`360/speed` s; +intro). `drawSegmentFrame()` branches to `drawRingFrame()` — a pure function of timeMs from `ringExportStartAngle` (frozen at export start). DoF uses pre-blurred bitmap variants cached per icon per quantized level (`getBlurredIconVariant`, 1/128 steps with a true zero level — coarser quantization was a bug that blurred everything except the focal icon). Video exports lock below speed 7 (`RING_MIN_VIDEO_SPEED`, `ringVideoLocked()`); PNG + Interactive HTML stay available. `exportRingHTML()` generates a self-contained interactive file (engine + config inlined, no external refs, prefers-reduced-motion parks the orbit). Animated SVG is guarded off in ring mode.
- **Gradient backgrounds** (both layouts): `state.bgMode` solid/linear/radial + `bgColorB/bgAngle/bgPos/bgSpread`. One source of truth: `getBgCSS()` (live stage + HTML embeds) and `fillBgCanvas()` (all canvas exporters, CSS-spec-matched), plus gradient defs in the SVG exporter.
- **Layout link** (chain button between LINE/RING): `state.layoutLink` (default on) keeps the shared settings (`LAYOUT_SHARED_KEYS` = count, zoom, background) mirrored. Off: each layout keeps its own copy in `state.layoutShared` and they swap on layout switch. Re-linking re-seeds both stashes from the active layout.
- **Per-layout Load defaults**: line keeps the historical preset (count 15); ring loads its designed preset (25 icons, zoom 48, radius 65, speed 3, depth 16, blur 16, fade 46, intro off, interaction off, solid white). Never mirrored by the link.

### Frame-scrub stills (post-v2.1.0)

PNG and still SVG can capture *any* exact frame, not just the settled one. Flow: **Pause Animation** (`btn-pause-anim`) → the stage enters scrub mode → **mouse-wheel or horizontal click-drag** over the stage moves the frame; export PNG/SVG grabs whatever's showing.

- `scrubPaused` (bool) gates it. Adds `.scrub-paused` to `#app` (ew-resize cursor, `touch-action: none`). The ring rAF freezes while scrubbing (the wheel drives the angle instead).
- **Line**: `scrubLineT` = ms into the active segment (starts at the settled frame `getSegmentTotalDuration`). Wheel step `LINE_SCRUB_STEP = 40` ms; `renderLineFrameAt(localT)` repaints.
- **Ring**: wheel step `RING_SCRUB_STEP = 2.5` degrees of orbit angle.
- Drag scrubbing is **absolute** (`scrubDragStart` / drag delta → frame), so a drag maps position → frame rather than accumulating.

### Animated aspect-ratio morphing (post-v2.1.0)

Switching aspect ratio now **tweens** the stage instead of snapping (live-preview nicety only — exports are untouched). Entry point `morphToAspect(ar)`; `applyAspectRatio(ar)` is still the instant path and is used directly when reduced-motion is set or nothing needs to move.

- **Strategy**: commit the real change first (so target geometry is readable from the DOM), then replay the visual *from* the old geometry *back to* the new over **~420 ms** with `cubic-bezier(0.4, 0.14, 0.3, 1)`.
- **Line** (`morphToAspect`): lerps stage width/height/scale, per-slot icon size, and row gap. Icons the new (smaller) ratio can't fit are snapshotted as `.morph-ghost` divs that fade + shrink out. Settles with `renderStage(true)`.
- **Ring** (`morphRingToAspect`): the ring loop already repositions every icon each frame, so it only tweens the stage dims (via `ringMorphCanvas`, read by `getCanvas()`/`applyZoom()`) and the four auto shape params (radius/tilt/angle/size). Dropped orbs fade as ghosts too.
- Respects `prefers-reduced-motion` (falls back to the instant `applyAspectRatio`). `aspectMorphRAF` holds the in-flight rAF so a rapid re-switch cancels cleanly.

### Mobile layout prototype (post-v2.1.0, experimental)

A first pass at phones, gated entirely behind `@media (max-width: 640px)` — **desktop layout is untouched**. Marked a *prototype*; not yet polished.

- The stage fills the top; the control panel becomes a **bottom sheet** that peeks as a grab handle (`#sheetHandle`, "Tap for settings") and slides up on tap. Toggle is a single `classList.toggle('sheet-open')` on `#app`.
- The segment timeline is hidden on mobile (too cramped); the floating bottom bar stacks vertically and fades out while the sheet is open.
- CSS `!important` overrides beat the inline top/left/transform the desktop dock JS writes. Touch-friendly min sizes on buttons/selects.
- Engine, animations, and exports are unchanged on mobile.

## Migrations in `restore()`

Existing `localStorage` saves get migrated:
- Legacy panel-state values (`'docked-open'` / `'docked-closed'`) → `'docked'`
- `stagger > 100` → divided by 4 (old 0–400 range, new 0–100)
- `slideDistance > 50` → reset to 0
- `spacing > 40` → linearly mapped from old 20–240 to new 0–40
- Missing `aspectRatio` → inferred from old resolution string
- Missing `auto` key → all auto flags set to `false` (preserve existing manual values)
- **v2.0.0**: missing `segments` → built from defaults; legacy top-level `duration`/`stagger`/etc. migrated into `segments.logosIn`. Missing `segmentPauses` → defaulted to `{ logosInToOut: 0.5, logosOutToA8cIn: 0.5, a8cInToOut: 0.5 }`. `reverseOrder` missing on any segment → false. `a8cScale > 80` clamped to 80 (the slider tightened from 40–200 to 10–80 in v2.0.0).
- **Post-v2.1.0**: missing/invalid `videoQuality` → `'high'` (`restore()` guards `if (!QUALITY_PRESETS[state.videoQuality]) state.videoQuality = 'high'`).

## What's done

All export formats work offline. Four-segment timeline (Logos Intro / Logos Outro / A8C Intro / A8C Outro) with per-segment animation panels. Auto mode for all 5 animation params per segment + Overall Control buttons (active segment scope). AUTOMATTIC wordmark split into 10 letter elements rendered with natural source-SVG spacing. Inter-segment pauses (0–10s) honored by both live PLAY ALL and export sequencer. Export range selector (1/2/3/4/ALL) feeds sequencer that swaps content/params/direction per segment. PLAY SEGMENT / LOOP SEGMENT / PLAY ALL. Order of movement toggle on outros. A8C scale slider (10–80) in its own section. 4× supersampled export rendering with halving downsample + 6× SVG bitmap raster + padding for AA fringe. Six aspect ratios with 18-icon cap on vertical/square. ~387 KB single-file HTML. Live on GitHub Pages.

**v2.1.0 adds:** Ring layout mode with full export support (one-revolution seamless loops), three live interactions (hover+spread / ripple / drag-to-spin), optional staggered intro, gradient backgrounds across all renderers, layout link toggle with per-layout setting stashes, per-layout Load defaults, Export Interactive HTML, and the export-DoF quantization fix.

**Post-v2.1.0 (unreleased, on `main`):** Export-panel overhaul — Motion/Stills split, **Web/High/Max video quality presets** back, **frame-scrub stills** (pause → wheel/drag → PNG/SVG of any exact frame), **still SVG** (single-frame vector, DoF baked as native SVG filters in Ring), and **GIF export removed**. **Animated aspect-ratio morphing** (Line + Ring, ~420ms, reduced-motion aware). Panel UI polish (neutral surfaces, hidden scrollbar, scroll masking). **Mobile bottom-sheet prototype** (≤640px, desktop untouched).

## What's pending or open

- **Animated (multi-frame) SVG export** — the SVG exporter now emits a **static single-frame still**, not the old SMIL animation. A full-sequence animated SVG (concatenated keyframes with correct SMIL timing) remains unimplemented and non-trivial.
- **Mobile layout is a prototype** — functional bottom-sheet pass behind `@media (max-width: 640px)`, but unpolished; needs real-device testing and refinement before it's called done.
- **Remove dead `gifenc`** — inlined but unused since GIF export was cut; ~9 KB of dead weight that could be dropped.
- **Per-letter A8C overrides** — every A8C letter currently animates with the same segment params. Per-letter offset/easing overrides would be a nice-to-have for fine-tuned wordmark choreography.
- **Node.js 20 deprecation warning** in the GitHub Actions workflow — should be fixed by June 2, 2026 (update `actions/checkout`, `actions/configure-pages` etc. to versions supporting Node 24).
- **Known limits**: MP4 won't preserve transparency (H.264); WebCodecs MP4 only works in Chromium + Safari 16.4+; Firefox lacks WebCodecs.
- **Version not yet cut** — `APP_VERSION` is still `'v2.1'` despite the unreleased work above. Bump it + tag a release (v2.2.0?) when this batch ships, and move the CHANGELOG **Unreleased** section under the new version.

## Conventions Claude should follow when iterating

- **One change at a time, small diffs.** Use `Edit` not full-file rewrites.
- **Always run a parse check** on the JS after risky edits to catch syntax errors before deploying.
- **Verify in the preview** (preview_eval) after any change that's observable in the browser. Don't claim "works" without inspecting state or DOM.
- **Keep the dual deliverable in sync.** After editing `index.html`, copy it to `product-logos-slide-tool.html`:
  ```bash
  cp index.html product-logos-slide-tool.html
  ```
- **Don't ask permission for tiny obvious things** but **do ask** when:
  - The user's spec is ambiguous
  - There are competing valid interpretations
  - A change has cross-cutting consequences (e.g. the v2.0.0 four-segment refactor was scoped via 4 upfront questions)
- **Prefer fixing the root cause** over patching symptoms
- **Trust user feedback over your own assumptions** about how things look — they're the designer
- **Save a fallback before risky changes.** When the user okays a destructive or hard-to-revert change, copy the current working file as `product-logos-slide-tool-vX.Y.Z-fallback.html` first.
- For GitHub workflow questions: this user prefers UI clicks over command line. Walk through screen-by-screen when needed.

## Moving this project to a personal account

The owner plans to continue this project from a personal Claude Max account (and possibly a personal GitHub account). For whoever picks this up there:

1. **This file is the context.** The new Claude account has no memory of prior sessions — read this handoff top to bottom; it replaces all accumulated session memory.
2. **GitHub transfer options** (owner prefers UI clicks):
   - *Transfer ownership*: repo Settings -> General -> Danger Zone -> "Transfer ownership" -> enter the personal account. GitHub auto-redirects the old URL; issues/releases/tags move along.
   - *Or fork/import* into the personal account if the `nick-a8c` copy should stay.
3. **Re-enable GitHub Pages** after transfer: Settings -> Pages -> Deploy from branch -> `main` / root. The live URL changes to `https://<personal-account>.github.io/product-logos-slide-tool/`. Update the Quick links above afterwards.
4. **Local checkout**: update the remote — `git remote set-url origin https://github.com/<personal-account>/product-logos-slide-tool.git`.
5. **Releases**: `v2.0` (pre-ring stable) and `v2.1.0` exist as tags/releases — they travel with an ownership transfer, but NOT with a fork (re-create from tags if forking).

## Useful one-liners

Find a function:
```bash
grep -n "function exportMP4\|setActiveSegment\|drawSegmentFrame\|startPlayAll" index.html
```

Parse-check all script blocks:
```bash
node -e "
const fs = require('fs');
const html = fs.readFileSync('index.html', 'utf8');
const re = /<script>([\\s\\S]*?)<\\/script>/g;
let m, idx = 0;
while ((m = re.exec(html))) {
  idx++;
  try { new Function(m[1]); }
  catch(e) { console.error('Block', idx, 'PARSE ERROR:', e.message); }
}
console.log('Parsed', idx, 'script blocks OK');
"
```

Keep the deliverables synced:
```bash
cp index.html product-logos-slide-tool.html
```

Git deploy:
```bash
git add -A
git commit -m "your message"
git push origin main
```
