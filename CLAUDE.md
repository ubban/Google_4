# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single self-contained page.

Files:

- `index.html` — the entire game: inline `<style>`, markup for the HUD/overlays/touch controls, and one inline `<script>` that drives the game (roughly lines 1–1669).
- `README.md` — a one-line title. There is no build, package manifest, or test suite.

There is no `package.json`, bundler, framework, or external script/CDN dependency. Keep it that way unless the user asks otherwise.

## Running and testing

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- No automated tests or lint exist. Verify changes by loading the page and playing: check the canvas renders, controls work with keyboard and touch, and the HUD updates.
- Check the browser console for errors after any change to the script.

## Architecture of `index.html`

The inline script is wrapped in a single scope and organized roughly as follows:

- **DOM references** — `document.getElementById(...)` lookups near the top of the script. Add new element references there.
- **Persistence keys** — `STORAGE_*` constants (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`). Reads and writes to `localStorage` are not wrapped in try/catch; keep any new storage access defensive, since storage can be unavailable (private windows, blocked site data).
- **Tuning tables** — `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `KILLS_PER_STAGE`. Balance changes should go here rather than into the game loop.
- **Performance scaling** — `trackPerf` samples frame times and sets `quality` (0 full, 1 medium, 2 low) to reduce effects on slow devices.
- **Audio** — a lazily created `AudioContext` (`ensureAudio`) with helpers `beep`, `noiseBurst`, and a `sfx` table. Audio must only start after a user gesture; call `ensureAudio()` from input handlers.
- **Accessibility** — `announce()` writes to the `srLive` live region. A reduced-motion setting (honoring `prefers-reduced-motion` unless the user has chosen otherwise) controls screen shake, flash, and pulse effects (toggled by `motionBtn`).
- **Input** — keyboard (`keydown`/`keyup` on `window`, tracked in `keys`), mouse on the canvas, and touch via a joystick (`moveZone`) plus `fireBtn` and `bombBtn`. Touch handlers call `preventDefault` to avoid scrolling.
- **Rendering** — a single 2D canvas (`gameCanvas`) sized in `resize()`, which runs on window `resize` and `orientationchange`.

When editing, keep the existing style: plain ES2015+ JavaScript, no modules, no build step, and all code in the one file.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. If you do split, publish any extra assets with relative paths and no leading slash.
- Preserve mobile behavior: the page uses `viewport-fit=cover`, safe-area insets, `touch-action: none`, and a 96vw-limited HUD. Test at phone width after layout changes.
- Preserve the dark color scheme (`color-scheme: dark`) and the existing HUD look (translucent pill stats, gold values).
- Do not add external fonts, scripts, or network calls without user approval.
- Don't commit generated files or editor/OS junk.

## Git workflow

- Default working branch for this repo's Claude sessions: `claude/gifted-tesla-f9ktu3`. Push there with `git push -u origin <branch>`, retrying network failures with backoff.
- Do not open a pull request unless the user explicitly asks for one.
- Commit messages should describe the gameplay or code change in the imperative mood, with the attribution lines that the session's instructions require.
