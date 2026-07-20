# Changelog

All notable changes to this project will be documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [2.3.1] — 2026-07-20

Onboarding and panel refinements from a walkthrough.

### Changed

- **Setup Assistant drops the "Who are you?" role step.** It only pre-sorted the use-case cards and set nothing on its own, so the assistant now opens straight on *What are you making?* — four steps instead of five (use-case → layout → brands → review).
- **Control (Overall Control AUTO / CUSTOM) is now in Essentials** for both Line and Ring, not just Advanced.
- **Undo / Redo buttons are now labelled.** The circular icon-only buttons became labelled pills (*‹ Undo* / *Redo ›*) so they're not mistaken for a prev/next stepper.

### Fixed

- **First-run tutorial can return to the Welcome fork.** Choosing *Show me how it works* was a one-way trip; its first step now shows **Back**, which returns to Welcome with the intro backdrop intact. (Opening the tutorial from the persistent launcher still has no first-step Back.)
- **Tutorial "Next" no longer drifts left on the first step.** With no Back button to balance it, the Skip/Next group slid to the centre; it's now anchored to the right on every step (`.wiz-foot-right { margin-left: auto }`), which also keeps the assistant's footer consistent.

## [2.3.0] — 2026-07-14

The **Panel** release. v2.2 made the tool approachable on the way *in*; this one makes it approachable once you're *there*. The settings panel went from **39 controls on screen to about 7** — with nothing removed and nothing renamed.

### Added

- **Essentials / Advanced tiers.** A two-button toggle at the top of the panel. **Essentials** (the default) shows only the decisions a user actually makes; **Advanced** reveals every control the tool has ever had. Twenty-nine sections and controls are tagged `data-tier="advanced"` and hidden by one CSS rule (`.panel-content:not(.adv) [data-tier="advanced"] { display: none }`), so nothing is deleted, unbound, or moved — it's the same DOM, just filtered. Remembered per user (`state.panelTier`).
- **Motion "Feel"** — Calm / Balanced / Snappy, an Essentials-level replacement for twenty timing sliders. Presets are **absolute durations, not multipliers**, so re-picking the same feel is idempotent (`FEEL_PRESETS`). Hand-tuning any timing slider flips the state to *Custom* (`markMotionCustom()`) rather than silently lying about which preset you're on. Line only — Ring's motion is its Speed/Direction, which live in the Ring section.
- **AUTO note.** While AUTO is on, Essentials shows one line — *"Spacing & scale sized automatically"* — with a **Customise** button, instead of six greyed-out sliders. Customise switches to Advanced and hands over control.
- **First-run intro backdrop.** A drifting accent grid (SVG `<pattern>`, 56px tile, 45°, ~6px/s) behind the whole first-run flow, replacing the blurred-out live UI. It's a standalone `#introBackdrop` layer, so it **persists across Welcome → Setup Assistant → tutorial** and only comes down when the user actually lands in the tool. Parked under `prefers-reduced-motion`.

### Changed

