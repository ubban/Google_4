# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a single-file, Raiden-style vertical shoot-'em-up that runs in the browser. There is no build step, no package manager, no test suite, and no external dependencies.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (an IIFE) containing all game logic, rendering, audio, and input.
- `README.md` — minimal project title. Keep it short.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is needed.

## Code layout (inside `index.html`)

The inline script is organized roughly in this order:

1. **DOM references** — `getElementById` lookups for HUD, overlays, and touch controls.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Persisted via `localStorage`.
3. **Tuning tables** — `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, cycled by stage). `KILLS_PER_STAGE` controls stage progression.
4. **Audio** — lazily created `AudioContext`; `beep()`, `noiseBurst()`, and the `sfx` table. Respects the mute setting.
5. **HUD helpers** — hearts, bombs, combo pop, stage banner, screen shake, vignette/flash.
6. **Input** — keyboard (`keys` map keyed by `e.code`), mouse steering and click-to-fire, and touch joystick plus FIRE/BOMB buttons.
7. **Game state** — `resetGame()` initializes the `player`, arrays (`bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`), `boss`, score, lives, bombs, chain, and multiplier.
8. **Drawing** — `drawPlane`, `drawBossShip`, and related canvas routines.
9. **Update/spawn loop** — enemy spawning, weapon firing, collisions, power-up pickup (`applyPowerUp`), boss phases, and game-over handling.
10. **Adaptive quality** — `trackPerf()` samples frame times and steps the `quality` level up (0 full, 1 medium, 2 low, i.e. less visual detail) when frames run slow, and back down when they recover.

Canvas is a fixed logical size of `BASE_W` × `BASE_H` (480 × 720), scaled to fit the window by `resize()`.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce a bundler, framework, or npm dependencies without being asked.
- **No external network calls.** The page makes no `fetch` or script-tag requests to other origins. Keep it that way.
- **Preserve the data-driven tables.** Balance changes should go into `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, and similar tables rather than being hard-coded into logic.
- **Respect accessibility settings.** Reduced-motion mode disables screen shake, flashes, and pulses. New visual effects should honor `reducedMotion`. Screen-reader announcements go through `announce()` into the `srLive` region.
- **Keep both input paths working.** Any new control must work with keyboard, mouse, and touch. The touch layout must stay usable at phone width.
- **Don't break persistence.** Storage access is currently direct `localStorage` calls. If you add more, keep the same `skyfighter.*` key prefix.

## Verification

There are no automated tests or linters. To verify a change:

1. Open `index.html` in a browser and play through at least one stage, including a boss fight.
2. Check all three difficulties if balance or spawn logic changed.
3. Confirm mute, reduced-motion, and pause toggles still work and persist across reloads.
4. For layout changes, check both a desktop window and a narrow phone-sized viewport.
5. Browser console should show no errors during normal play.

## Git workflow

- Develop on the branch specified for the session. Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open pull requests unless the user explicitly asks.
