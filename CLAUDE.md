# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static project containing **Sky Fighter**, a Raiden-style vertical shoot-'em-up game written as a single-page HTML5 canvas app.

Files:

- `index.html` (~1,670 lines) — the entire game: inline `<style>`, markup, and one inline `<script>` (starts around line 422). No build step, no dependencies, no external assets. Sounds are synthesized with the Web Audio API; graphics are drawn on the canvas.
- `README.md` — placeholder only.

## Running the game

There is no build, package manager, or test suite. Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.

## Code structure (inside `index.html`)

The script is organized by section comments and top-level functions. Key areas:

- **Config constants** (~lines 460–530): `DIFFICULTIES` (easy/normal/hard: lives, bombs, spawn rates, enemy shot chance, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes, cycled by stage), `ENEMY_TYPES` (grunt, interceptor, tank with `minStage` gating), `BOSSES` (cycled per stage, each with an HP multiplier).
- **Storage keys** (`STORAGE_*`): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Changing these keys resets players' saved state.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, `panFor`.
- **Input**: keyboard (`keydown`/`keyup` on `e.code`), mouse (move to steer, click to fire), and touch (joystick zone plus fire/bomb buttons).
- **Game state**: `resetGame()` initializes everything for a new run; `update()` runs per-frame logic; `draw()` renders; `loop()` drives the frame cycle.
- **Weapons and patterns**: `fireWeapon`, `applyPowerUp`, and the bullet pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`). `BOSS_PATTERNS` maps boss attacks; `updateBoss` / `killBoss` manage boss fights.
- **Progression**: a stage advances after `KILLS_PER_STAGE` kills, then a boss fight. Kill chains feed a score multiplier (`breakChain`, `addScore`).
- **Rendering helpers**: `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion`, `screenShake`, `flashHit`, `flashBomb`.
- **UI/HUD**: element IDs such as `score`, `mult`, `weaponName`, `hearts`, `bombs`, `bossBarFill`, `overlay`, `startBtn`, and `touchControls`. The `announce()` helper updates the `srLive` aria-live region for screen readers.
- **Adaptive quality**: `trackPerf` samples the last 45 frame times. If the average is slow (under ~40 FPS) it steps `quality` up (0 = full, 1 = medium, 2 = low, i.e. cheaper rendering); if there is headroom (over ~56 FPS) it steps back down.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a build step, bundler, or npm dependency without explicit approval.
- Use plain ES2015+ JavaScript and CSS that run in modern evergreen browsers. Keep the existing style: `const`/`let`, arrow functions, and 2-space indentation.
- Preserve mobile behavior: the layout uses `viewport-fit=cover`, safe-area insets, `touch-action: none`, and touch controls. Test changes to input or layout at phone width.
- Respect the reduced-motion and mute settings when adding effects or sounds. Reduced motion defaults to the `prefers-reduced-motion` media query unless the user has toggled it (`STORAGE_REDUCED`); mute is `STORAGE_MUTE`. Screen shake and hit/bomb flashes are suppressed or softened when reduced motion is on.
- Any new UI text should be announced to assistive tech via `announce()` when it reflects game state changes.
- Use the stored-state key names as-is; treat them as a public contract.

## Verification

Since there are no automated tests, verify gameplay changes by loading `index.html` in a browser and checking:

1. Title overlay appears; each difficulty starts a run with the expected lives and bombs.
2. Firing, weapon power-ups, bombs (X / Shift), and pause (P / Esc) work.
3. Stage banner, stage progression, and boss fight (health bar, patterns, defeat) work.
4. Game over shows the final score, and best score persists after reload.
5. Touch controls work on a phone-sized viewport, with no horizontal scroll.
6. The browser console shows no errors.

## Git workflow

- Default working branch for this session: `claude/gifted-tesla-p90mdh`. Push there with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
