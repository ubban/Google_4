# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in an IIFE). There is no build step, no bundler, and no package manager.
- `README.md` — minimal project readme.

There are no tests, no linter config, and no dependencies to install.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout inside `index.html`

- **Markup / CSS** (roughly lines 1–420): HUD pills, hearts and bomb counters, overlay card, combo popups, stage banner, mobile touch controls, and `reduced-motion` styles.
- **Script** (roughly lines 422 onward, one IIFE):
  - Storage keys (`skyfighter.*`) and DOM lookups at the top.
  - Adaptive performance (`quality` 0–2) that lowers visual detail when frame times slip.
  - `DIFFICULTIES` (easy / normal / hard) — spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier.
  - `THEMES` — sky palettes cycled per stage.
  - Audio: a small WebAudio `sfx` set, with `beep` and `noiseBurst` helpers. Audio is initialised lazily via `ensureAudio()` on the first user gesture.
  - Input: keyboard, mouse-aimed fire, a virtual joystick and buttons for touch.
  - Game state and logic: `resetGame`, `update(dt)`, `spawnEnemy` / `spawnBoss`, power-ups, bombs (`useBomb`), weapons (`fireWeapon`), boss attack patterns (`fire*` helpers), `loseLife` and `endGame`.
  - Rendering: `draw()` with helpers such as `drawPlane`, `drawBossShip` and `drawClouds`, driven by a `requestAnimationFrame` loop (`loop(now)`).

## Conventions

- **Keep it one file.** Do not split the game into modules or add a build step unless the user asks for it.
- **Plain JavaScript, no dependencies.** Use the platform APIs already in use (canvas 2D, WebAudio, Pointer/Touch/Keyboard events). Do not add CDN libraries.
- **Stay inside the IIFE.** Game state lives in closure variables; do not leak globals.
- **Time-based updates.** Movement and timers use the `dt` passed to `update(dt)`. Preserve this when adding entities.
- **Persist only through the existing keys.** Best score, mute, difficulty and reduced-motion use `localStorage` under the `skyfighter.*` prefix. Storage access is currently unguarded, so when adding new reads or writes, wrap them in `try`/`catch` because storage can be blocked in some browsers.
- **Respect accessibility hooks.** Honour the `reduced-motion` class and the `srLive` announcements when adding effects or messages.
- **Mobile first.** Changes to layout or input must keep the touch controls, safe-area insets and the no-scroll viewport working.
- **Match the existing style.** Two-space indentation, single quotes in JS, `const`/`let`, and section comments such as `// ---------- Name ----------`.

## Making changes

- Read the surrounding section of `index.html` before editing; the file is large and most logic is in one script block.
- Balance changes usually touch the `DIFFICULTIES` table, `KILLS_PER_STAGE`, the enemy definitions near line 503, or boss patterns in `updateBoss`. Adjust them together and keep the three difficulty levels consistent.
- After editing, verify by opening the page in a browser and playing: start, move, fire, use a bomb, collect a power-up, reach a boss, pause/resume, and game over. Check both keyboard and touch layouts where relevant.
- Commit with a clear message describing the gameplay or UI change.

## Git

- Develop on the branch assigned for the session, and push with `git push -u origin <branch>`.
- Do not open pull requests unless the user explicitly asks.
