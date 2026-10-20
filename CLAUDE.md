# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in an HTML5 `<canvas>`.

| File         | Purpose                                                  |
| ------------ | -------------------------------------------------------- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one file |
| `README.md`  | Repository title only                                    |

There is no build system, package manager, bundler, test suite, or linter. Nothing needs
to be installed. Open `index.html` in a browser to play.

## Running and verifying

- Run: open `index.html` directly, or serve the folder with any static server, for example
  `python3 -m http.server 8000` and then visit `http://localhost:8000/`.
- There are no automated tests. Verify changes by playing in a browser and checking the
  affected feature (movement, firing, bombs, bosses, pause, menus, mobile controls).
- Test both desktop (keyboard and mouse) and touch layouts when you change input or HUD code.

## Structure of `index.html`

- **`<style>`** (about lines 10–362): all CSS. HUD, overlay card, touch controls, and the
  `.reduced-motion` class live here.
- **`<body>`** (about lines 363–421): the `<canvas id="gameCanvas">`, HUD elements, overlay,
  touch controls, and hidden live region (`#srLive`) for screen readers.
- **`<script>`** (about lines 422–1668): one IIFE (`'use strict'`) that holds all game state.
  Its sections, in order:
  1. DOM references and `STORAGE_*` keys
  2. Adaptive performance (`trackPerf` lowers quality when frames run slow)
  3. Config tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`
  4. Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers (Web Audio, created lazily)
  5. HUD helpers: hearts, bombs, combo pops, stage banner, screen shake
  6. Canvas sizing (`BASE_W` 480 × `BASE_H` 720, scaled to fit the window)
  7. Input: keyboard, mouse, and touch joystick and buttons
  8. Game logic: `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`,
     `update`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`
  9. Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`
  10. Main loop: `loop(now)` calls `update` and `draw` through `requestAnimationFrame`.
      `dt` is clamped to 0.05 s.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split.
  Do not add a build step or dependencies without asking.
- **Fixed logical resolution.** Game logic and drawing use the 480 × 720 canvas coordinate
  space. Scaling is handled by `resize()`. Do not hard-code CSS pixel sizes for game objects.
- **Time-based updates.** Movement and timers use `dt` in seconds. Do not tie speed to frame count.
- **Config-driven tuning.** Balance values (spawn rates, enemy HP, boss HP multipliers, lives,
  bombs) belong in the config tables near the top of the script, not inline in logic.
- **Persistence** goes through `localStorage` with keys prefixed `skyfighter.`:
  `bestScore`, `muted`, `difficulty`, `reducedMotion`. Wrap new reads and writes so the game
  still works when storage is unavailable (private mode, blocked site data).
- **Accessibility.** Announce important state changes through `announce()` (the `#srLive`
  region). Respect the reduced-motion setting: check `reducedMotion` before adding screen
  shake, flashes, or heavy animation.
- **Audio** starts only after a user gesture. Call `ensureAudio()` from input handlers, not on load.
- **Match the existing style.** The file uses 2-space indentation, single quotes in JS,
  `const`/`let` (no `var`), arrow functions for callbacks, and short section comments
  like `// ---------- Input ----------`. Follow the same pattern for new code.

## Controls (for reference when testing)

- Mouse: move to steer, click to fire.
- Keyboard: Arrow keys or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Touch: drag on the left to fly, tap FIRE, tap BOMB.

## Making changes

- Read the relevant section of `index.html` before editing. The file is large, so use
  search (for example `grep -n "function update"`) rather than reading it all at once.
- Keep edits focused. Do not reformat unrelated code.
- After changing gameplay, check that the other difficulties, stages, and boss patterns
  still work, since they share the same config tables and logic.
- Commit with clear messages that describe the gameplay or UI change.
