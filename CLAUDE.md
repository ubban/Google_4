# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up built as an HTML5 canvas game. There is no build step, no package manager, and no test suite.

Files:

- `index.html` (~1,670 lines) — the entire game: inline `<style>` (lines ~10–359), HTML markup with the canvas and HUD/overlay/touch controls, and one inline `<script>` (starts ~line 422) wrapped in an IIFE.
- `README.md` — minimal project readme.

## Running the game

No build. Open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`) and visit it. Nothing to install.

## Code structure (inside `index.html`)

The script is a single IIFE. Key sections, roughly in file order:

- **DOM refs and storage keys** — `STORAGE_*` constants use the `skyfighter.*` prefix (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
- **Config tables** — `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `THEMES` (six sky palettes that rotate by stage), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses cycled by stage), `KILLS_PER_STAGE`.
- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, `panFor`. Procedural WebAudio only; there are no audio files.
- **Adaptive performance** — `trackPerf` drops a `quality` level (0/1/2) when frame time is poor.
- **Input** — keyboard (`keys` map keyed by `e.code`), mouse/canvas pointer, and touch joystick + fire/bomb buttons.
- **Game state and loop** — `resetGame`, `update(dt)`, `draw()`, `loop(now)` driven by `requestAnimationFrame`.
- **Weapons and power-ups** — `fireWeapon`, `applyPowerUp`, `maybeSpawnPowerUp`; three weapon levels (Vulcan/Laser).
- **Bosses** — `BOSS_PATTERNS` (bullet patterns such as radial, spiral, cross, homing), `spawnBoss`, `updateBoss`, `killBoss`.
- **Scoring** — `addScore`, kill-chain multiplier (`breakChain`), best score persisted on game over.
- **Lifecycle** — `loseLife`, `endGame`, `togglePause`.

### Controls (for reference when editing input code)

- Move: Arrow keys or WASD
- Fire: hold Space (keyboard) or fire button (touch/mouse)
- Bomb: `X` or Shift
- Pause: `P` or Escape
- Mute, reduced-motion, and difficulty are toggled in the UI and persisted to localStorage.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. The single-file, zero-dependency design is intentional.
- Match the existing style: 2-space indentation, `const`/`let`, small named functions, short comments that label sections (`// ---------- Section ----------`).
- Game tuning values live in the config tables near the top of the script. Prefer changing those over hard-coding numbers in update/draw logic.
- Keep the game fully playable without audio and without localStorage. Guard storage access if you add new keys.
- Respect `prefers-reduced-motion` and the in-game reduced-motion toggle when adding screen shake, flashes, or other motion effects.
- Keep the layout working on small touch screens; the canvas scales via `resize()` against `BASE_W` × `BASE_H` (480 × 720).
- Accessibility: announce important state changes through the `announce()` live region.

## Making changes

- There are no automated tests or linters. Verify changes by loading `index.html` in a browser and playing through at least one stage, plus the affected feature (pause, bomb, boss, touch controls, etc.).
- When changing a boss or enemy, check both the hard-coded pattern logic and the config entries in `BOSSES` / `ENEMY_TYPES`.
- Do not introduce external CDN scripts, fonts, or assets without a specific request; the game currently has no network dependencies.

## Git

- Work on the branch you were assigned; do not push to other branches without permission.
- Commit messages in this repo use a short imperative subject followed by a bulleted body describing the change (see `git log`).
