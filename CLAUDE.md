# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

- `index.html` — the entire game: inline `<style>`, markup, and one `<script>` IIFE. No build step, no dependencies, no external assets (sounds are synthesized with the Web Audio API; icons are inline data URIs).
- `README.md` — minimal project readme.

There is no package manager, test suite, linter, or CI configured. Verify changes by opening `index.html` in a browser.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it. Nothing needs to be installed.

## Code structure (`index.html`)

- **Styles** (`<style>`, roughly lines 10–420): CSS custom properties are not used; colors are literals. HUD is a pill-style bar with heart lives and bomb icons; overlays are glassmorphic cards; keyframes handle combo pops, stage banners, hit flashes, and the low-health vignette.
- **Markup** (roughly lines 380–420): canvas, HUD, overlay/start card, pause button, mute and reduced-motion toggles, mobile joystick and FIRE/BOMB buttons, and the controls hint text. Keep the controls hint in sync with the actual key bindings.
- **Script** (`<script>` at ~line 422, wrapped in an IIFE): all game state lives in closure variables. Key sections, in order:
  - Storage keys (`skyfighter.*`) and persisted settings (best score, mute, difficulty, reduced motion).
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  - Audio (`ensureAudio`, `beep`, `noiseBurst`) and HUD/effects helpers.
  - Input: keyboard (`keys` map), touch joystick and buttons, and mouse steering/firing on the canvas.
  - Game lifecycle: `resetGame`, `update(dt)`, pause handling, game over, and the stage/boss flow.
  - Rendering: `drawPlane`, `drawBossShip`, and related draw helpers.
  - Spawning and combat: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `addScore` / `breakChain` (kill-chain multiplier).
  - Boss attack patterns: `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, columns, spiral, cross, homing, double ring).

## Game rules worth knowing

- **Controls:** mouse steers and clicks to fire; Arrows/WASD move; Space fires; X or Shift drops a smart bomb; P or Esc pauses. Mobile uses a left-side drag joystick, a FIRE button, and a BOMB button.
- **Difficulty** (`easy` / `normal` / `hard`) changes spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages:** a stage ends after `KILLS_PER_STAGE` kills, which triggers a boss fight from `BOSSES`. Defeating the boss increments `stage`, which advances the `THEMES` palette and unlocks stage-gated enemies (`minStage`).
- **Power-ups:** `weapon` (raises the current weapon level from 1 to 3; at level 3 it switches between Vulcan and Laser and resets to level 1), `bomb` (+1 smart bomb, capped), `life` (+1 life, capped).
- **Persistence:** only the best score, mute state, difficulty, and reduced-motion preference are saved, in `localStorage` under `skyfighter.*` keys. Keep these keys stable; changing them resets players' saved settings.

## Conventions

- Keep the game as a single file unless there is a clear reason to split it. Do not add a build system or external dependencies without asking.
- Preserve the existing style: 2-space indentation, `const`/`let`, plain ES2015+ JavaScript, no frameworks.
- Tuning values live in the constant tables near the top of the script. Prefer editing those over hard-coding numbers in logic.
- Respect the reduced-motion setting: effects that shake the screen or flash should early-return when `reducedMotion` is true (see `screenShake`, `flashHit`, `flashBomb`).
- Keep touch and keyboard paths working together; any new action needs both a key binding and a mobile control, plus an update to the controls hint.
- Every new visual or audio effect should stay cheap enough for mobile frame rates; the `update(dt)` loop is the hot path.

## Git workflow

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-fr8d8k`); do not push to `main` directly.
- Write descriptive commit messages that explain the gameplay or code change, not just the file touched.
- Only open a pull request when explicitly asked.

## Manual verification checklist

Since there are no automated tests, check these in a browser after gameplay changes:

1. Start screen appears; starting a run works with each difficulty.
2. Keyboard and mouse both fire and move; pause with P/Esc resumes correctly.
3. Bomb clears the screen and damages bosses; X and Shift both trigger it.
4. Stage transition and boss fight start after the kill threshold; boss health bar updates.
5. Power-ups (weapon, bomb, life) are picked up and reflected in the HUD.
6. Best score persists across reloads; mute and reduced-motion toggles persist.
7. Layout works at phone width and in landscape with touch controls.
