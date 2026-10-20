# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up, written as a single self-contained HTML5 canvas page. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (inline `<script>`, wrapped in an IIFE with `'use strict'`).
- `README.md` — a stub; it has no setup or usage instructions yet.

To run the game, open `index.html` in a browser. Nothing needs to be installed.

## Repository layout (`index.html`)

Roughly in order of appearance:

1. **`<style>`** (~lines 10–361) — HUD pills, glassmorphic overlay card, touch controls, vignette/flash effects, and the `.reduced-motion` class.
2. **Markup** (~lines 362–420) — `#container` holding `#gameCanvas`, the HUD (`#score`, `#best`, `#mult`, `#weaponName`/`#weaponLvl`, `#hearts`, `#bombs`), boss bar, `#topBar` (mute / reduced-motion / pause), touch controls (`#moveZone`, `#fireBtn`, `#bombBtn`), and the start overlay (`#overlay`, `#startBtn`, `.diffBtn`).
3. **Script constants** (~lines 422–528) — DOM lookups, `STORAGE_*` keys, adaptive performance (`quality`), `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
4. **Systems** — audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`), HUD rendering helpers, input (keyboard, touch joystick, mouse), and game-state helpers (`resetGame`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore`, `breakChain`, `useBomb`, `fireWeapon`).
5. **Simulation** — `update(dt)`, `applyPowerUp`, the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, … `BOSS_PATTERNS`), `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
6. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
7. **Loop and bootstrap** — `loop(now)` runs on `requestAnimationFrame`; the start button calls `resetGame()`.

## Game design constants to keep in sync

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn timing, enemy speed, enemy fire chance, starting `lives`, starting `bombs`, and `bossHpMul`.
- **Stages**: a stage advances every `KILLS_PER_STAGE` (18) kills. `THEMES` has six entries and the theme cycles with the stage.
- **Enemies** (`ENEMY_TYPES`): `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- **Bosses** (`BOSSES`): six entries, each with its own colors, `hpMul`, and attack pattern drawn from `BOSS_PATTERNS`.
- **Score**: base points per kill are multiplied by a kill-chain multiplier (`addScore`, `breakChain`).

When you change one of these, check the HUD, the overlay copy (`#controlsHint`, `#overlaySub`), and the touch-control labels for consistency.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'`/`'0'`; when absent, falls back to `prefers-reduced-motion` |

Storage access is currently unguarded. If you add new reads or writes, consider wrapping them in `try/catch`, since storage can throw in private windows or when site data is blocked.

## Conventions

- **Single file.** Keep HTML, CSS, and JS in `index.html` unless the user asks for a split. Do not add a bundler or framework.
- **No external dependencies.** The game uses only browser APIs. Do not add CDN scripts without asking.
- **Match the existing style.** Two-space indentation, single quotes in JS, `const` by default, section headers written as `// ---------- Name ----------`, and small named functions over large inline blocks.
- **Fixed logical resolution.** The canvas is scaled from `BASE_W × BASE_H` (480 × 720) in `resize()`. Gameplay coordinates should stay in logical units, not CSS pixels.
- **Time-based updates.** `update(dt)` takes a delta-time argument. Keep movement and timers frame-rate independent.
- **Accessibility.** Announce important state changes through `announce()` (the `#srLive` region). Respect reduced motion: screen shake, flashes, and pulses should go through the existing `reducedMotion` checks.
- **Audio.** Create the `AudioContext` only from a user gesture via `ensureAudio()`. Respect `muted`.

## Testing and verification

There is no automated test suite or linter. To verify a change:

1. Open `index.html` in a browser (or use the Playwright/Chromium setup available in the cloud environment) and play through at least one stage.
2. Check all three difficulties if you touched difficulty-dependent logic.
3. Check touch controls with a mobile viewport if you touched input or layout.
4. Check the console for errors. Run a syntax check if one is available, for example `node --check` on the extracted script.

Say explicitly in your summary when a change has been verified by playing and when it has only been checked by reading.

## Git workflow

- Develop on the branch the session specifies (currently `claude/gifted-tesla-j6tw7w`), not the default branch.
- Write clear, imperative commit messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.
