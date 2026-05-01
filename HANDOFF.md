# Product Logos Slide Tool — Project Handoff

This document captures the state of the **Product Logos Slide Tool** (internal nickname: *Icon Stage*) as of v1.1.0 so a fresh Claude session can pick up from here without re-deriving context. Read it top to bottom before making changes.

## Quick links

- **Live site:** https://nick-a8c.github.io/product-logos-slide-tool/
- **Source:** https://github.com/nick-a8c/product-logos-slide-tool
- **Owner:** Nebojsa Jurcic (`nick-a8c`)
- **Current version:** v1.1.0

## What this project is

A single-file HTML tool that animates a row of brand icons appearing center-outward and exports the result as MP4, WebM, GIF, animated SVG, or PNG. Built for a motion designer at Automattic (Nebojsa) for use during Radical Speed Month and beyond. Initially scoped as a self-serve alternative to MOGRT for non-Adobe users (sales/business folks). The whole thing lives in **one HTML file** with all dependencies inlined — drop it on any disk, double-click, it works offline.

The deliverable is `product-logos-slide-tool.html` (~240 KB as of v1.1.0). Plain HTML/CSS/JS, no build step, no framework. The repo also contains an identical `index.html` for GitHub Pages serving.

## Audience and tone

- User is **Nebojsa**, motion designer at Automattic, based in Subotica
- Communication style: informal, direct, action-oriented
- Prefers concise practical guidance over broad framing
- Will tell you what to fix in plain terms; assume good design taste, no need to over-explain
- This is the user's first serious GitHub project — be patient with git/GitHub workflow questions, but don't over-explain the code itself

## File layout in the working directory

```
/home/claude/icon-tool/
  product-logos-slide-tool.html              # The single deliverable. All code, all deps, all SVGs.
  product-logos-slide-tool-v1.0.1-fallback.html  # Pre-25-icon snapshot, kept for rollback.
  HANDOFF.md                                  # This doc.
```

Outputs go to `/mnt/user-data/outputs/product-logos-slide-tool.html` and are surfaced via `present_files` after each iteration.

## Architecture

### Single-file, three script blocks

The HTML body has, in order:
1. **Inlined `gifenc`** (~9 KB IIFE-wrapped, exposes `window.gifenc`) — GIF encoder
2. **Inlined `mp4-muxer`** (~73 KB IIFE-wrapped, exposes `window.Mp4Muxer`) — MP4 container muxer
3. **Main app script** — everything else

Both libraries were sourced from the npm registry (the only allowed CDN in this environment), wrapped in IIFEs to keep their internal short-named vars from leaking into global scope, and inlined so the file is fully self-contained.

### State

