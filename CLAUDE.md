# CLAUDE.md

Guidance for working on this repository.

## What this is

Sky Fighter, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas. The whole game is one self-contained file, `index.html`: markup, CSS (`<style>`), and game logic (one inline `<script>` wrapped in an IIFE). `README.md` is a placeholder.

There is no `package.json`, build step, bundler, test suite, or linter. To run the game, open `index.html` in a browser, or serve the directory with any static server.

## Layout of `index.html`

- **Lines ~1–360:** `<head>` (meta tags, theme color, inline SVG favicon) and the `<style>` block.
- **Lines ~361–421:** DOM. HUD (`#hud`: score, best, multiplier, weapon, hearts, bombs), boss bar, combo and stage banners, top-bar buttons (mute, reduced motion, pause), touch controls (`#touchControls`), and the start/pause overlay (`#overlay`).
- **Lines ~422 onward:** the game script, in roughly this order:
  - Storage keys (`skyfighter.*` in `localStorage`) and persisted state: best score, mute, difficulty, reduced motion.
  - Config constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - Audio: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` table). No audio files.
  - Input: keyboard listeners, a touch joystick, and mouse/touch canvas handlers.
  - Simulation: `resetGame`, `spawnEnemy`, `spawnBoss`, `fireWeapon`, `update(dt)`, `applyPowerUp`, `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
  - Boss attack patterns: `BOSS_PATTERNS` and the `fire*` helpers.
  - Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`.
  - Main loop: `loop(now)`, which runs `update` then `draw` each frame.

## Game rules

- **Difficulty** (`DIFFICULTIES`): easy, normal (default), hard. Each sets spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier.
- **Enemies:** `grunt` (1 HP), `interceptor` (fast, weaves, 1 HP), `tank` (4 HP, appears from stage 2).
- **Stages:** `KILLS_PER_STAGE` (18) kills trigger a boss. Stage N uses `THEMES[(N-1) % 6]` for its palette and `BOSSES[(N-1) % 6]` for its boss. Boss HP is `(70 + N*35) * bossHpMul * design.hpMul`, using the difficulty's `bossHpMul` and the boss design's `hpMul`.
- **Power-ups:** `life`, `bomb`, and `weapon`. Weapon power-ups raise the weapon level up to 3, then toggle between `vulcan` and `laser` at level 1.
- **Scoring:** each kill extends a chain. The multiplier is `1 + floor(chain / 5)`, capped at x10. The chain breaks (`breakChain`) after 2.2s without a kill.

## Controls

- Desktop: mouse to steer and click to fire, or arrows/WASD to move and Space to fire. X or Shift drops a bomb. P or Esc pauses.
- Mobile: drag the left zone to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game in one file unless the user asks for a split. The code uses plain browser JavaScript with no modules, so do not add a bundler or `import` syntax.
- Match the existing style: 2-space indentation, `const`/`let`, camelCase names, and `UPPER_SNAKE` for config constants. Keep the IIFE wrapper.
- Game tuning lives in the config constants near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Change those before touching logic.
- Respect the reduced-motion setting. Screen shake, flashes, and pulses go through the existing `screenShake`, `flashHit`, and `flashBomb` helpers, which check `reducedMotion`.
- Keep the touch and keyboard paths working together. Any new control needs both.
- Persisted values use the `skyfighter.*` `localStorage` keys. Storage reads and writes are currently not wrapped in try/catch, so a blocked-storage environment can throw. Wrap any new storage access.

## Verifying changes

No automated checks exist. Verify by opening `index.html` in a browser and playing: start a game on each difficulty, check pause and resume, bombs, power-ups, a stage transition, a boss fight, and game over. Also check the mobile layout at phone width if the change touches the HUD or touch controls.
