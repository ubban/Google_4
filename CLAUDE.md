# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a static, dependency-free browser game called **Sky Fighter**: a
Raiden-style vertical shoot-'em-up. The entire game lives in one file,
`index.html` (inline `<style>`, inline markup, and one inline `<script>`).
`README.md` is a placeholder with no setup instructions.

There is no build step, package manager, test suite, or linter configured.

## Running it

Open `index.html` directly in a browser, or serve the folder with any static
server (for example `python3 -m http.server` and visit `localhost:8000`).

## Repository layout

| Path         | Purpose                                                    |
|--------------|------------------------------------------------------------|
| `index.html` | The whole game: markup, CSS, and JavaScript (~1,670 lines) |
| `README.md`  | Placeholder README                                         |

## Code structure inside `index.html`

- **Markup (lines ~362–420):** `#container` holds the `<canvas id="gameCanvas">`,
  the HUD (`#hud`, `#bossBarWrap`, `#topBar`), touch controls
  (`#touchControls`), and the start/pause/game-over `#overlay`.
- **Script (lines ~422–1667):** one IIFE with `'use strict'`. Nothing is
  exported or attached to `window`, so keep new code inside the IIFE.
- **Config tables near the top of the script:**
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, shoot
    chance, lives, bombs, boss HP multiplier.
  - `THEMES`: background gradients cycled per stage.
  - `ENEMY_TYPES`: grunt, interceptor, tank.
  - `BOSSES`: six bosses, cycled by stage; each has colors and an `hpMul`.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `BASE_W` / `BASE_H` (480 × 720): logical canvas size, scaled by `resize()`.
- **Game logic:** `update(dt)` runs each frame. Helpers include `spawnEnemy`,
  `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`,
  `loseLife`, `endGame`, and `togglePause`.
- **Boss attacks:** `BOSS_PATTERNS` is an array of bullet-pattern functions
  (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, …).
- **Rendering:** `draw()` paints each frame. `drawPlane`, `drawBossShip`, and
  `drawClouds` are the sprite helpers, all drawn with canvas primitives. No
  image assets are loaded.
- **Audio:** WebAudio synthesis only (`beep`, `noiseBurst`, `ensureAudio`).
  There are no audio files.
- **Loop:** `loop(now)` is a `requestAnimationFrame` loop. `dt` is clamped to
  50 ms. `trackPerf` adjusts a `quality` level (0–2) from the average frame time.
- **Persistence:** `localStorage` keys are prefixed `skyfighter.`:
  `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads are not wrapped in
  try/catch, so keep that in mind if storage is blocked.
- **Accessibility:** `announce()` writes to the `#srLive` ARIA live region.
  `prefers-reduced-motion` is respected and can be toggled in-game.

## Controls

- Move: arrow keys or WASD; touch uses the on-screen joystick.
- Fire: hold Space (or the fire button).
- Bomb: `X` or Shift (or the bomb button).
- Pause: `P` or Escape.
- Mute and difficulty are toggled from the UI.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it.
  If assets or modules are added later, update this file.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes,
  arrow functions for callbacks, and short section comments such as
  `// ---------- Input ----------`.
- Keep the game dependency-free. Do not add frameworks or CDN scripts without
  asking.
- Tune gameplay through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `BOSS_PATTERNS`) rather than hard-coding numbers in logic.
- Preserve the `prefers-reduced-motion` and screen-reader behavior when
  changing visual effects or HUD text.

## Verification

No automated tests exist. After changes, load `index.html` in a browser and
check at least: start, movement, firing, a bomb, a power-up, a stage transition
with a boss, pause/resume, game over, and the best-score update. Test both
keyboard and touch layouts if the change touches input or the HUD.

## Git

- Develop on the branch assigned to the session (currently
  `claude/gifted-tesla-qwkx6b`) and push with `git push -u origin <branch>`.
- Write clear, descriptive commit messages.
