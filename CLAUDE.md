# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up:

- `index.html` — the entire game: markup, CSS (in a `<style>` block), and JavaScript (in one `<script>` block wrapped in an IIFE with `'use strict'`). Roughly 1,700 lines.
- `README.md` — a one-line title only.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install is needed.

## Architecture of `index.html`

- **Rendering:** a single `<canvas id="gameCanvas">` drawn with the 2D context. The logical play field is `BASE_W = 480` × `BASE_H = 720`; `resize()` scales it to the viewport.
- **Main loop:** `loop(now)` → `update(dt)` → `draw()`. Game state (score, lives, bombs, weapon, stage, bosses, enemies, bullets, power-ups) lives in closure-scoped variables; `resetGame()` initializes them.
- **Content tables:** difficulty settings (`DIFFICULTIES`), enemy definitions (`ENEMY_TYPES`, filtered by `minStage`), boss designs (`BOSSES`, cycled by `(stage - 1) % BOSSES.length`), and stage themes (`THEMES`). Balance changes usually belong in these tables rather than in the update logic.
- **Boss attacks:** named pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) driven by `updateBoss`.
- **Audio:** synthesized with the Web Audio API (`beep`, `noiseBurst`, and the `sfx` object). Audio is started lazily by `ensureAudio()` on the first user gesture.
- **Persistence:** `localStorage` with the keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Reads and writes are not wrapped in try/catch; keep this in mind if you change them, since storage can be unavailable in some browsers or private modes.
- **Input:** keyboard (arrows/WASD to move, Space to fire, X or Shift to bomb, P or Escape to pause), touch joystick and buttons (`#touchControls`), and optional device motion (`motionBtn`).
- **Accessibility:** an `aria-live` region (`srLive`) announces events through `announce()`. A reduced-motion mode (`body.reduced-motion`, honoring `prefers-reduced-motion` by default) turns off most transient animations: the low-health vignette, combo and stage banners, hit/bomb flashes, and the overlay entrance.

## Conventions

- Keep the game in the single `index.html` file unless the user asks for a split. Do not introduce external dependencies or CDN scripts.
- Keep the existing style: 2-space indentation, single quotes in JavaScript, `const`/`let`, and small named functions.
- Use `e.code` for keyboard checks, as the existing handlers do.
- Preserve the touch and mobile behavior: `touch-action: none`, `user-select: none`, and the safe-area insets in the HUD CSS. Test layout changes at phone width.
- When adding gameplay content, follow the pattern of the existing tables (add an entry to `ENEMY_TYPES`, `BOSSES`, or `THEMES` rather than hard-coding a special case).

## Verifying changes

There are no automated tests. For any change:

1. Open the game in a browser (or use the `run` skill / a headless Chromium) and confirm it starts, plays, pauses, and reaches game over without console errors.
2. Check the affected feature directly: for example, weapon power-ups, bomb behavior, boss phases, difficulty switching, or the mute and reduced-motion toggles.
3. If the change touches layout, check both a desktop viewport and a phone-sized viewport.

## Git

- Development branch for this session: `claude/gifted-tesla-e6rzik`. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Keep commits focused, with a short imperative summary line.
