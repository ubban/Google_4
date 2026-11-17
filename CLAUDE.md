# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its main deliverable is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, inline CSS, and inline JavaScript (~1,670 lines).
- `README.md` — minimal placeholder.

There is no build system, package manager, bundler, framework, or test suite. Do not add
one unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server
(e.g. `python3 -m http.server`). No install step is needed.

## Code layout inside `index.html`

The file is organized as one `<style>` block followed by one `<script>` block.

- **Persistence keys** (`STORAGE_*` constants near line 463): best score, mute,
  difficulty, and reduced-motion are stored in `localStorage` under the `skyfighter.*`
  namespace. Keep these keys stable; changing them resets players' saved settings.
- **Tuning tables**: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and
  `BOSS_PATTERNS` hold gameplay balance values. Prefer editing these over hard-coding
  numbers in update logic.
- **Canvas sizing**: the logical play area is `BASE_W` × `BASE_H` (480 × 720). `resize()`
  scales the canvas to fit the viewport while keeping that aspect ratio.
- **Input**: keyboard handlers are on `window`. Movement uses arrow keys or WASD; Space
  or the on-screen fire button fires; `X` or Shift triggers a smart bomb; `P` or Escape
  toggles pause. Touch devices get a virtual joystick and fire button.
- **Game loop**: `update(dt)` → `draw()`, driven by `requestAnimationFrame` in `loop(now)`.
  Boss logic lives in `updateBoss` and the drawing helpers (`drawPlane`, `drawBossShip`,
  `drawClouds`) sit beside `draw()`.
- **Audio**: a Web Audio `AudioContext` is created lazily by `ensureAudio()` on the first
  user gesture, since browsers block autoplay. Respect the mute flag.

## Conventions

- Keep the game self-contained in `index.html`. Do not split out files, add modules, or
  introduce external dependencies or CDN links without asking first.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default,
  and section banners written as `// ---------- Name ----------`.
- Preserve mobile behavior. The layout must work at phone width with no page scroll
  (the body is `overflow: hidden` by design), and touch controls must keep working.
- Respect the reduced-motion setting when adding new screen-shake, flash, or animation
  effects.

## Verifying changes

There are no automated tests. After editing gameplay or UI:

1. Load `index.html` in a browser and check the browser console for errors.
2. Play through at least one stage start, a pause/resume, a bomb use, and a game over.
3. For layout changes, check both a desktop viewport and a narrow phone-sized viewport.

## Git

- Development branch for this work: `claude/gifted-tesla-16far3`. Push there and nowhere else
  unless the user says otherwise.
- Commit messages in this repo use a short imperative summary line, followed by a body
  describing the change in bullet points where helpful.
