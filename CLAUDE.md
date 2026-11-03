# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-page HTML5 canvas game. The entire game (markup, CSS, and JavaScript) lives in
`index.html`. `README.md` is a placeholder and carries no project documentation.

There is no build step, package manager, test suite, or linter. Nothing needs to be
installed.

## Repository layout

| Path         | Purpose                                                              |
| ------------ | -------------------------------------------------------------------- |
| `index.html` | The whole game: inline `<style>` (~lines 10–362), canvas, and one `<script>` (~line 422 onward) |
| `README.md`  | Placeholder only                                                     |

Inside the main `<script>`, everything is wrapped in an IIFE `(function () { ... })()`,
so nothing leaks onto `window`. Keep new code inside that wrapper.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server.
- There are no automated tests. Verify changes by playing the game in a browser. Check at
  least the affected system, plus the title screen, pause, and game-over flows.
- For mobile-layout changes, check both a touch-sized viewport and a desktop one. The
  canvas is scaled from a fixed logical size (`BASE_W = 480`, `BASE_H = 720`) in `resize()`.

## Architecture notes

Key sections of the main script, in file order:

- **Config constants:** `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`,
  `ENEMY_TYPES`, `BOSSES`.
- **Audio:** Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`) and per-enemy explosion
  SFX (`EXPLODE_SFX`). Audio is created lazily on first user gesture via `ensureAudio()`.
- **HUD and feedback:** `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`,
  `showStageBanner`, `screenShake`.
- **Input:** keyboard handlers, a virtual joystick for touch (`joystickStart/Move/End`), and
  pointer mapping through `canvasPointFromEvent`.
- **Game state:** `resetGame()` initializes all state. `update(dt)` advances the simulation
  and `draw()` renders it. `loop(now)` drives both.
- **Weapons and bombs:** `fireWeapon()`, `useBomb()`, and `applyPowerUp()`.
- **Bosses:** `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread,
  spiral, cross, homing, double ring, and others). `updateBoss` and `killBoss` manage phases.
- **Persistence:** `localStorage` only (see below).

### Gameplay constants that matter

- Difficulty (`easy` / `normal` / `hard`) changes spawn rate, enemy speed, enemy shoot
  chance, starting lives, starting bombs, and boss HP multiplier. The current difficulty is
  stored in `localStorage`.
- A stage is `KILLS_PER_STAGE` (18) kills. Stage *N* uses `THEMES[(N-1) % THEMES.length]` for
  the background and `BOSSES[(N-1) % BOSSES.length]` for its boss, so both cycle.
- Enemy types are `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2).
- Power-up types are `life`, `bomb`, and `weapon`.

## Controls

- Move: Arrow keys or WASD.
- Fire: Space (hold to keep firing).
- Bomb: `X` or Shift.
- Pause: `P` or Escape.
- Touch: virtual joystick for movement, plus on-screen `FIRE` and bomb buttons.

## Persisted data

These `localStorage` keys are part of the user-facing contract. Do not rename them without
a migration plan:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

The reduced-motion default follows `prefers-reduced-motion` until the user sets it.

## Conventions

- Plain vanilla JavaScript (ES2015+) and CSS. Do not add frameworks, bundlers, or CDN
  dependencies unless the user asks.
- Keep the single-file structure unless the user asks for a split.
- Name constants in `UPPER_SNAKE_CASE` and keep the existing tuning-table style for new
  enemies, bosses, and difficulty settings.
- Respect the reduced-motion setting when adding screen shake, flashes, or other motion
  effects. Gate them through the existing `reducedMotion` flag.
- Keep the game loop allocation-light. Reuse existing arrays and objects where practical.
- Do not add analytics, network calls, or external assets.

## Git workflow

- Develop on the branch the session names. Do not push to `main` directly.
- Write descriptive commit messages that explain the gameplay or code change.
