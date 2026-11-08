# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one self-contained file, `index.html` (inline `<style>` and one inline `<script>`, no external dependencies, no build step). `README.md` is a placeholder.

There is no package manager, test suite, linter, or CI configuration in the repo. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

- `index.html` — the entire game: markup, CSS, and JavaScript.
- `README.md` — minimal placeholder.
- `CLAUDE.md` — this file.

## Code structure of `index.html`

The script is one IIFE. Rough map of the sections (line numbers approximate):

- **HUD/DOM markup** (~lines 362–420): `#container`, `<canvas id="gameCanvas">`, `#hud` (score, multiplier, weapon, hearts, bombs), `#bossBarWrap`, `#topBar` (mute, reduced-motion, pause buttons), `#touchControls` (joystick, FIRE, BOMB), and `#overlay` (start/title/game-over card with difficulty buttons).
- **Persistence keys** (~463–466): `localStorage` keys prefixed `skyfighter.` for best score, mute, difficulty, and reduced motion. Every `localStorage` access should stay wrapped so the game still works when storage is unavailable.
- **Tuning tables** (~485–516): `DIFFICULTIES` (spawn rates, enemy speed, enemy fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses with colors and HP multipliers). Balance changes usually belong here.
- **Audio** (~543–680): a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` primitives and a named `sfx` table.
- **Rendering helpers**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()` for the frame.
- **Input** (~766–862): keyboard (arrows/WASD move, Space held to fire, X/Shift for bomb, P/Esc pause), a virtual joystick and FIRE/BOMB buttons for touch, and mouse steering plus click-to-fire on the canvas (`firing` flag).
- **Game state** (~865–912): module-level `let` variables (`score`, `lives`, `bombCount`, `stage`, `kills`, `chain`, `multiplier`, etc.). `resetGame()` restores them.
- **Simulation**: `update(dt)` (player movement and firing, enemy spawning, collisions, power-ups), `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `addScore`/`breakChain`.
- **Boss AI**: `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss`, `killBoss`.
- **Lifecycle**: `loseLife`, `endGame`, `togglePause`.
- **Loop**: `loop(now)` runs on `requestAnimationFrame`. `trackPerf` adjusts a `quality` level (full / medium / low) when frame times are poor.

## Conventions

- Keep the game as a single file unless the user asks for a restructure. New code goes inside the existing IIFE and follows its style (2-space indent, `const`/`let`, camelCase, `UPPER_SNAKE` for tuning tables).
- Put gameplay numbers in the tuning tables rather than hard-coding them in logic.
- Respect the reduced-motion setting when adding screen shake, flashes, or pulsing effects (`reducedMotion`, `screenShake`, `flashHit`).
- Keep touch and keyboard controls working together. Any new action needs both a keyboard binding and a touch/button path where applicable.
- Preserve mobile layout: the canvas is scaled from a `480×720` base (`BASE_W`, `BASE_H`) via `resize()`, and the HUD and touch controls must stay usable at phone widths.
- Respect the `<meta name="viewport">` and `touch-action: none` settings; don't add scrolling behavior to the page.

## Verifying changes

No automated tests exist. For gameplay or rendering changes:

1. Open `index.html` in a browser (for example via `python3 -m http.server` from the repo root, then visit the page) and check the console for errors.
2. Exercise the affected path: start a game, play through the changed behavior, pause/resume, lose a life, and reach game over.
3. For layout changes, check both a desktop viewport and a narrow phone-sized viewport.

## Git

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-yc3mcs`). Do not push to other branches without explicit permission.
- Write descriptive commit messages that explain the why when it isn't obvious.
- Do not open pull requests unless the user asks for one.
