# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a single-page, Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,700 lines). |
| `README.md` | Placeholder; contains only the title. |
| `CLAUDE.md` | This file. |

There is no build step, package manager, bundler, framework, or test suite. Do not add one unless asked.

## Running and testing

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`) and visit it.
- There are no automated tests or linters configured. Verify changes by loading the game in a browser and playing through the affected behavior. For headless checks, Playwright is pre-installed in the cloud environment (see the environment notes; do not reinstall it).
- Check the browser console for errors after any change to the script.

## Architecture of `index.html`

The game lives in a single IIFE (`(function () { 'use strict'; ... })()`) inside the `<script>` tag. Key sections, in file order:

1. **DOM references** — `const` lookups for canvas, HUD, overlay, and buttons.
2. **Persistence** — `localStorage` keys, all prefixed `skyfighter.`:
   - `skyfighter.bestScore`
   - `skyfighter.muted`
   - `skyfighter.difficulty`
   - `skyfighter.reducedMotion`
   Every read/write should tolerate missing or blocked storage gracefully.
3. **Adaptive performance** — `trackPerf()` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to keep the frame rate up.
4. **Tuning constants**
   - `DIFFICULTIES` — per-difficulty `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`. Tune gameplay here rather than hard-coding numbers in logic.
   - `KILLS_PER_STAGE` (18) — kills needed before a stage boss warning.
   - `THEMES` — sky palettes that cycle per stage.
5. **Input** — a global `keys` map keyed by `KeyboardEvent.code`, plus touch joystick and mouse handling.
   - Move: Arrows / WASD, mouse, or touch drag on the left.
   - Fire: Space (held) or mouse click, or the on-screen FIRE button.
   - Bomb: X or Shift, or the BOMB button.
   - Pause: P or Esc.
6. **Audio** — a lazily created `AudioContext` (`ensureAudio()`) with synthesized `beep()` and `noiseBurst()` effects. Audio must only start after a user gesture.
7. **Game state and logic** — `resetGame()`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons (`fireWeapon`, weapon levels 1–3), bombs (`useBomb`), kill-chain scoring (`addScore`, `breakChain`), boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, etc.), and `loseLife()` / `endGame()`.
8. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`, screen shake, hit flash, vignette).
9. **Main loop** — `loop(now)` drives `update` and `draw` via `requestAnimationFrame` with a clamped delta time.

Conventions observed in the code:
- Game time is advanced by `dt`; keep new movement and timers `dt`-based, not frame-based.
- Visual effects should respect the `reducedMotion` setting and the `quality` level.
- Announce important state changes to screen readers through the existing `announce()` / `srLive` live region.

## Conventions for changes

- **Keep it a single file.** Add new CSS to the `<style>` block and new JS inside the existing IIFE unless the user asks for a split. Do not introduce external dependencies or CDN links without asking.
- **No build tooling.** Code must run as-is in modern evergreen browsers. Plain ES2017+ syntax is fine; avoid features that need transpilation.
- **Match the existing style:** 2-space indentation, single quotes in JS, `const`/`let` (no `var`), and short descriptive function names.
- **Mobile matters.** The game supports touch controls and safe-area insets. Check layout at phone width after any HUD or overlay change.
- **Keep the controls hint accurate.** If you change a key binding, update the `#controlsHint` text in the overlay as well.
- **Don't break persistence keys.** Renaming a `skyfighter.*` storage key silently resets players' saved data; only do it deliberately and mention it in the summary.

## Git workflow

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-lavu3u`). Do not push to other branches without explicit permission.
- Commit only when asked or when the session's workflow requires it. Write descriptive, imperative commit messages that explain the why when it isn't obvious.
- Do not create a pull request unless the user explicitly asks for one.
- Never commit secrets or local environment files.

## Things to avoid

- Don't add a framework, package manager, or build pipeline to a project that is intentionally a single static file.
- Don't refactor the whole file into modules or split it up without a request; it is organized as one IIFE on purpose.
- Don't change `DIFFICULTIES` or scoring constants as a side effect of an unrelated fix. Gameplay balance changes should be called out explicitly.
- Don't remove the `touch-action`, `user-select`, or viewport meta settings; they are what keep the canvas from scrolling or selecting on mobile.
