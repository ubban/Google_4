# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. There is no build step, package manager, test suite, or server.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (`<style>`, ~lines 10–359), and JavaScript (`<script>`, ~lines 422–1667). |
| `README.md` | Placeholder. Contains only the title. |

Run it by opening `index.html` in a browser. Nothing else is required.

## Architecture of `index.html`

**Markup (lines ~361–420).** A `#container` holds:
- `<canvas id="gameCanvas">` — the only rendered game surface (all drawing is Canvas 2D).
- HUD: score, best, multiplier, weapon name/level, hearts, bombs.
- Boss health bar (`#bossBarWrap`), combo pop-up, stage banner.
- Top-right icon buttons: mute, reduced-motion toggle, pause.
- Touch controls (`#touchControls`): a left-side joystick zone plus FIRE and BOMB buttons.
- `#overlay` / `#overlayCard`: the title / game-over card with difficulty buttons and Start.
- `#srLive`: an `aria-live` region used by `announce()` for screen-reader updates.

**Script (one IIFE, `'use strict'`).** Everything lives inside `(function () { ... })()`, so nothing leaks to `window`. Key sections, in file order:

1. **DOM refs and constants** — `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (6 sky palettes, cycled per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), `BOSSES` (6 bosses, each with `hpMul`).
2. **Adaptive quality** — `trackPerf()` averages the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low) to keep frame rate up.
3. **Settings and persistence** — best score, mute, difficulty, and reduced motion are read from and written to `localStorage` under the `skyfighter.*` keys. Reduced motion defaults to the `prefers-reduced-motion` media query until the user sets it.
4. **Audio** — `ensureAudio()` lazily creates the Web Audio context (browsers require a user gesture first). `beep()` and `noiseBurst()` synthesize every sound effect; there are no audio files.
5. **Layout and input** — `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) logical playfield to the window. Keyboard, mouse, and touch handlers write into the `keys` map and `firing`/`joystick` state.
6. **Game state and entities** — `resetGame()` builds `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, and `boss`. Spawning is in `spawnEnemy()`, `spawnBoss()`, and `maybeSpawnPowerUp()`.
7. **Simulation** — `update(dt)` advances everything per frame. `fireWeapon()` implements two weapons (`vulcan` and `laser`), each with three levels. `applyPowerUp()` handles `life`, `bomb`, and `weapon` pickups. `useBomb()` clears the screen.
8. **Bosses** — `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss()` cycles through them; `killBoss()` advances the stage.
9. **Life and flow** — `loseLife()` (with invulnerability frames), `endGame()` (updates best score and shows the stats card), `togglePause()`.
10. **Rendering** — `drawPlane()`, `drawBossShip()`, `drawClouds()`, and `draw()` render one frame.
11. **Main loop** — `loop(now)` is driven by `requestAnimationFrame`, computes `dt`, calls `update` and `draw`.

## Controls (keep the in-game hint in sync)

The hint text lives in `#controlsHint` in the markup. If you change a binding, update it.

- **Desktop:** mouse moves the ship and click fires; arrows or WASD move; Space fires; X or Shift bombs; P or Esc pauses.
- **Touch:** drag on the left to fly, tap FIRE, tap BOMB.

## Conventions

- **Single file on purpose.** Keep the game in `index.html`, with inline CSS and JS. Do not add a bundler, framework, or external CDN dependency unless the user asks for it.
- **No external network calls.** The game runs fully offline. Persistence is `localStorage` only.
- **Use the existing patterns.** Entities are plain objects in arrays; collision uses `rectsOverlap()`; visual feedback uses `spawnExplosion()`, `screenShake()`, and `flashHit()`. Respect `reducedMotion` when adding new screen shake, flashes, or pulses.
- **Keep sound synthesized.** Add new effects with `beep()` or `noiseBurst()` rather than adding audio assets.
- **Balance lives in tables.** Difficulty tuning belongs in `DIFFICULTIES`, enemy stats in `ENEMY_TYPES`, boss stats in `BOSSES`, and boss attacks in `BOSS_PATTERNS`. Prefer editing these tables over hard-coding numbers in logic.
- **Accessibility.** Announce important state changes through `announce()`, keep buttons labeled with `title`/text, and keep the reduced-motion toggle working.
- **Mobile.** Preserve the touch layout (`#touchControls`, joystick and FIRE/BOMB buttons). After markup or CSS changes, check touch controls still appear and respond, and that the overlay fits a phone-sized viewport.

## Working on this repo

- There is no test runner, linter, or build. Verify changes by loading `index.html` in a browser and playing through: start, a stage transition, a boss fight, a game over, and a best-score save on reload.
- Check the browser console for errors after edits. The script is a single IIFE, so one syntax error stops the entire game.
- Keep changes small and focused. Because everything is in one file, avoid unrelated reformatting, which makes diffs hard to review.
- Git: develop on the branch you were given; do not push to `main` without permission.

## Known gaps

- `README.md` is a placeholder and does not describe the game. Update it if the project grows beyond this single page.
- There are no automated tests, so balance and behavior changes depend on manual play-testing.
