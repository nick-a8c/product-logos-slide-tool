# Changelog

All notable changes to this project will be documented here. Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

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