`state` is a single global object holding everything. Persisted to `localStorage` under key `icon-stage-v2` (kept this key name for backward compatibility — don't rename without a migration). Key fields:

- `count`, `spacing`, `scale`, `stageZoom` — composition (default count = 15 in v1.1.0)
- `duration` (ms), `stagger` (ms), `slideDistance`, `startScale`, `easing` — animation
- `auto: { spacing, scale, duration, stagger, slideDistance, startScale, easing }` — per-control auto-mode flags. **All true by default for fresh installs.** Existing saves (no `auto` key) get them all `false` for backward compatibility.
- `bgOn`, `bgColor`, `aspectRatio`, `resolution`, `quality`, `fps`
- `library` (uploaded SVGs), `assignments` (slot → icon id mapping)
- `theme`, `panelState` (`docked` / `undocked` / `hidden`), `panelX`, `panelY`

### Default icon library and curated layouts (v1.1.0)

`DEFAULT_ICONS` ships 25 Automattic brand SVGs with **stable IDs** `'01'`–`'25'` (matching their numeric prefix in the source filenames: `01_tumblr.svg`, `02_p2.svg`, etc.). User-uploaded icons still get random IDs from `uid()`. Numeric-string IDs and 8-char hex IDs don't collide.

`DEFAULT_LAYOUTS[count]` returns an array of icon IDs (in slot order, left to right) for any count from 2 to 25. Anchor counts have hand-curated arrangements:

- **2:** `[10, 19]` (WordPress.com, WordPress VIP)
- **5:** `[01, 10, 17, 18, 19]` (Tumblr, WP.com, Jetpack, Woo, WP VIP)
- **9:** `[01, 10, 11, 12, 16, 17, 18, 19, 24]`
- **15:** `[01, 02, 07, 08, 09, 10, 11, 12, 15, 16, 17, 18, 19, 20, 24]` ← **default count on fresh load**
- **25:** all icons in numeric order

Counts between anchors inherit from the lower anchor and append icons in numeric order from the next anchor's set. **At anchor crossings (5, 9, 15, 25) the slot order re-shuffles to the curated arrangement** — this is intentional, per Nebojsa's spec.

### Auto-fill behavior

`autoFillNewSlots()` runs after the count slider changes. It pulls icons from `DEFAULT_LAYOUTS[count]`, **skipping IDs that are already in earlier slots** (so manual placements are preserved), and fills `null` slots from the remaining pool. If a layout references an icon not in the user's library (e.g. they removed it), that icon is silently skipped. Result: empty slots only appear when the user explicitly removes an icon from the library.

### Auto mode (the big one)

Every animation-affecting control has Auto mode that derives its value from `state.count`. Anchor table at counts `[2, 5, 9, 15, 19, 25]`, linearly interpolated for in-between counts, clamped at endpoints. See `AUTO_ANCHORS` in code.

UI: hover the label to reveal an `AUTO`/`CUSTOM` toggle. Auto-on shows `A` in the value field, dims slider+thumb to non-interactive. Manually moving any slider auto-flips that control to Custom. Easing is special: dropdown stays clickable in Auto mode, picking any concrete easing flips to Custom; picking the "Auto (Original)" option flips to Auto.

### Animation model

- Icons appear center-outward with **true mirror staggering**: `getMirrorStep(i, n)` returns distance-from-center for index `i` of `n` total. Pairs equidistant from center fire simultaneously.
- Slide distance: slider `-50` to `+50`. Negative = icons start compressed inward and flare out; positive = start spread outward and contract in. Outermost icons travel ±100px max (slider × 2). Travel scales linearly with distance from center; middle icon doesn't move.
- Default easing: `cubic-bezier(0.28, 0, 0, 1)` ("Original")

### Aspect ratios

Six ratios with fixed canvas dimensions (`ASPECT_RATIOS` table): 16:9 / 21:9 / 4:3 / 9:16 / 4:5 / 1:1. Each maps to a canvas size used for both rendering and as the default export resolution. Resolution dropdown is auto-populated when the user picks a ratio.

### Panel UX

Top-right floating card, 340px wide. Three states: `docked` (default, top-right) / `undocked` (floating, ~20% shorter, vertically centered, draggable via 8-dot handle) / `hidden` (offscreen).

Two stacked buttons on the panel's left edge:
- **Top button = dock/undock toggle**. Arrow points ↖ when docked (click to undock), ↘ when undocked (click to re-dock).
- **Bottom button = hide**. Always shows `>`. Click to slide panel offscreen right.

When hidden, a small `<` tab appears centered on the right edge of the screen. Click to restore to whichever state the panel was in before hiding (`docked` or `undocked`).

### Centered top/bottom bars

Light/dark switch (top-center) and aspect-ratio + play/loop bar (bottom-center) both use `--right-offset` CSS variable so they recenter relative to the *stage area*, not the viewport. Set in `setPanelState`:
- `docked` → `--right-offset: 388px`
- `undocked` / `hidden` → `--right-offset: 0px`

### Quality system

`QUALITY_PRESETS` table with `low`/`medium`/`high`. Affects MP4, WebM, GIF (PNG and SVG are lossless, unaffected). `getQuality()` returns the active preset.

- `videoBpp`: bits per pixel per frame for video bitrate calc → `Math.max(2_000_000, w*h*fps*videoBpp)`
- `gifMaxW`, `gifColors`, `gifKsize`: GIF width cap, palette color count, color-space format
- `keyframeIntervalSec`: MP4 keyframe interval. `0` means "every frame is a keyframe" (used by High for visually lossless output)

### Export pipeline

- **Quality fix**: `renderIconsToImages(renderSize)` pre-rasterizes each SVG to an offscreen canvas at 2× the target render size before any export. Without this step, browsers rasterize SVGs at default ~300px and `ctx.drawImage` upscales, producing fuzzy edges.
- **WebM**: VP9 with alpha when bg off, via `MediaRecorder` + `canvas.captureStream`
- **MP4**: H.264 via WebCodecs `VideoEncoder` API + `mp4-muxer`. Even dimensions enforced. Tries codecs in order: `avc1.640034`/`640028` (High) → `4D0028` (Main) → `42E028` (Baseline). Fully offline. Chrome/Edge/Safari 16.4+ only (no Firefox).
- **GIF**: gifenc with palette quantization. No workers.
- **Animated SVG**: SMIL via inline `<animateTransform>` / `<animate>` elements
- **PNG**: final frame rasterized via the same canvas pipeline
- **Embed code**: CSS keyframes HTML snippet via clipboard

### Filename format

`buildFilename(ext, extra?)` → `product-logos-slide-tool[_extra]_<resolution>_<quality>-q_v1.<ext>`. Examples:
- `product-logos-slide-tool_1920x1080_high-q_v1.mp4`
- `product-logos-slide-tool_1080x1920_medium-q_v1.gif`
- `product-logos-slide-tool_preset_1920x1080_medium-q_v1.json`

`APP_VERSION` constant controls the `v1` suffix — bump on major changes.

## Critical CSS / JS patterns to know

- `.stage-inner` needs `flex-shrink: 0` to prevent flex parent shrinking (was a bug where 16:9 canvas displayed squashed)
- `pointer-events: none` on `.ctrl.is-auto .slider` but **not** on `.row.is-auto select` (dropdown stays clickable in Auto mode for easing)
- All `.val` number inputs are `type="text"` — number inputs silently reject non-numeric strings, so they couldn't display "A" for Auto mode

## Migrations in `restore()`

Existing `localStorage` saves get migrated:
- `panelState: 'docked-open'` / `'docked-closed'` → `'docked'`
- `stagger > 100` → divided by 4 (old 0–400 range, new 0–100)
- `slideDistance > 50` → reset to 0
- `spacing > 40` → linearly mapped from old 20–240 to new 0–40
- Missing `aspectRatio` → inferred from old resolution string
- Missing `auto` key → all auto flags set to `false` (preserve existing manual values)

**v1.1.0 note:** Existing users with localStorage from v1.0.x have a 9-icon library with random IDs. They will NOT get the new 25-icon library automatically (their library is preserved). When they slide count above what they have, `autoFillNewSlots` will fail to find icons matching `DEFAULT_LAYOUTS` IDs (`'01'`..`'25'`) and leave those slots empty. They can hit "Reset / Load defaults" to upgrade. This is acceptable behavior.

## What's done

V1.1.0. All export formats work offline. Auto mode for all 7 controllable parameters. Quality presets (low/medium/high) for video + GIF. Light/dark theme. Six aspect ratios. Dock/undock/hide panel UX. Top and bottom bars track stage area. **25 default brand icons** with curated layouts at every count. Default count = 15 on fresh load. Auto-fill on count grow with manual placements preserved. Mirror-staggered animation with proportional slide distance. Descriptive filenames. localStorage persistence with migrations. ~240 KB single-file HTML. Live on GitHub Pages with auto-deploy via GitHub Actions.

## What's pending or open

- **Node.js 20 deprecation warning** in the GitHub Actions workflow — needs updating the `actions/checkout`, `actions/configure-pages`, etc. to versions that support Node 24 by **June 2, 2026**. Easy fix when the new versions are tagged.
- **Pablo's feedback loop** — sent update on May 1, 2026 about the original 9-icon version. v1.1.0 with the 25-icon library may better match his "self-serve for non-editors" vision.
- **Possible future work**: more anchor points in `DEFAULT_LAYOUTS` if the curated set needs more/fewer icons; user presets / sharing via URL hash; explicit "preset library" feature; per-icon offset/easing overrides
- **Known limits**: MP4 won't preserve transparency (H.264); WebCodecs MP4 only works in Chromium browsers + Safari 16.4+; FFmpeg.wasm fallback was removed (was bloat)

## Conventions Claude should follow when iterating

- **One change at a time, small diffs.** Use `str_replace` not full-file rewrites.
- **Always run a parse check** on the JS blocks after edits: `node -e "..."` to catch syntax errors before deploying.
- **Always copy the file to `/mnt/user-data/outputs/` and call `present_files`** at the end of each turn.
- **Don't ask permission for tiny obvious things** but **do ask** when:
  - The user's spec is ambiguous (e.g. "AUTO/CUSTOM" semantics needed clarification)
  - There are competing valid interpretations
  - A change has cross-cutting consequences
- **Prefer fixing the root cause** over patching symptoms (e.g. the "A doesn't show" bug was solved by changing input type, not by hacking display logic)
- **Trust user feedback over your own assumptions** about how things look — they're the designer
- **Save a fallback before risky changes.** When the user okays a destructive or hard-to-revert change, copy the current working file as `product-logos-slide-tool-vX.Y.Z-fallback.html` first.
- The `userMemories` block has long-form context about Nebojsa, his work at Automattic, and prior Claude sessions. Skim it for tone but don't over-rely on it.
- For GitHub workflow questions: this user is new to git/GitHub. Walk through clicks step-by-step with screenshots. Don't assume command-line familiarity.

## Useful one-liners

Deploy current state:
```bash
cp /home/claude/icon-tool/product-logos-slide-tool.html /mnt/user-data/outputs/product-logos-slide-tool.html
```

Parse-check all script blocks:
```bash
node -e "
const fs = require('fs');
const html = fs.readFileSync('/home/claude/icon-tool/product-logos-slide-tool.html', 'utf8');
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

Find a function quickly:
```bash
grep -n "function exportMP4\|setPanelState\|autoFillNewSlots" /home/claude/icon-tool/product-logos-slide-tool.html
```
