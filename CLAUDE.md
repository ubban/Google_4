# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html` (inline CSS and one inline `<script>` IIFE, no external dependencies, no build step). `README.md` is a placeholder.

There is no package manager, bundler, test suite, linter, or CI configuration in the repo. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

- `index.html`: the entire game (markup at ~L1–420, CSS in the `<style>` block at the top, game code in the `<script>` at ~L422 onward).
- `README.md`: placeholder title only.
- `CLAUDE.md`: this file.

## Architecture of `index.html`

Everything inside the script is wrapped in an IIFE. Key sections, roughly in file order:

- **Config constants**: `STORAGE_*` localStorage keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`), `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, each with its own colors and HP multiplier).
- **Audio**: Web Audio API synth (`ensureAudio`, `beep`, `noiseBurst`, `sfx.*`). No audio files. Audio is created lazily on the first user gesture.
- **Rendering**: `drawPlane`, `drawBossShip`, and related draw helpers, with parallax clouds, sun glow, and particle/explosion effects.
- **Spawning and entities**: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`. Power-up types are `life`, `bomb`, and `weapon`.
- **Weapons and bosses**: `fireWeapon` (Vulcan/Laser with power levels), `useBomb` (screen clear, damages bosses), `BOSS_PATTERNS` and the `fire*` pattern functions, `updateBoss`.
- **Game state**: `resetGame`, `update(dt)`, `applyPowerUp`, `addScore` and the kill-chain multiplier (`breakChain`), `togglePause`.
- **Input**: keyboard (`keydown`/`keyup` on `window`, using `e.code`), mouse on the canvas, and touch joystick plus FIRE/BOMB buttons for mobile.
- **Loop**: `loop(now)` runs under `requestAnimationFrame`; `update(dt)` and the draw calls run only when `running && !paused`. `resize()` scales the fixed 480×720 logical canvas (`BASE_W`/`BASE_H`) to the viewport.

Game flow: start screen with difficulty select → play → stages advance every 18 kills, each with a new theme and a boss fight → game over. Best score and settings persist via localStorage.

## Controls (keep the in-game hint in sync)

The hint text lives in `#controlsHint`. If you change controls, update that element too.

- Keyboard: arrows/WASD move, Space fires, X or Shift bombs, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag left zone to fly, tap FIRE, tap BOMB.

## Conventions and guidance

- **Keep it a single file.** Do not add a build system, framework, or external dependencies unless the user asks. Keep CSS inline in the `<style>` block and JS inline in the script.
- **Preserve the storage keys.** Renaming any `skyfighter.*` key silently resets players' saved best score, mute, difficulty, and motion settings.
- **Respect reduced motion.** Screen shake, hit flash, and the low-health pulse are gated by `reducedMotion`. New visual effects that move the screen should be gated the same way.
- **Keep the logical resolution fixed.** Game logic assumes the 480×720 base size; coordinates and entity sizes are in those units, and `resize()` handles display scaling.
- **Use `dt`-based movement.** `update(dt)` is frame-rate independent; new motion should multiply by `dt` rather than assume a fixed frame rate.
- **Be careful with the pause/bomb guards.** `useBomb` and `togglePause` check `running`, `paused`, and `gameOver`. Preserve those checks when editing input handling.
- **Mobile first.** The layout uses `touch-action: none`, safe-area insets, and a portrait-friendly HUD. Verify changes at phone width as well as desktop.
- **Audio starts on gesture.** Call `ensureAudio()` from user-input handlers, not on load, to satisfy browser autoplay policies.

## Verifying changes

There are no automated tests. For changes to gameplay or UI:

1. Open `index.html` in a browser (or serve the folder statically).
2. Play through at least one stage on each difficulty you touched, and check the boss fight if you changed boss, bomb, or weapon code.
3. Check the pause (P/Esc), mute, reduced-motion toggle, and game-over → restart path.
4. Check the layout at a phone-sized viewport and at desktop size.
5. Confirm the browser console shows no errors.

If you add a test harness or tooling, document the commands here.

## Git

- Development branch for this work: `claude/gifted-tesla-py05q9`.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
