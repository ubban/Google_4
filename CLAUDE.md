# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project containing **Sky Fighter**, a Raiden-style
vertical shoot-'em-up that runs in the browser on a `<canvas>`.

| File         | Purpose                                                        |
|--------------|----------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines). |
| `README.md`  | Minimal project readme.                                        |

There is no build step, no package manager, no bundler, and no test suite. Do not
introduce one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

- **`<head>`**: viewport meta (no user scaling), description, theme color, title.
- **`<style>`** (≈ lines 10–360): all styling, including the responsive HUD, overlay, and touch controls.
- **`<body>`** (≈ lines 363–420): the `<canvas id="gameCanvas">`, HUD elements, overlay with the Start button, touch controls (`#moveZone` joystick, `#fireBtn`, `#bombBtn`).
- **`<script>`** (≈ line 422 onward): one IIFE wrapping everything, with `'use strict'`. Sections, in order:
  1. DOM references and `STORAGE_*` keys
  2. Adaptive performance (`trackPerf` lowers `quality` when frame times are slow)
  3. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
  4. Settings: mute, reduced motion, difficulty (persisted to `localStorage`)
  5. Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers (Web Audio, created lazily on first user gesture)
  6. HUD and feedback effects: hearts, bombs, combo pop, stage banner, screen shake, flash
  7. Input: keyboard, touch joystick, fire/bomb buttons, mouse steering
  8. Game state: `resetGame`, `update(dt)`, spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), collisions, scoring (`addScore`, combo chain), bombs, weapons (`fireWeapon`, `applyPowerUp`)
  9. Bosses: `BOSS_PATTERNS` (bullet-pattern functions) and `updateBoss` / `killBoss`
  10. Lifecycle: `loseLife`, `endGame`, `togglePause`
  11. Rendering: `draw`, `drawPlane`, `drawBossShip`, `drawClouds`
  12. Main loop: `loop(now)` uses `requestAnimationFrame` and clamps `dt` to 0.05 s

## Game constants and where to change them

- **Difficulty** (`DIFFICULTIES`): spawn rate, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier. Selected via `.diffBtn` buttons.
- **Stages**: once `stageKills` reaches `KILLS_PER_STAGE` (18), a boss warning fires and `spawnBoss()` runs 1.2 s later. Bosses come from `BOSSES` (`hpMul` scales HP). `THEMES` sets the sky palette per stage.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaves), `tank` (4 HP, unlocked at stage 2). Each has size, HP, speed multiplier, colors, score value, and `minStage`.
- **Canvas**: logical resolution is fixed at `BASE_W × BASE_H` = 480 × 720. `resize()` scales the canvas by CSS to fit the window (max scale 1.6). Game coordinates always use the logical size, never CSS pixels.

## Controls

- **Keyboard**: Arrows / WASD move, Space fire, X or Shift bomb, P or Esc pause.
- **Mouse**: move to steer, click to fire.
- **Touch**: drag on the left zone to fly (joystick), tap FIRE, tap BOMB.

## Persistence

All persistence uses `localStorage`, under these keys:

| Key                        | Value                          |
|----------------------------|--------------------------------|
| `skyfighter.bestScore`     | Number, updated on game over   |
| `skyfighter.muted`         | `'1'` or `'0'`                 |
| `skyfighter.difficulty`    | `easy` / `normal` / `hard`     |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; defaults to the `prefers-reduced-motion` media query if unset |

When adding new settings, follow the same `STORAGE_` constant pattern. Keep the
storage keys namespaced with `skyfighter.`.

## Conventions

- **Keep it a single file.** Styles, markup, and script stay in `index.html` unless the user asks for a split. Do not add frameworks or external dependencies; the game uses only vanilla JS, Canvas 2D, and Web Audio.
- **Stay inside the IIFE.** New code should be added within the existing `(function () { ... })()` wrapper, not as globals.
- **Match the existing style**: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and section headers written as `// ---------- Name ----------`.
- **Frame-rate independence.** Movement and timers use `dt` (seconds) from the loop. Do not hard-code per-frame increments for new motion.
- **Respect reduced motion.** Effects such as screen shake and flashes should check the `reducedMotion` flag.
- **Accessibility.** Announce important state changes through the `srLive` live region (`announce()`), and keep the `title` attributes on icon buttons up to date.
- **Mobile first.** Verify changes at phone-sized viewports; the touch controls must stay usable and the page must not scroll horizontally.

## Verifying changes

There are no automated tests or linters. Verify manually:

1. Open `index.html` in a browser (or serve it) and check the console for errors.
2. Play through: start, move, fire, bomb, pause/resume, lose a life, game over, and restart.
3. Check each difficulty, mute and reduced-motion toggles, and that the best score persists after reload.
4. Check the layout at a narrow (phone) width and at desktop width.

## Git

- Development branch for this work: `claude/gifted-tesla-npq9xf`. Commit and push there.
- Use clear, descriptive commit messages in the imperative mood.
- Do not open a pull request unless explicitly asked.
