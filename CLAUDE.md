# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser on a single HTML5 canvas. The entire game is one self-contained file, `index.html`, with inline CSS and one inline `<script>` block wrapped in an IIFE.

- `index.html` — the whole game (markup, styles, and logic)
- `README.md` — minimal project readme

There is no build step, no package manager, no dependencies, and no test suite. To run the game, open `index.html` in a browser.

## Working on the code

- Keep everything in `index.html` unless the user asks for a structural change. Do not introduce a bundler, framework, or external libraries without asking.
- Preserve the IIFE and `'use strict'` at the top of the script block. Game state lives in closure-scoped `let` variables, not on `window`.
- Match the existing style:
  - Section dividers use the form `// ---------- Name ----------`.
  - Tuning data lives in UPPER_CASE or named `const` tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `BOSS_PATTERNS`).
  - Use arrow functions for callbacks and `function` declarations for named game routines.
  - Use 2-space indentation and single quotes in JavaScript.
- Storage keys are namespaced with `skyfighter.` (for example `skyfighter.bestScore`). Reuse the existing `STORAGE_*` constants and wrap new storage access so the game still works if `localStorage` is unavailable.
- Use the existing `announce()` helper for screen-reader updates when adding state changes the player should hear about.

## Game architecture

Major parts of the script, roughly in file order:

1. **DOM references and constants** — element lookups and `STORAGE_*` keys.
2. **Adaptive quality** — `trackPerf()` samples frame time and steps `quality` (0 full, 1 medium, 2 low) to shed effects on slow devices.
3. **Tuning tables** — `DIFFICULTIES` (spawn rate, enemy speed and fire chance, lives, bombs, boss HP multiplier), `THEMES` (six stage backgrounds), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses with colors and HP multipliers). `KILLS_PER_STAGE` controls stage progression.
4. **Audio** — a Web Audio synthesizer (`beep`, `noiseBurst`, and named `sfx` entries). Audio is created lazily by `ensureAudio()`, which is called from user input because browsers require a gesture first.
5. **Input** — keyboard (arrows or WASD, Space to fire, X or Shift for bomb, P or Escape to pause), touch joystick with fire and bomb buttons, and mouse on the canvas.
6. **Responsive canvas** — `resize()` keeps a fixed 480×720 logical resolution and scales the canvas CSS size to fit the window.
7. **Game state and loop** — `resetGame()`, `update(dt)`, and `draw()`, driven by `requestAnimationFrame` through `loop()`. `dt` is clamped to 50 ms so a tab switch does not teleport objects.
8. **Combat** — player weapons (`vulcan` and `laser`) in `fireWeapon()`, enemy spawns, power-ups (`life`, `bomb`, `weapon`) in `applyPowerUp()`, combo multiplier handling (`addScore`, `breakChain`), and bombs in `useBomb()`.
9. **Bosses** — `spawnBoss()`, `updateBoss()`, and `killBoss()`. Each boss cycles through projectile patterns defined in `BOSS_PATTERNS`, built from helpers such as `fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and `fireHomingSeeded`.
10. **Lifecycle** — `loseLife()`, `endGame()`, and `togglePause()`.

Gameplay values such as speeds, sizes, and damage are inline numbers inside the update and draw routines. Prefer moving a value into a named table only when it is shared or clearly tunable.

## Accessibility and settings

The game has player-facing settings that persist to `localStorage`: difficulty, mute, and reduced motion. Reduced motion defaults to the `prefers-reduced-motion` media query when no saved value exists. When adding visual effects such as shake, flashes, or particles, respect the `reducedMotion` flag. Keep `aria-live` announcements and the HUD readable at phone widths.

## Verification

Because there are no automated tests, verify changes by opening `index.html` in a browser and playing through the affected path. For touch changes, check the layout at a phone-sized viewport. Read the diff carefully for:

- Stale references to removed DOM IDs
- Missing `dt` scaling on time-based movement
- Listeners attached more than once
- Anything that would break when `localStorage` throws

## Git

- Develop on the branch assigned for the session. Do not push to `main` directly.
- Write clear, present-tense commit messages that describe the change.
- Do not open a pull request unless the user asks for one.
