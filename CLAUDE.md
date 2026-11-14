# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a 2D `<canvas>`.

- `index.html` — the entire game: markup, CSS, and JavaScript in one self-contained file (~1,670 lines).
- `README.md` — minimal placeholder (title only).

There is no build system, package manager, bundler, test suite, or linter configured. Do not introduce one unless asked.

## Running the game

No build step is required. Either:

- open `index.html` directly in a browser, or
- serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.

## Architecture of `index.html`

The file is organized top to bottom as:

1. **`<head>`** — viewport meta (locked, `user-scalable=no`), an inline SVG favicon, and an inline `<style>` block. Colors and layout are defined inline; there are no external stylesheets or fonts.
2. **`<body>`** — the `#container`, the `<canvas id="gameCanvas">`, the HUD (`#hud`), overlay screens, touch controls (`#touchControls`, joystick + fire/bomb buttons), and a visually hidden `#srLive` region for screen-reader announcements.
3. **`<script>`** — a single `'use strict'` IIFE. Everything is local to it; nothing is exposed on `window`.

### Inside the script

- **Canvas sizing**: logical resolution is `BASE_W = 480` × `BASE_H = 720`. `resize()` scales the displayed canvas to fit the window (capped at 1.6×). Drawing and game logic always use the logical coordinates.
- **Config tables** (near the top of the script — change balance here, not in logic):
  - `DIFFICULTIES` — `easy` / `normal` / `hard`: spawn timing, enemy speed, enemy fire chance, starting `lives` and `bombs`, and `bossHpMul`.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES` — six sky palettes, one per stage cycle.
  - `ENEMY_TYPES` — `grunt`, `interceptor`, `tank`, with `minStage` gating.
  - `BOSSES` — six boss definitions (name, colors, `hpMul`), plus `BOSS_PATTERNS` for attack patterns.
- **Game loop**: `requestAnimationFrame(loop)` → `update(dt)` → `draw()`. `dt` is clamped to 0.05 s. Update and draw only run while `running && !paused`.
- **Update/draw split**: `update(dt)` mutates state (player, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`); `draw()` only reads it. Keep new logic on the correct side of that split.
- **Audio**: a tiny WebAudio synth (`beep`, `noiseBurst`, and the `sfx` object). Audio is created lazily via `ensureAudio()`, which must be triggered by a user gesture.
- **Adaptive performance**: `trackPerf(dt)` averages the last 45 frame times and steps a `quality` level (0 full → 2 low) up or down. Shadow blur and particle counts read `quality`.
- **Persistence**: `localStorage` keys, all prefixed `skyfighter.`:
  - `skyfighter.bestScore`
  - `skyfighter.muted`
  - `skyfighter.difficulty`
  - `skyfighter.reducedMotion` (falls back to the `prefers-reduced-motion` media query when unset)

  Every access should tolerate storage being unavailable (private mode, blocked storage).

### Controls

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys or WASD | Left-side joystick |
| Fire (hold) | Space | Fire button (hold) |
| Bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Mouse steering | Mouse move over canvas (ship follows the cursor) | — |
| Mouse fire | Hold mouse button over canvas | — |

Space's default is prevented so the page doesn't scroll.

## Conventions

- **Keep it dependency-free.** Plain HTML/CSS/JS, no frameworks, no CDN imports, no build tooling. Everything the game needs lives in `index.html`.
- **Keep the single-file structure** unless the user asks to split it. If you split it, update every reference above.
- **Coordinates and balance**: prefer editing the config tables over hard-coding numbers in update/draw logic.
- **Match the existing style**: 2-space indentation, `const`/`let`, arrow functions for callbacks, section comments like `// ---------- Input ----------`.
- **Don't break the touch path.** Touch controls use `{ passive: false }` with `preventDefault()` on the buttons. Test any input change with both keyboard and touch in mind.
- **Accessibility**: respect `reducedMotion`, and route user-facing status text through `announce()` so screen readers get it.
- **Name new localStorage keys** with the `skyfighter.` prefix.

## Verification

No automated tests exist. Before reporting a change as working:

- Syntax-check the inline script, e.g. extract the `<script>` body and run `node --check` on it.
- For gameplay changes, load the page in a browser and play through the affected path (start, fire, bomb, pause, game over, and a stage transition when relevant).

## Git

- Development branch for this work: `claude/gifted-tesla-rw8jxn`. Commit and push there; don't push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
