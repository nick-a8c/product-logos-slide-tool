# Changelog

All notable changes to this project will be documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.1.0] — 2026-05-01

### Added
- Default library expanded from 9 to **25 Automattic brand icons** (Tumblr, P2, VideoPress, Atavist, Clay Mesh, WPScan, Gravatar, Simplenote, Newspack, WordPress.com, Beeper, Day One, Crowdsignal, Sensei, Parse.ly, Akismet, Jetpack, Woo, WordPress VIP, Cloudup, MailPoet, Pocket Casts, WP Cloud, Pressable, Longreads)
- **Curated layout system** (`DEFAULT_LAYOUTS`): every icon count from 2 to 25 has a hand-arranged set of icons. Anchor counts (2, 5, 9, 15, 25) follow specific aesthetic groupings; in-between counts inherit from the lower anchor and add icons in numeric order.
- **Auto-fill on count grow**: when raising the icon count slider, empty trailing slots are automatically populated from the layout for that count (skipping icons already manually placed in earlier slots). Manual placements are always preserved.
- Stable icon IDs (`'01'` through `'25'`) for default library icons; user uploads continue to receive random IDs.

### Changed
- **Default icon count**: 9 → 15 on first load. Matches the new 15-icon curated layout.
- **Default state values** (spacing 18, scale 75) now match the n=15 auto-mode anchor for visual coherence.

### Fixed
- (carried from v1.0.1) Icon assignments are no longer lost when lowering then raising the icon count slider. Trailing slots preserve their icons.
- (carried from v1.0.1) Library "in-use" indicator now only counts visible slots.

## [1.0.1] — 2026-05-01

### Fixed
- Icon assignments are no longer lost when lowering then raising the icon count slider. Trailing slots now preserve their icons.

## [1.0.0] — 2026-05-01

Initial public release.

### Added
- Six aspect ratios (16:9, 21:9, 4:3, 9:16, 4:5, 1:1) with per-ratio default resolutions
- Per-control Auto mode for Spacing, Scale, Speed, Stagger, Slide dist., Start scale, Easing — interpolates between anchors at counts 2/5/9/15/19/25
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
- Descriptive export filenames including resolution, quality, app version
- localStorage persistence with backward-compatible migrations
- Live deployment via GitHub Pages with auto-deploy GitHub Actions workflow
