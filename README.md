# A8c Product Logos Slide Tool

A single-file browser tool for building brand animations from a row of product icons and the **AUTOMATTIC** wordmark, then exporting them as **MP4, WebM, PNG, SVG, or a self-contained interactive HTML page**. No install, no build step, no server. One HTML file.

It ships with **two layout modes**:

- **Line** — a four-segment timeline: *logos in → logos out → AUTOMATTIC in → AUTOMATTIC out*, each segment with its own timing and easing.
- **Ring** — a continuous 3D orbit of icons with depth-of-field, optional intro, and live interactions (hover, ripple, drag-to-spin).

[**▶ Try it live**](https://nick-a8c.github.io/product-logos-slide-tool/) · [Download](product-logos-slide-tool.html)

——

<!-- Hero: drop a Ring-layout screenshot at examples/00-cover-ring.png and add it above this line for a Ring hero. -->
![Product Logos Slide Tool](examples/AAA.png)
![Product Logos Slide Tool](examples/BBB.png)

## What it does

Drop in a row of brand icons and configure them — count, scale, animation, background. Pick a layout, preview it, and export whatever your destination needs. New visitors open in **Line** mode; the tool then remembers your last layout and settings between visits.

- **Two layouts** — a four-segment **Line** timeline and a continuous 3D **Ring** orbit, switchable any time
- **AUTOMATTIC wordmark** — split into 10 letters that animate like the icons, using the wordmark's natural spacing
- **6 aspect ratios** — 16:9, 21:9, 4:3, 9:16, 4:5, 1:1
- **Gradient backgrounds** — solid, linear, or radial, applied in both layouts and every export
- **Auto modes** — per-control AUTO/CUSTOM in Line; an aspect-aware AUTO/CUSTOM for the whole Ring shape
- **Export formats** — MP4 (H.264), WebM (VP9 with alpha), PNG, still SVG, and an interactive self-contained HTML page (Ring)
- **Video quality presets** — Web (small) / High / Max (lossless), so shared files stay small
- **Frame-accurate stills** — pause the animation and scrub (mouse wheel or click-drag) to capture any exact frame as PNG or SVG
- **Fully offline** — every dependency is inlined. Works without network access.

## How to use

### Option 1: Open the live version

Visit [nick-a8c.github.io/product-logos-slide-tool](https://nick-a8c.github.io/product-logos-slide-tool/) in Chrome, Edge, or Safari 16.4+.

### Option 2: Run locally

Download `product-logos-slide-tool.html`, double-click to open in your browser.

### Option 3: Embed in a static site

Drop `product-logos-slide-tool.html` anywhere a static file can be served. No build, no bundler.

## Line layout — the four-segment timeline

The four segments live on a horizontal timeline under the stage. Click any segment pill to focus and replay it. Each segment has:

- A **duration dropdown** (0–10s) on the right
- An **animation panel** (Animate IN / Animate OUT / A8C Intro / A8C Outro) with Speed, Stagger, Slide distance, Start scale, and Easing
- A short **inter-segment pause** dropdown sitting between it and the next segment

Three play buttons live in the bottom bar:

- **PLAY SEGMENT** — plays the currently focused segment
- **LOOP SEGMENT** — loops the currently focused segment
- **PLAY ALL** — walks all four segments back-to-back, honoring the pause dropdowns

### Animation model

- **Intros** play *inner first → outer last*: middle icons land first, outer icons fan out with stagger.
- **Outros** by default mirror the intros (inner first → outer last); each outro panel has an **Order of movement** toggle that flips this to *outer first → inner last*.
- Slide distance is signed: negative compresses inward then flares out, positive starts spread then contracts in. Outermost icons travel ±100px max. Middle icon stays put.

## Ring layout — the 3D orbit

Switch **Stage Layout** to **RING** for a continuous orbit instead of a timeline. Icons travel a tilted ellipse with depth-of-field: icons toward the back scale down, blur, and fade; icons in front are sharp and full-size.

**Ring controls** — Radius, Tilt, Angle, Speed, Direction (forward/reverse), Icon Size, Depth Scale, Blur (DoF), and Fade Back, plus an optional staggered **Intro**.

### Ring AUTO / CUSTOM

The **Overall Control** at the top works in Ring mode too:

- **AUTO** sets the four shape sliders — **Radius, Tilt, Angle, Icon Size** — to values tuned for the current aspect ratio, and re-snaps them whenever you change aspect. Those four plus **Speed** appear grayed out (still draggable — grabbing one drops you into CUSTOM).
- **CUSTOM** hands you full manual control of every Ring setting.
- AUTO is **non-destructive**: switching to AUTO stashes your CUSTOM "playground." Switch back without touching anything and it's restored exactly. The first time you change *any* Ring setting in AUTO, the stash is committed and CUSTOM keeps the new values.

| Aspect | Radius | Tilt | Angle | Icon Size |
|---|---|---|---|---|
| 16:9 | 65 | 32 | -18 | 12 |
| 21:9 | 48 | 32 | -18 | 12 |
| 4:3 | 60 | 32 | -18 | 9 |
| 9:16 | 95 | 35 | -60 | 7 |
| 4:5 | 89 | 30 | -60 | 7 |
| 1:1 | 60 | 32 | -18 | 9 |

### Ring interactions

These run in the live preview and in the exported interactive HTML (never in video exports):

- **Hover** — grow or shrink the icon under the cursor, with an adjustable **spread** that eases the effect into neighboring icons
- **Ripple** — **Shift-click** the ring to send a wave travelling both ways around the orbit
- **Drag-to-spin** — grab the ring and fling it; **Grip / Resistance / Weight** shape how it grabs, decays, and settles back into auto-rotation

A plain click on any ring icon opens the picker to swap it — same as Line. (That's why ripple is Shift-click here.)

### Layout link

The chain button between **LINE** and **RING** mirrors the settings the two layouts share (icon count, zoom, background). It's **off by default** for new visitors — each layout keeps its own copy of those settings, so switching layouts doesn't disturb the other. Turn it on to keep them in sync.

## Backgrounds

A single background control feeds the live stage and every exporter. Choose **Solid**, **Linear**, or **Radial**:

- Solid uses one color.
- Linear/Radial blend between two colors, with angle (linear), center position (radial), and spread controls.

Backgrounds can also be turned off entirely — useful for WebM and SVG exports, which preserve transparency.

## Controls

| Control | Range | Scope | What it does |
|---|---|---|---|
| Icons | 2 – 25 (or 18 for 9:16 / 4:5 / 1:1) | Composition | How many icons in the row / ring |
| Spacing | 0 – 40 | Composition (Line) | Pixel gap between icons (doesn't affect the wordmark) |
| Scale | 40 – 200 | Composition (Line) | Size multiplier for each icon |
| Zoom | 20 – 100 | Composition (preview only) | Viewport zoom (not exported) |
| A8C scale | 10 – 80 | A8C section (Line) | Wordmark height — separate from icon scale |
| Speed | 0.2s – 3.0s | Per-segment (Line) | Per-item animation duration |
| Stagger | 0 – 100ms | Per-segment (Line) | Delay between paired items |
| Slide dist. | -50 – +50 | Per-segment (Line) | Travel distance, signed |
| Start scale | 0 – 100 | Per-segment (Line) | Starting size as % of final |
| Easing | dropdown | Per-segment (Line) | Cubic-bezier curve |
| Order of movement | toggle | Outros only (Line) | Inner-first (default) vs outer-first stagger |
| Radius / Tilt / Angle | sliders | Ring | Orbit geometry |
| Speed / Direction | slider + toggle | Ring | Rotation rate and direction |
| Icon Size | slider | Ring | Icon size on the orbit |
| Depth Scale / Blur (DoF) / Fade Back | sliders | Ring | Depth-of-field falloff toward the back |
| Intro | toggle + sliders | Ring | Optional staggered fade-in |
| Hover / Ripple / Drag-spin | toggles + sliders | Ring | Live interactions |

In Line mode, hover any control's label to reveal its **AUTO / CUSTOM** toggle. In Ring mode, the **Overall Control** at the top drives AUTO / CUSTOM for the whole shape.

## Exports

The export panel splits into **Motion** (WebM, MP4, Interactive HTML / embed) and **Stills** (PNG, SVG). Every export uses **4× full-frame supersampling** (downsampled via a halving pyramid) and a **6× SVG bitmap raster** with a 6% transparent margin, so edges stay clean at output resolution.

### Video quality

A **Quality** dropdown controls MP4/WebM file size by trading off bitrate and inter-frame compression:

| Preset | Use it for |
|---|---|
| **Web (small)** | Sharing — Slack, P2, social. Smallest files. |
| **High** *(default)* | Great quality, roughly 20× smaller than lossless. |
| **Max (lossless)** | Archival — every frame a keyframe, very large files. |

### Motion

**Line** — pick which segment(s) render via the `1 / 2 / 3 / 4 / ALL` toggle:

| Range | What you get |
|---|---|
| `1` | Just Logos Intro |
| `2` | Just Logos Outro |
| `3` | Just A8C Intro |
| `4` | Just A8C Outro |
| `ALL` | All four segments back-to-back with your configured pauses between them |

**Ring** — video covers exactly one revolution, so it loops seamlessly (plus the intro if enabled). **Export Interactive HTML** produces a self-contained page with the orbit and all interactions baked in — host it anywhere, no dependencies. (Video locks out at the highest spin speeds, where a single revolution is too short for smooth playback; stills stay available.)

### Stills (PNG + SVG)

PNG and SVG capture a single frame. To pick the exact one, hit **Pause Animation**, then **scroll the mouse wheel or click-drag across the stage** to scrub (Ring → orbit angle, Line → segment time), and export.

- **PNG** — rasterized current frame.
- **SVG** — vector current frame. In Ring, the depth-of-field blur is baked in as native SVG filters, so the still matches the on-screen look while staying scalable.

## Browser support

| Format | Chrome / Edge | Safari 16.4+ | Firefox |
|---|---|---|---|
| MP4 | ✓ | ✓ | ✗ |
| WebM | ✓ | ✓ | ✓ |
| PNG | ✓ | ✓ | ✓ |
| SVG | ✓ | ✓ | ✓ |
| Interactive HTML | ✓ | ✓ | ✓ |

MP4 export uses the browser-native [WebCodecs](https://developer.mozilla.org/en-US/docs/Web/API/WebCodecs_API) `VideoEncoder`, which Firefox doesn't yet support. Use WebM there. MP4 (H.264) does not preserve transparency — use WebM or SVG if you need an alpha channel.

## Filename format

```
product-logos-slide-tool_1920x1080_v2.mp4
product-logos-slide-tool_1080x1920_v2.png
product-logos-slide-tool_preset_1920x1080_v2.json
```

Format: `product-logos-slide-tool_<resolution>_<app-version>.<ext>`

## Architecture

Single HTML file (~385 KB), three inlined script blocks:
1. `gifenc` (~9 KB) — GIF encoder, retained from earlier versions (GIF export has since been removed)
2. `mp4-muxer` (~73 KB) — MP4 container muxer
3. App code — UI, animation, both layout engines, sequencer, export pipeline

State persists in `localStorage`. No server, no analytics, no telemetry.

See `HANDOFF.md` for full architecture notes.

## Development

```bash
git clone https://github.com/nick-a8c/product-logos-slide-tool.git
cd product-logos-slide-tool
# Open index.html in a browser. That's it. No build step.
```

For a tighter dev loop, serve with any static server:

```bash
npx serve .
# or
python3 -m http.server 8000
```

`index.html` and `product-logos-slide-tool.html` are kept identical — edit one, copy to the other.

## Contributing

PRs welcome. Keep it single-file. If you need a build step, propose it in an issue first.

## License

[MIT](LICENSE) — use it however you like.

## Credits

- [`gifenc`](https://github.com/mattdesl/gifenc) by Matt DesLauriers
- [`mp4-muxer`](https://github.com/Vanilagy/mp4-muxer) by Vanilagy
- Built collaboratively with Claude (Anthropic) for [Automattic's](https://automattic.com/) Radical Speed Month, 2026.
