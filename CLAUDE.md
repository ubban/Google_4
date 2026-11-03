# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up game. It is a single-file, dependency-free HTML5 canvas app:

- `index.html` — the entire game: markup, CSS (`<style>` at the top), and JavaScript (one IIFE in a `<script>` block near the end of the file).
- `README.md` — minimal placeholder.

There is no build step, package manager, test suite, or linter configured. Open `index.html` in a browser (or serve the directory with any static server) to run it.

## Architecture (index.html)

Everything lives inside a single `(function () { 'use strict'; ... })();` closure. Key sections, in file order:

1. **DOM references** — `getElementById` lookups for HUD, overlay, touch controls, and buttons. IDs in the HTML must stay in sync with these lookups.
2. **Persistence keys** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every access should tolerate storage being unavailable or blocked.
3. **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full / 1 medium / 2 low) up or down.
4. **Config tables** — `DIFFICULTIES` (spawn rates, enemy speed/fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes; cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, each with its own palette and HP multiplier).
5. **Audio** — `ensureAudio` lazily creates the `AudioContext` (must be called from a user gesture). `beep` and `noiseBurst` are the primitives; `sfx.*` wraps them.
6. **Input** — keyboard (`keys[e.code]`), touch joystick + fire/bomb buttons, and mouse/pointer tracking on the canvas. `KeyP`/`Escape` pause, `KeyX`/`Shift` bomb, `Space` fires.
7. **Game state and rules** — `resetGame`, `update(dt)` (movement, firing, spawning, collisions, scoring), `applyPowerUp` (life, bomb, weapon level 1–3 then switch vulcan↔laser), `useBomb`, `loseLife`, `endGame`, `togglePause`.
8. **Weapons and boss patterns** — `fireWeapon`, plus `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`. `BOSS_PATTERNS` is an array (one entry per boss design, looked up by index in `updateBoss`) of attack functions.
9. **Rendering** — `draw()` renders parallax clouds (`drawClouds`), planes (`drawPlane`), boss ships (`drawBossShip`), bullets, power-ups, and HUD effects (screen shake, hit flash, vignette).
10. **Main loop** — `loop(now)` clamps `dt` to 50 ms, calls `trackPerf`, then `update` and `draw` while the game is running and not paused.

Game flow: title overlay → `startBtn` → `resetGame` → waves of enemies per stage → after `KILLS_PER_STAGE` kills a boss spawns (`spawnBoss`; boss design is `BOSSES[(stage - 1) % 6]`) → defeating it (`killBoss`) increments `stage` and changes the theme → lose all lives → `endGame` (updates best score). Each stage's theme is `THEMES[(stage - 1) % 6]`.

## Conventions

- **Keep it a single file** unless the user asks for a split. New code goes inside the existing IIFE, not in global scope.
- **Style**: 2-space indentation, single quotes, `const` by default, `let` only when reassigned. Match the surrounding code's terse style.
- **Config over magic numbers**: tunable values belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, or near their use with a clear name.
- **Accessibility**: respect `prefers-reduced-motion` (the `reducedMotion` flag and `reduced-motion` body class) when adding screen shake or flashing effects. Announce important state changes through `announce()` (the `srLive` live region).
- **Mobile**: any new control needs touch handling (`touchstart`/`touchend`, `preventDefault` where needed) as well as mouse/keyboard. Keep the layout working in portrait and landscape; `resize()` scales the `480×720` base canvas.
- **Storage**: wrap `localStorage` reads/writes in try/catch if you add new keys.
- **Performance**: avoid per-frame allocations in hot paths (`update`, `draw`); respect the `quality` level when adding expensive effects.

## Working on the game

- To verify a change, open `index.html` in a browser and play through: start, move/fire, bomb, pause/resume, lose a life, game over, and reload to confirm the best score persists.
- Check touch controls with a device emulator or DevTools touch mode when touching input code.
- Check the console for errors after any edit; the game runs inside an IIFE, so errors surface only in the console.
- Keep balance changes (spawn rates, boss HP, drop chances in `maybeSpawnPowerUp`) small and call them out in commit messages.

## Git

- Default development branch for this work: `claude/gifted-tesla-q9438q`. Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages in the imperative mood, describing the gameplay or code change.
