# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`. There is no build step, no package manager, no framework, and no test suite.

Files:

- `index.html` — the entire game: markup (`<body>`, ~line 361), CSS (`<style>`, ~line 10), and one inline `<script>` IIFE (~line 422 to end). All game logic lives in that IIFE.
- `README.md` — a stub. Not useful as documentation.

## Running and verifying

- Open `index.html` directly in a browser. No server is required.
- There are no automated tests or linters configured. Verify changes by:
  1. Syntax-checking the inline script: `sed -n '/<script>/,/<\/script>/p' index.html | sed '1d;$d' > /tmp/game.js && node --check /tmp/game.js` (use the scratchpad directory instead of `/tmp` when working inside a Claude session).
  2. Playing the affected path in a browser — start, fire, collect power-ups, use a bomb, reach a boss, lose a life, and game over — at desktop width and a phone-sized viewport (touch controls appear on touch devices).

## Code architecture

Everything is inside one IIFE with `'use strict'`. Top-level state is `let` variables inside that closure; nothing is exported or attached to `window`.

Sections, in source order (search for the `// ----------` banners and function names):

- **DOM references** — `document.getElementById` lookups at the top of the IIFE. Add new element refs here.
- **Persistence** — `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Every read must tolerate missing or invalid values.
- **Adaptive performance** — `trackPerf(dt)` samples the last 45 frame times and steps a `quality` level (0 full, 1 medium, 2 low) that the draw code uses to reduce effects.
- **Tuning tables** — `DIFFICULTIES` (spawn rate, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage), `BOSS_PATTERNS` (bullet patterns).
- **Audio** — `ensureAudio()` lazily creates the `AudioContext` (it must follow a user gesture). Sound effects are synthesized with `beep()`, `noiseBurst()`, and the `sfx` object. No audio files.
- **Input** — keyboard (`keys` map keyed by `e.code`), mouse on the canvas, and touch joystick plus FIRE/BOMB buttons.
- **Game state and flow** — `resetGame()`, `update(dt)`, `draw()`, `loop(now)`. `loop` clamps `dt` to 0.05 s, calls `update` then `draw` only when running and not paused, then re-queues itself with `requestAnimationFrame`.
- **Combat** — `fireWeapon()`, `useBomb()`, `addScore()` with a chain/multiplier system (`breakChain()` resets it), `loseLife()`, `endGame()`, `killBoss()`.
- **Power-ups** — `maybeSpawnPowerUp()` drops `life`, `bomb`, or `weapon` items; `applyPowerUp()` applies them. Weapon pickups raise the level 1→3, then switch between `vulcan` and `laser` and reset to level 1.
- **Stages** — after `KILLS_PER_STAGE` (18) kills, a boss is spawned (`spawnBoss()`). Defeating it (`killBoss()`) increments `stage`. Boss design and sky theme are both chosen by `(stage - 1) % length`, so they cycle after six stages.
- **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, and the HUD helpers (`renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, `flashBomb`). The HUD is plain DOM, not canvas.
- **Accessibility** — `announce()` writes to the `#srLive` live region. `reducedMotion` (user setting, defaulting to `prefers-reduced-motion`) reduces screen shake to 15% and skips the hit/bomb flash effects; it also sets the `reduced-motion` body class.

Canvas logical size is `BASE_W` × `BASE_H` = 480 × 720. `resize()` scales it to the window; keep game coordinates in that space.

## Conventions

- Keep the single-file structure. Do not split into modules or add a build step unless the user asks.
- Match the existing style: 2-space indent, single quotes in JS, `const` by default, small named functions, and `// ----------` banner comments between sections.
- Put new tuning values in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than as magic numbers in logic.
- Gameplay state resets belong in `resetGame()`. Anything that must survive a restart belongs in `localStorage` with a `skyfighter.` prefix.
- Keep the controls hint (`#controlsHint`, ~line 417) in sync when changing input bindings. Current bindings: arrows/WASD move, Space or mouse click fire, X/Shift bomb, P/Esc pause.
- Respect `reducedMotion` for any new screen-shake, flashing, or pulsing effect.
- Guard any new audio call behind `ensureAudio()` and `muted`.

## Gotchas

- `dt` is clamped to 0.05 s. Gameplay timers therefore slow down, rather than jump, when the tab is backgrounded.
- Pause and game-over both gate `update()`, so timers that should freeze must be inside `update()`, not in independent `setTimeout` calls.
- `ensureAudio()` must run from a user gesture; the keydown, touchstart, and start-button handlers already call it.
- Touch controls are shown only when `ontouchstart` or `maxTouchPoints` indicates a touch device, so test touch changes with touch emulation.
- The title is "Sky Fighter" in `<title>`, `<meta name="description">`, and the favicon; keep them consistent if the name changes.

## Git

- Development branch for this work: `claude/gifted-tesla-kodbi0`. Default branch: `main`.
- Commit with clear messages describing the gameplay or code change.
