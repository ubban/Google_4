# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free web game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game is a single file, `index.html` (~1,670 lines), containing inline `<style>`, markup, and one IIFE-wrapped `<script>` that draws everything on a `<canvas>`.

- `README.md` — placeholder; contains only the title.
- `index.html` — the whole game.

There is no build step, package manager, bundler, test suite, linter, or CI configuration. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`) and visit it. No install step is needed.

## Code layout inside `index.html`

- **Markup and CSS (lines ~1–421):** HUD (`#hud`), boss bar (`#bossBarWrap`), touch controls (`#touchControls`), start/pause overlay (`#overlay`), and a `sr-only` live region (`#srLive`) for screen-reader announcements.
- **Script (starts ~line 422):** a single `(function () { 'use strict'; ... })()` wrapper. Nothing is global; keep it that way.

Major sections of the script, in source order:

| Section | What it holds |
| --- | --- |
| Element refs and constants | `getElementById` lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Adaptive performance | `trackPerf` lowers effect quality (`quality` 0–2) when frame time stays high |
| Settings | best score, mute, difficulty, reduced motion — all persisted to `localStorage` |
| Audio | lazily created `AudioContext` (`ensureAudio`), `beep`/`noiseBurst` helpers, `sfx` table |
| HUD helpers | hearts, bombs, combo pop, stage banner, screen shake |
| Input | keyboard (`keys`), virtual joystick and buttons for touch, mouse steering and fire |
| `resetGame` / `update(dt)` | game state and per-frame simulation |
| Weapons, bombs, power-ups | `fireWeapon`, `useBomb`, `applyPowerUp`, `maybeSpawnPowerUp` |
| Bosses | `spawnBoss`, `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, double ring), `updateBoss`, `killBoss` |
| Drawing | `drawPlane`, `drawBossShip`, `draw()` |
| Loop | `loop(now)` drives `update` and `draw` via `requestAnimationFrame` |

### Gameplay rules worth knowing

- **Stages:** each stage requires `KILLS_PER_STAGE` (18) kills. Reaching it triggers a boss warning, then `spawnBoss()`. The boss for stage *n* is `BOSSES[(n - 1) % BOSSES.length]`, and its HP scales with stage, difficulty (`bossHpMul`), and the boss's own `hpMul`.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Enemies:** `grunt` (stage 1+), `interceptor` (stage 1+, weaving), `tank` (stage 2+, 4 HP).
- **Power-ups:** `life`, `bomb`, `weapon`. The weapon power-up raises the level 1→3 and then toggles between `vulcan` and `laser`.
- **Kill chain:** kills build a score multiplier (`addScore`, `breakChain`).
- **Persistence:** best score and settings use `localStorage` keys prefixed `skyfighter.`. Keep that prefix and the existing key names so saved data keeps working.

## Conventions

- **Single file, vanilla JS.** Don't introduce frameworks, npm dependencies, or external CDN scripts without an explicit request.
- **Match the local style:** two-space indent, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, short section comments in the form `// ---------- Name ----------`.
- **Keep the IIFE.** Don't leak new globals; add new state as `let`/`const` inside the wrapper.
- **Frame-rate independence:** movement and timers use the `dt` passed to `update(dt)`. Preserve that when adding entities.
- **Accessibility:** respect the reduced-motion toggle (`reducedMotion`) for shake, flash, and pulse effects. Announce important state changes through `announce()`. Keep touch controls and keyboard controls both working.
- **Performance:** avoid allocating large arrays or objects per frame in hot paths (`update`, `draw`); consider `quality` when adding costly effects.

## Testing and verification

There is no automated test suite. Verify changes by:

1. Opening the game in a browser (desktop and, if the change touches input or layout, a narrow/touch viewport).
2. Playing through at least one stage boss on each relevant difficulty when changing combat, bosses, or difficulty values.
3. Checking the browser console for errors.
4. Confirming that pause (`P`/`Esc`), mute, reduced-motion toggle, and difficulty selection still work and persist across reloads.

If a headless browser is available, a Playwright script is a reasonable way to smoke-test start, play, and game over.

## Git workflow

- Develop on the branch the task specifies (currently feature branches such as `claude/...`). Don't push to other branches without permission.
- Write clear commit messages in the imperative mood. Existing history uses a short subject line followed by a bulleted body describing the change.
- Don't open pull requests unless explicitly asked.

## When editing this file

Keep `CLAUDE.md` accurate as the game changes. If you change the file structure, add or remove a major section, alter a difficulty value, add a boss, or change a `localStorage` key, update the relevant section here in the same change.
