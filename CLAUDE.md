# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up browser game.

- `index.html` — the entire game: inline CSS (`<style>`, ~lines 1–421), markup (HUD, menus, controls hint), and one inline `<script>` (starts ~line 422) containing all game logic, rendering, audio, and input.
- `README.md` — minimal title only.

There is no build step, package manager, test suite, or dependency manifest. Do not add one unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by loading the game in a browser and playing through the affected path (menus, shooting, bombs, boss fights, pause, game over).

## Code structure (inside the `<script>`)

- **Persistence keys** (`STORAGE_*`): best score, mute, difficulty, reduced motion, stored in `localStorage` under the `skyfighter.*` prefix. Reads are unguarded; keep it that way unless adding a fallback everywhere.
- **Difficulty** (`DIFFICULTIES`): `easy` / `normal` / `hard` control spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier. `KILLS_PER_STAGE` (18) sets stage progression.
- **Entities and content tables**: `ENEMY_TYPES`, `BOSSES`, `THEMES`, and `BOSS_PATTERNS` (bullet pattern functions such as radial, spiral, cross, homing, double ring).
- **Core loop**: `update(dt)` handles movement, firing, collisions, and scoring; `draw()` renders; `loop(now)` drives both per frame. `resetGame()` restores initial state.
- **Systems**: `fireWeapon()` (vulcan and other weapons, scaled by `player.level`), `useBomb()`, `applyPowerUp()`, `updateBoss()` / `killBoss()`, `loseLife()`, `endGame()`, `togglePause()`.
- **Audio**: Web Audio helpers `ensureAudio()`, `beep()`, `noiseBurst()`. Audio must be initialized from a user gesture (`ensureAudio()` is called on input).
- **Accessibility**: `reducedMotion` honors `prefers-reduced-motion` and a stored override; `announce()` writes to a live region.
- **Layout**: `resize()` scales a fixed `BASE_W × BASE_H` (480 × 720) canvas to the viewport.

## Input

- Keyboard: arrows / WASD move, Space fire, X or Shift bomb, P or Esc pause. Key state is tracked in `keys[e.code]`.
- Mouse: move to steer, hold/click to fire (`firing` flag).
- Touch: left-side joystick (`joystickStart/Move/End`) to fly, FIRE and BOMB buttons. Use `{ passive: false }` plus `preventDefault()` on the fire controls.

Keep the controls hint in the markup (`#controlsHint`) in sync if bindings change.

## Conventions

- Keep the game in a single self-contained file. Do not split out modules or introduce a bundler without an explicit request.
- Preserve the existing style: 2-space indentation, `const`/`let`, camelCase functions, UPPER_CASE constant tables.
- Keep the layout responsive down to phone widths (`viewport-fit=cover`, safe-area insets, `touch-action: none` on the play area).
- Do not add external scripts, fonts, or stylesheets. The page currently uses only inline assets and a data-URI favicon.
- When changing gameplay numbers, prefer editing the tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`) over hard-coding values in logic.

## Git

- Default development branch for this session: `claude/gifted-tesla-bs4yq5`.
- Commit messages are short, imperative summaries (e.g. `Add Sky Fighter: ...`).
