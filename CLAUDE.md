# CLAUDE.md

Guidance for working in this repository.

## What this repo is

A small, self-contained browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667). No build step, no bundler, no dependencies, no external network requests.
- `README.md` — placeholder title only.

There is no package.json, test suite, linter, or CI configuration. Verify changes by opening `index.html` in a browser (or serving the directory with any static server) and playing through the affected behavior.

## Layout of `index.html`

- **Markup** (~lines 361–420): `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`, `#bossBarWrap`, `#comboPop`, `#stageBanner`), the top bar buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#touchControls` with a joystick, FIRE, and BOMB), the start/pause/game-over `#overlay`, and a screen-reader live region `#srLive`.
- **Script**: a single IIFE wrapping everything. Top-level constants come first, then state, then functions, then input handlers, then the game loop (`loop`, started with `requestAnimationFrame`).

Key sections inside the script, roughly in order:

| Section | What it holds |
|---|---|
| Element refs and `STORAGE_*` keys | DOM lookups; `localStorage` keys prefixed `skyfighter.` |
| `trackPerf` / `quality` | Adaptive render quality (0 full, 1 medium, 2 low) based on frame time |
| `DIFFICULTIES` | Per-difficulty spawn rates, enemy speed, shot chance, lives, bombs, boss HP multiplier |
| `THEMES`, `ENEMY_TYPES`, `BOSSES` | Stage backdrops (cycled per stage), enemy stats, boss designs (cycled per stage) |
| Audio (`beep`, `noiseBurst`, `sfx`) | All sound is synthesized with the Web Audio API; no audio files |
| Input | Keyboard, mouse aim/fire on the canvas, touch joystick and buttons |
| `resize` | Scales the 480×720 logical canvas (`BASE_W`/`BASE_H`) to the viewport, capped at 1.6× |
| `resetGame`, `update`, `draw` | Core loop: state reset, simulation step, rendering |
| `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp` | Entity spawning and drops |
| `fireWeapon`, `BOSS_PATTERNS`, `updateBoss` | Player weapons (vulcan/laser, levels 1–3) and per-boss bullet patterns |
| `killBoss`, `loseLife`, `endGame`, `togglePause` | Progression and session flow |

## Gameplay facts (keep docs and code in agreement)

- Stage advances after `KILLS_PER_STAGE` (18) kills, which triggers a boss fight. Defeating the boss increments `stage`.
- Stages cycle through 6 themes and 6 bosses via modulo.
- Enemies: `grunt` (stage 1+), `interceptor` (stage 1+, weaving), `tank` (stage 2+, 4 HP).
- Power-ups: `life`, `bomb`, and `weapon` (raises weapon level to 3, then toggles vulcan ↔ laser).
- Persisted per browser: best score, mute, difficulty, reduced-motion preference. Best score is saved only when a game ends with a new high.

## Controls

- Move: arrow keys / WASD (keyboard), mouse movement on the canvas (aim), or touch joystick.
- Fire: Space, or hold left mouse / the FIRE button.
- Bomb: X or Shift, or the BOMB button. Clears enemy bullets.
- Pause: P or Escape, or the pause button.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a build tool or framework without being asked.
- **Match existing style.** Two-space indentation inside the script, `const`/`let`, plain functions, short inline comments only where the logic is not obvious. Constants are UPPER_SNAKE_CASE; DOM refs are camelCase with an `El`/`Btn` suffix where they describe an element.
- **Keep the frame loop cheap.** Per-frame code runs at 60 fps on mobile too. Use the existing `quality` level when adding particles or gradients, and reuse `spawnExplosion` rather than creating new effect paths.
- **Respect reduced motion.** Screen shake and flashes should go through `screenShake` / `flashHit` / `flashBomb`, which honor `reducedMotion`.
- **Accessibility.** Announce significant state changes through `announce()` so they reach `#srLive`. Keep buttons labeled and keyboard-reachable.
- **Touch and desktop.** Changes to input must keep both the touch controls and the keyboard/mouse paths working. The viewport meta disables user zoom; keep the layout usable at phone width.
- **Persistence keys.** Changing a `skyfighter.*` storage key resets that setting for existing players. Only do so deliberately.

## Testing and verification

- No automated tests exist. For gameplay changes, play a session in a browser: start, fire, collect power-ups, trigger a bomb, reach a boss, pause/resume, and die to the game-over screen.
- Check difficulty switching, mute, and reduced-motion toggles still persist across reloads.
- Check the layout at a phone-sized viewport and with touch emulation.
- Open the browser console and confirm there are no errors during play.

## Git

- Default development branch for this workspace: `claude/gifted-tesla-rlovji`.
- Write commit messages in the imperative mood and describe the gameplay or code change, not the file.
