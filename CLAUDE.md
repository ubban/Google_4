# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines: inline CSS, markup, and a single inline `<script>`). `README.md` is a placeholder.

There is no build step, package manager, bundler, test suite, or linter. To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

```
index.html   # The whole game: <style>, HUD/overlay markup, and <script>
README.md    # Placeholder
CLAUDE.md    # This file
```

## Structure of `index.html`

- **`<style>` (about lines 10–360):** layout, HUD, overlay card, touch controls, boss bar, and banners.
- **`<body>` (about lines 361–421):** `#container` holding the `#gameCanvas` canvas, the `#hud` (score, best, multiplier, weapon, hearts, bombs), boss bar, combo/stage popups, `#topBar` (mute, reduced-motion, pause buttons), touch controls (`#touchControls`, `#joystickBase`, FIRE/BOMB buttons), and the start/pause `#overlay` with difficulty buttons (`.diffBtn`) and the `#startBtn`.
- **`<script>` (about lines 422–1669):** an IIFE-style block with no modules. Major sections, in order:
  1. **DOM refs and constants**: element lookups, `STORAGE_*` localStorage keys (`skyfighter.*`).
  2. **Config tables**: `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs).
  3. **Persistence and settings**: best score, mute, difficulty, and reduced-motion preference, stored in `localStorage`.
  4. **Audio**: lazily created `AudioContext` (`ensureAudio`), `beep` and `noiseBurst` primitives, and the `sfx` table.
  5. **HUD and effects**: hearts/bombs rendering, hit/bomb flashes, combo pop, stage banner, screen shake.
  6. **Responsive scaling**: `BASE_W = 480`, `BASE_H = 720`, scaled to the window (capped at 1.6x) in `resize()`.
  7. **Input**: keyboard (`keys`), virtual joystick and fire/bomb buttons for touch, and mouse steering/firing on the canvas.
  8. **Game state**: `resetGame()` initializes the module-level `let` variables (player, bullets, enemies, enemyBullets, particles, powerUps, boss, score, lives, bombs, stage, chain/multiplier).
  9. **Entities and rendering**: `drawPlane`, `drawBossShip`, enemy spawning, power-ups, explosions, scoring/chains (`addScore`, `breakChain`).
  10. **Weapons and bosses**: `fireWeapon` (vulcan/laser with levels), `useBomb`, `BOSS_PATTERNS` (per-boss bullet patterns such as radial, aimed spread, columns, spiral, cross, homing, and double ring), `updateBoss`, `killBoss`.
  11. **Flow**: `loseLife`, `endGame`, `togglePause`.
  12. **Main loop**: `update(dt)` → `draw()` → `loop(now)` driven by `requestAnimationFrame`. `trackPerf` adjusts a `quality` level (0 full, 1 medium, 2 low) when frame times are poor. Changing quality affects particle counts and glow.

## Conventions

- **Keep it in one file** unless the user asks for a split. The project has no build tooling, so new JS or CSS should go inline in `index.html`.
- **Vanilla JS and Canvas 2D only.** Do not add frameworks or external CDN dependencies without asking.
- **Preserve the global-state style.** Game state uses top-level `let`s inside the script block and is reset in `resetGame()`. New per-run state belongs there.
- **Tune gameplay through the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `THEMES`) rather than hard-coding magic numbers in update logic.
- **Respect accessibility settings.** Screen shake, flashes, and pulses must honor `reducedMotion` (`#motionBtn`, `prefers-reduced-motion`). Announcements go through `announce()` into the `#srLive` live region.
- **Support touch and keyboard/mouse.** Any new control needs both input paths, and the touch layout must work on phone-sized screens.
- **Persist user preferences** only through the existing `skyfighter.*` localStorage keys, wrapped in the same guarded style used now.

## Working on the game

- Open `index.html` directly in a browser to test. Verify changes by playing: start on each difficulty, check stage transitions (every `KILLS_PER_STAGE` kills), boss fights, bombs, pause/resume, mute, and reduced-motion toggling.
- Test on a narrow viewport or with touch emulation as well as desktop, since the layout and controls change with input type.
- Keep changes small and localized. Avoid reformatting unrelated sections of the 1,600-line script.

## Git

- Develop on the branch the session specifies (currently `claude/gifted-tesla-2prmd5`). Do not push to `main` without explicit permission.
- Write clear, descriptive commit messages.
- Do not open a pull request unless the user asks for one.
