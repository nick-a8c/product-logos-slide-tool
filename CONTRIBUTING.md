# Contributing

Thanks for considering a contribution. A few ground rules to keep this project simple.

## Principles

- **Single-file constraint.** `index.html` should remain the entire deliverable. Inline any new dependencies. If something can't be inlined, propose it in an issue first.
- **No build step.** Plain HTML, CSS, JS. No bundlers, no transpilers, no package.json. Yes, this means the file is large. That's the trade-off.
- **No external runtime calls.** The tool must work on `file://` with no network. Everything ships in the HTML.
- **Backward-compatible state.** If you add fields to `state`, also add a migration in `restore()` so existing users don't lose data.

## Workflow

1. Fork and clone
2. Open `index.html` in your browser
3. Edit, refresh, repeat. (Or use `npx serve .` if you want auto-reload via a different tool.)
4. Verify your script blocks parse cleanly (the project's `HANDOFF.md` has a one-liner for this)
5. Submit a PR with a clear description and ideally a before/after screenshot or screen recording

## Style

- 2-space indent
- Semicolons in JS
- Function-level comments where the *why* isn't obvious from the *what*
- Prefer fixing root causes over patching symptoms

## What we'd love

- More easing presets
- Additional aspect ratios (with reasoning)
- Better default icon library
- Performance improvements for large icon counts
- Accessibility improvements

## What we'd push back on

- Adding a build step
- Adding analytics, telemetry, or external API calls
- Splitting the single file into many files
- Heavyweight dependencies that bloat the inlined size