- **The Motion panel is now contextual.** Four near-identical Motion sections (one per segment — 22 of the panel's 39 controls were the same 5 knobs repeated) collapsed into **one** section with a segment toggle: *In / Out / A8C in / A8C out*. It retargets via `panel.dataset.segment`, so the per-control AUTO toggles follow the active segment. *Order of movement* only appears for the two outro segments, where it means something.
- A **returning** user reopening the Setup Assistant still gets the normal translucent overlay over their own composition — the opaque pattern backdrop is first-run only (`endIntro()` is a no-op outside the intro flow).
- `APP_VERSION` bumped to `v2.3` (export filename suffix).

### Notes

- No control was removed and no state field changed meaning — an Advanced user sees exactly the v2.2 panel.
- `panelTier` and `motionFeel` were added to `serializeState()`. They must be: `persist()` uses a curated allow-list, so a new state field that isn't listed there is silently never saved.

## [2.2.0] — 2026-07-14

The **Onboarding** release. A guided **Setup Assistant**, in-app templates, a step-through tutorial, undo/redo, and count-driven Ring composition — so a marketer, designer, sales person or developer can all get a good result without learning every slider.

### Added

- **Welcome fork** on first run — one modal (never two stacked) offering *Setup Assistant* / *Show me how it works* / *Just start*. Gated by the new `state.onboarded`; existing users are never re-onboarded.
- **Setup Assistant** (`#btn-open-wizard`, top of the panel — a holographic pill). A dismissible 5-step wizard over the tool:
  1. **Role** (Marketing / Design / Sales / Engineering / Just exploring) — only pre-sorts the use-case cards.
  2. **Use-case** — drives layout, aspect, size, format and motion "feel" (Social Reel, Social square, Slide, Website hero, Logo loop, Email banner, Wallpaper).
  3. **Line or Ring** — two **animated thumbnails**; pre-selected from the use-case, but the user's pick wins.
  4. **Brands** — 9×3 grid with a **live preview** of the chosen layout that updates as icons are toggled.
  5. **Review** → *Open in tool*, plus optional *Save as template*.
  Guided setup always lands on a **solid white background**. Everything stays fully editable afterwards.
- **Animated layout thumbnails** — pure CSS/SVG (no Lottie dependency): Line = 5 dots growing/fading with a centre-out stagger; Ring = an 8-dot tilted orbit with depth. Both share one blue so they read as a set.
- **In-app templates** — save a named setup and reuse it. Stored in `localStorage` under `icon-stage-templates-v1`, separate from the main state. Appears as a "Start from a template" step 0 in the assistant (load / delete), and "Save as template" on the review step. Design fields only — a template never carries your theme or panel position.
- **Tutorial** — a 5-step walkthrough (brands → Line vs Ring → motion → background & aspect → export), reachable from the Welcome fork or a persistent bright **"How it works"** button. Progress dots, Back/Next/Skip, Esc, and arrow-key navigation. *(Media slots are placeholders pending the real clips.)*
- **Undo / Redo** — two circular buttons in the bottom bar plus **`Ctrl/Cmd+Z`**, **`Ctrl/Cmd+Shift+Z`** and **`Ctrl+Y`**. In-session history, 50 steps deep, debounced so a slider drag is a single undo step. Ignores UI chrome, so undo never flips your theme or moves the panel.
- **Ring composition AUTO now scales with icon count** — radius and icon size follow a **per-aspect** anchor table (`RING_COUNT_ANCHORS_BY_ASPECT`), the Ring analogue of Line's Spacing/Scale. Tilt and angle still follow the aspect. Interpolates between anchors and holds flat below the smallest one:

  | Aspect | anchors (icons → radius / size) |
  |---|---|
  | 16:9 & 21:9 | 3→20/18 · 10→29/15 · 16→40/14 · 25→48/12 |
  | 4:3 | 7→29/14 · 25→57/9 |
  | 9:16 | 7→60/11 · 25→87/7 |
  | 4:5 | 7→47/14 · 25→81/10 |
  | 1:1 | 7→61/17 · 25→75/10 |

- **Overall Control refresh button** — a one-click "Reset to ideal default" (same as Load defaults, layout-aware) with a custom hover tooltip.

### Changed

- `setLayout()` hoisted from a `bindEvents` closure to a global function so the Setup Assistant can switch layout programmatically.
- `serializeState()` factored out of `persist()` so the template store and undo history snapshot exactly what the app persists.
- `APP_VERSION` bumped to `v2.2` (export filename suffix).

### Fixed

- **Existing users were being re-onboarded on upgrade.** The "has this user seen onboarding" migration tested `state.onboarded`, which already defaults to `false` (a boolean), so the check never fired. Now tests the *saved* data.
- New state fields silently failed to save because `persist()` uses a curated allow-list rather than a full state dump — the onboarding flags are now included.
- The Setup Assistant's live preview could size its ring against a stale container width and cluster every icon into one corner; it now retries once layout settles.
- The Setup Assistant button's top corners were clipped unevenly by the panel's rounded top + `overflow:hidden`.

## [2.1.1] — 2026-06-30

An export-panel overhaul, animated aspect morphing, UI polish, and a mobile prototype.

### Added

- **Video quality presets** are back. A **Quality** dropdown for MP4/WebM with three presets driven by `state.videoQuality`: **Web (small)** (`videoBpp 0.010`, keyframe every 2s), **High** *(default)* (`0.030`, every 2s), **Max (lossless)** (`1.000`, every frame a keyframe). Bitrate = `max(2 Mbps, w·h·fps·videoBpp)`.
- **Frame-accurate stills.** A **Pause Animation** toggle puts the stage into scrub mode; **mouse-wheel or horizontal click-drag** across the stage moves the frame (Ring → orbit angle at 2.5°/notch, Line → segment time at 40ms/notch). PNG and SVG then export that exact frame. Drag scrubbing is absolute (position → frame).
- **Still SVG export.** The SVG exporter now produces a **single-frame vector still**: each icon nested as a static `<svg>` mapped into export space. In **Ring** mode the depth-of-field blur is **baked in as native `<feGaussianBlur>` filters**, so the still matches the on-screen DoF while staying scalable.
- **Animated aspect-ratio morphing** (live preview only; exports unchanged). Switching aspect ratio tweens the stage over ~420ms with `cubic-bezier(0.4, 0.14, 0.3, 1)` instead of snapping. Line lerps stage/icon/gap geometry; Ring tweens stage dims + the four auto shape params and lets the orbit reshape itself. Icons the new ratio can't fit fade out as ghosts. Respects `prefers-reduced-motion` (falls back to the instant path).
- **Mobile layout prototype** (`@media (max-width: 640px)`, desktop untouched). Stage fills the top; the control panel becomes a **bottom sheet** that peeks as a grab handle and slides up on tap. Timeline hidden on phones; bottom bar stacks and fades out while the sheet is open; touch-friendly control sizes. Marked experimental.

### Changed

- **Export panel** reorganized into **Motion** (WebM / MP4 / Interactive HTML) and **Stills** (PNG / SVG).
- Panel UI polish: neutral surfaces, hidden scrollbar, scroll masking.

### Removed

- **GIF export.** The Motion panel no longer offers GIF; `exportGIF` is gone. The inlined `gifenc` library remains but is now unused (dead code, candidate for future removal).
- The previous **animated (SMIL) SVG** export — the SVG output is now a static still (see Added). Full-sequence animated SVG remains unimplemented.

## [2.1.0] — 2026-06-12

The **Ring layout** release. A second stage layout — a continuous 3D orbit with depth-of-field — alongside the classic line, with its own interactions, exports, and defaults.

### Added

- **Ring layout mode.** LINE/RING toggle in the new Layout panel section. Icons orbit a tilted ellipse; depth along the ring drives scale, blur (DoF), opacity, and stacking. Controls: Radius, Tilt, Angle, Speed, Direction (forward/reverse), Icon Size, Depth Scale, Blur, Fade Back. Ring mode bypasses the segment timeline.
- **Ring interactions** (live preview + Interactive HTML export; not in video):
  - *Hover* grow/shrink with adjustable amount and **spread** to ring neighbors (gaussian falloff).
  - *Click ripple* — a wave travelling both ways around the ring (strength + speed).
  - *Drag-to-spin* — grab and fling the ring; Grip / Resistance / Weight controls, fling capped at 720°/s, decays back into auto-rotation. Touch works via pointer events.
- **Ring intro** (optional): icons scale/fade in at their orbit positions, staggered around the ring; prepended to exports.
- **Ring exports.** WebM/MP4/GIF render exactly one revolution (360 ÷ speed) for a seamless loop; DoF via pre-blurred bitmap variants cached per icon. PNG renders the settled ring. **Export Interactive HTML** produces a self-contained file with the engine + interactions inlined (replaces Copy embed code in ring mode). Video exports lock below speed 7 with an explanatory hint. Animated SVG is not available in ring mode yet.
- **Gradient backgrounds** (both layouts): solid / linear / radial with Color B, angle (linear), center (radial), and spread — applied consistently across the live stage, canvas exports, SVG export, and HTML embeds.
- **Layout link toggle** (chain icon between LINE/RING): when on, the settings both layouts share (icon count, zoom, background) stay mirrored; when off, each layout keeps its own copy and they swap on layout switch. Re-linking transfers the active layout's values to the other.
- **Per-layout Load defaults**: line keeps its historical preset; ring loads its designed preset (25 icons, zoom 48, radius 65, speed 3, depth 16, blur 16, fade 46, intro off, interaction off, solid white).

### Fixed

- Export DoF blur quantization forced a heavy minimum blur, leaving only the focal icon sharp in PNG/video frames. Now quantizes at 1/128 of the icon with a true zero level — exports match the preview's gradual falloff.

### Changed

- `APP_VERSION` bumped to `v2.1` (export filename suffix).

## [2.0.1] — 2026-05-19

A polish pass on top of v2.0.0 — defaults that match the intended choreography, timeline-bar animation that tracks playback time, and a couple of boot-order bug fixes.

### Added

- **Animated timeline progress bars.** Each segment's track fills 0→100% over the segment's actual animation duration, then fades opacity → 0 before the next phase. Inter-segment pause tracks animate the same way over the pause duration. Single PLAY SEGMENT bar settles back to the static blue resting state after the animation finishes.

### Changed

- **First-load defaults retuned to a sensible "designed" choreography**:
  - Overall Control = AUTO
  - Logos Intro = 1s (auto-derived), pause = 2s
  - Logos Outro = 700ms / stagger 35 / slide 6 / startScale 70 / Outer-first
  - Pause = 0s
  - A8C Intro = 1s (auto-derived), pause = 2s
  - A8C Outro = 900ms / stagger 21 / slide 0 / startScale 55 / Outer-first
- **Default export Resolution = Large** (3840×2160 for 16:9 boot) on first install.
- A8C scale slider tightened to the **10–80** range (was 40–200); default 40.

### Fixed

- `applyAspectRatio()` no longer clobbers a still-valid current resolution on boot — it now only resets to the ratio's Standard when the current resolution isn't among that ratio's small/standard/large options.
- Timeline bar no longer "bounces back" after fade-out — the fade ends in the empty resting state in a single silent step rather than clearing inline styles and letting the CSS width transition animate 100→0%.
- `APP_VERSION` bumped to `v2` (changes export filename suffix to `_v2.<ext>`).

## [2.0.0] — 2026-05-19

A major rebuild. The single-segment intro animation is now a **four-segment timeline** (Logos Intro → Logos Outro → A8C Intro → A8C Outro), with a new AUTOMATTIC wordmark asset, sequenced exports, and a substantially upgraded export rendering pipeline.

### Added

- **Four-segment timeline UI** under the stage canvas. Each segment shows a duration dropdown (0–10s) and a track-fill indicator; between segments are pause dropdowns (0–10s). Click a segment pill to focus + replay it.
- **A8C wordmark** as a new asset type. The AUTOMATTIC logo is parsed once on load into 10 individual letter elements (with the O outline + slash combined), each animated like an icon.
- **Per-segment animation panels** — the old "Animation" section is now **Animate IN**, joined by **Animate OUT**, **A8C Intro**, and **A8C Outro**. Each segment owns its own duration / stagger / slide distance / start scale / easing + per-control auto flags. Animate IN is open by default; the others collapse.
- **Outro engine** — `Animate OUT` and `A8C Outro` play the animation in reverse (rest → slid out, scaled down, faded). Slider values share semantics with the matching intro, so an outro with the same values cleanly mirrors the intro.
- **Order of movement** toggle on both outro panels — flips stagger order between *Inner first* (mirror-natural, default) and *Outer first* (mirror-inverted).
- **A8C scale** lives in its own panel (separate from Composition), range 10–80. The wordmark uses its natural source-SVG letter spacing, not the icon Spacing slider.
- **Export range selector** (`1 / 2 / 3 / 4 / ALL`) above the export buttons. Picks which segments the exporters emit. `ALL` concatenates all four back-to-back with the chosen inter-segment pauses.
- **Sequencer rendering** — `MP4 / WebM / GIF / PNG` exports route through a timeline that swaps content (icons vs letters), animation params, and direction per-segment. PNG renders the final frame of the last selected segment.
- **PLAY ALL** button — walks the full timeline on the live stage, honoring each segment's duration and the inter-segment pauses.
- **PLAY SEGMENT** / **LOOP SEGMENT** — the existing PLAY / LOOP, renamed to clarify scope vs the new PLAY ALL.
- **Aspect-aware icon cap**: 9:16 / 4:5 / 1:1 cap the icon count at **18** (icons overflow above that). Auto anchors retuned for those aspects so the cap fits cleanly (Spacing 10, Scale 53).

### Changed

- **Export quality overhaul.** Composition now uses **4× full-frame supersampling** rendered into an offscreen buffer, then downsampled via a halving pyramid (Chrome reliably honors `imageSmoothingQuality='high'` only at 2:1 ratios). SVG bitmap raster bumped to **6× the on-screen icon size** with a **512 px floor** and a **6% transparent margin** per side so AA fringe survives the downsample. Supersample buffer capped at 64 MP so 4K exports don't OOM.
- **Live preview crispness** — removed `will-change: transform, opacity` from `.icon-slot`. Without it, the browser stops baking each slot into a compositor bitmap before the stage downscale, and SVGs render vector-clean at any zoom.
- **Single, top-tier quality preset.** The Quality dropdown was removed entirely; every export now uses the highest preset (1.0 bpp video, every-frame keyframes, full-resolution GIF with rgb565 palette, 30fps GIF).
- **State schema** restructured around `state.segments` (per-segment params + auto flags) with a top-level mirror of the active segment for backwards compatibility with read sites. Legacy saves migrate transparently.

### Fixed

- A8C scale slider, segment toggle pills, and timeline dropdowns adapt correctly in dark mode (previously used hardcoded light backgrounds).
- The PLAY ALL button stays visually distinct in both themes (dark fill in light mode, light fill in dark mode).
- Timeline UI no longer overlaps the floating aspect / play / loop bar (stage area reserves clearance).

## [1.2.0] — 2026-05-04

### Added
- **Overall Control section** at the top of the side panel with global **AUTO** and **CUSTOM** buttons. Flip every Auto-capable control (Spacing, Scale, Speed, Stagger, Slide dist., Start scale, Easing) to the same mode in one click.
- **Mixed state indicator**: when any per-slider override breaks uniformity, both Overall buttons display as outlined-and-dimmed (no blue), signaling that the current mode is partial.

### Changed
- Per-slider AUTO/CUSTOM toggles continue to work as before. The new global buttons are an additional shortcut, not a replacement.

## [1.1.0] — 2026-05-01

### Added
- Default library expanded from 9 to **25 Automattic brand icons**
- **Curated layout system** (`DEFAULT_LAYOUTS`): every icon count from 2 to 25 has a hand-arranged set of icons. Anchor counts (2, 5, 9, 15, 25) follow specific aesthetic groupings.
- **Auto-fill on count grow**: when raising the icon count slider, empty trailing slots are populated from the layout for that count (skipping icons already manually placed). Manual placements are preserved.
- Stable icon IDs (`'01'` through `'25'`) for default library icons; user uploads continue to receive random IDs.

### Changed
- **Default icon count**: 9 → 15 on first load.
- **Default state values** (spacing 18, scale 75) now match the n=15 auto-mode anchor.

### Fixed
- (carried from v1.0.1) Icon assignments are no longer lost when lowering then raising the icon count slider.
- (carried from v1.0.1) Library "in-use" indicator now only counts visible slots.

## [1.0.1] — 2026-05-01

### Fixed
- Icon assignments are no longer lost when lowering then raising the icon count slider.

## [1.0.0] — 2026-05-01

Initial public release.

### Added
- Six aspect ratios (16:9, 21:9, 4:3, 9:16, 4:5, 1:1)
- Per-control Auto mode for Spacing, Scale, Speed, Stagger, Slide dist., Start scale, Easing
- Three quality presets (Low / Medium / High); High uses every-frame keyframes for visually-lossless MP4
- MP4 export via native WebCodecs `VideoEncoder` + `mp4-muxer`
- WebM export with VP9 alpha when background is off
- GIF export via inlined `gifenc`
- Animated SVG export via SMIL
- PNG final-frame export
- Embed code snippet copy
- Light + dark theme
- Dock / undock / hide panel UX
- Mirror-staggered animation with proportional, signed slide distance
- localStorage persistence with backward-compatible migrations
- Live deployment via GitHub Pages
