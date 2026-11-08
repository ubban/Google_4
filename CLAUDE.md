# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up built as a
single-page HTML5 canvas game. The whole game lives in one file, `index.html`
(~1,670 lines: HTML, CSS, and one inline `<script>` IIFE). There is no build
step, no package manager, no bundler, and no third-party dependencies.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup (lines ~1–361), CSS (in `<style>`), and game logic (inline `<script>` starting ~line 422). |
| `README.md` | Minimal project readme. |
| `CLAUDE.md` | This file. |

## Running the game

- Open `index.html` directly in a browser. No server is required.
- Desktop: mouse or keyboard. Mobile: touch controls (shown automatically on touch devices).
- There is no test suite, linter, or CI configured. Verify changes by playing the game
  in a browser (and, for layout changes, by checking a phone-width viewport).

## Architecture of `index.html`

Everything is inside one `(function () { 'use strict'; ... })();` IIFE. Key sections,
in source order:

1. **DOM references** (`const canvas`, `const scoreEl`, `startBtn`, etc.) — all looked up
   once at startup via `getElementById` / `querySelectorAll`.
2. **Storage keys and persistence** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`,
   `STORAGE_REDUCED` (all prefixed `skyfighter.`). Best score is written in `endGame()`.
3. **Accessibility** — `announce()` writes to the `#srLive` aria-live region.
4. **Adaptive performance** — `trackPerf()` samples frame times and steps a `quality`
   level (0 full / 1 medium / 2 low) up or down.
5. **Game data tables** —
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy fire
     chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18): kills needed before a boss warning triggers.
   - `THEMES`: six sky palettes, cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). `minStage` gates availability.
   - `BOSSES`: six boss designs, selected by `(stage - 1) % BOSSES.length`.
6. **Audio** — a lazily created `AudioContext` (`ensureAudio()`), with `beep()` and
   `noiseBurst()` primitives and a named `sfx` object. Audio is created on first user
   gesture; keep it that way to satisfy browser autoplay rules.
7. **UI helpers** — `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`,
   `screenShake`, `flashHit`, `flashBomb`, `setDifficulty`.
8. **Layout / input** — `resize()` scales the canvas (base 480×720) to the viewport.
   Keyboard (`keys` map, keyed by `e.code`), mouse (`mouseTarget`, `canvasPointFromEvent`),
   and touch (joystick + FIRE/BOMB buttons) feed the same `keys`/`firing`/`joyVec` state.
9. **Game state** — top-level `let`s: `score`, `lives`, `maxLives`, `bombCount`,
   `stage`, `stageKills`, `chain`, `multiplier`, `player`, `enemies`, `bullets`,
   `powerUps`, `boss`, etc. `resetGame()` reinitializes them.
10. **Update logic** — `update(dt)` moves the player, fires weapons (`fireWeapon()`),
    spawns enemies (`spawnEnemy()`), handles collisions, stage progression, and
    power-up pickups (`applyPowerUp()`).
11. **Boss logic** — `spawnBoss()`, `updateBoss(dt)`, `killBoss()`, and a set of
    bullet-pattern generators (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`,
    `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, …) wired together by `BOSS_PATTERNS`.
12. **Lifecycle** — `loseLife()` (grants 1.5 s invulnerability), `endGame()`,
    `togglePause()`, `useBomb()`.
13. **Rendering** — `draw()` renders the parallax sky, clouds (`drawClouds`), planes
    (`drawPlane`), boss ships (`drawBossShip`), bullets, power-ups, and HUD effects.
14. **Main loop** — `loop(now)` computes `dt` from `requestAnimationFrame` timestamps,
    calls `update(dt)` then `draw()`, and feeds `trackPerf()`.

### Gameplay rules worth knowing

- **Weapons:** `vulcan` and `laser`, each with 3 power levels. A weapon power-up raises
  the level; past level 3 it switches weapon and resets to level 1.
- **Lives and bombs** are capped at `maxLives` and `maxBombs` (2 by default, 3 on easy
  for bombs). Bombs clear the screen and damage bosses.
- **Power-up drop odds** (`maybeSpawnPowerUp`): 6% extra life, 8% bomb, 20% weapon.
- **Score multiplier** comes from a kill chain (`chain`, `chainTimer`, `breakChain()`).
  Taking a hit breaks the chain.
- **Stages:** after `KILLS_PER_STAGE` kills, a boss warning fires and `spawnBoss()` runs.
  Defeating the boss advances the stage and cycles the theme.

## Controls (for reference when testing)

- Move: arrow keys or WASD, or mouse (steers the ship toward the cursor).
- Fire: Space (held), or hold left mouse button.
- Bomb: X or Shift.
- Pause: P or Esc (or the pause button).
- Mute, reduced-motion toggle, and difficulty selector are in the top bar / overlay.

## Conventions

- **Keep it a single file.** New code goes into `index.html` unless the user asks
  for a split. Do not introduce a build step or external dependencies without asking.
- **Vanilla JS only.** Use `const`/`let`, arrow functions, and the existing helper style.
  Match the surrounding code's formatting (2-space indent, single quotes in JS).
- **Constants over magic numbers.** Tunable values belong in the data tables near the
  top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, etc.).
- **Wrap `localStorage` access** if you add new keys, since storage can be unavailable
  (private mode, blocked site data). Use the existing `skyfighter.*` prefix.
- **Respect reduced motion and accessibility.** Screen shake and flashes are gated by the
  reduced-motion setting; announce important state changes through `announce()`.
- **Keep touch working.** Any new control needs a touch path as well as keyboard/mouse.
- **Preserve the mobile layout.** The HUD and touch controls are sized with `clamp()` and
  `env(safe-area-inset-*)`; check phone-width viewports after UI changes.

## Git workflow

- Develop on the branch specified by the session (currently `claude/gifted-tesla-jkslrk`).
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
