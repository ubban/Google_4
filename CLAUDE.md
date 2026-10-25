# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-em-up (Raiden-style). There is no build step, no package manager, and no test suite.

Files:

- `index.html` — the entire game: CSS (inline `<style>`, lines 10–359), markup (lines 361–~420), and JavaScript (inline `<script>`, lines 422–1667).
- `README.md` — a placeholder; it contains no project documentation.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There is nothing to install or compile.

## Architecture (inside `index.html`)

All game code lives in one IIFE (`(function () { 'use strict'; ... })();`) to avoid globals. Keep new code inside it.

Key sections, in source order:

- **DOM references** — `getElementById` lookups for HUD, overlay, and touch controls, grouped at the top of the script.
- **Storage keys** — `STORAGE_*` constants for `localStorage` (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
- **Config tables** — `DIFFICULTIES` (spawn rate, enemy speed, lives, bombs, boss HP multiplier), `THEMES` (sky palettes; stage N uses `THEMES[(stage - 1) % THEMES.length]`), `ENEMY_TYPES` (grunt / interceptor / tank with `minStage` gating), and `BOSSES` (six bosses, one per stage cycle).
- **Adaptive quality** — `trackPerf()` samples frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low).
- **Audio** — a lazily created `AudioContext` (`ensureAudio()`), with `beep()` / `noiseBurst()` helpers and an `sfx` object. Audio is created only after a user gesture.
- **Input** — keyboard (`keys` map keyed by `e.code`), mouse/pointer on the canvas (`canvasPointFromEvent`), and a touch joystick plus FIRE / BOMB buttons.
- **Game state and entities** — `resetGame()` initializes state; `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `fireWeapon`, and `useBomb` mutate arrays of entities.
- **Update / draw** — `update(dt)` advances simulation, `draw()` renders to the canvas, and `loop(now)` drives both via `requestAnimationFrame`. `dt` is clamped to 50 ms.
- **Bosses** — `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring); `updateBoss` selects among them.
- **Accessibility** — `announce()` writes to an `aria-live` region; `reducedMotion` (user toggle, defaulting to `prefers-reduced-motion`) suppresses screen shake and flashes.

### Gameplay facts worth knowing

- Lives, bombs, and boss HP scale with difficulty (`easy` / `normal` / `hard`).
- Every `KILLS_PER_STAGE` (18) kills advances the stage; each stage ends in a boss fight.
- Power-ups: `life`, `bomb`, and `weapon`. Weapon upgrades go level 1 → 2 → 3, then toggle between `vulcan` and `laser` and reset to level 1.
- Controls: arrows / WASD or mouse to move, Space or click to fire, X / Shift to bomb, P / Esc to pause.

## Conventions

- **Match the existing style.** The code uses 2-space indentation, single quotes in JS, `const` / `let` (no `var`), arrow functions for callbacks, and short section comments in the form `// ---------- Name ----------`.
- **No new dependencies or build tooling.** Keep the game as a self-contained static page. Don't add external scripts or fonts without a clear reason.
- **Persist state only through the existing `STORAGE_*` keys**, and wrap any new storage access so the game still works when `localStorage` is unavailable.
- **Respect reduced motion.** Any new screen shake, flash, or heavy particle effect should check `reducedMotion` and degrade gracefully.
- **Keep the frame loop cheap.** Gameplay runs every frame; avoid per-frame allocations where a reused value will do, and respect the `quality` level when adding visual effects.
- **Keep user-facing text in sync.** If you change controls, update `#controlsHint` in the overlay markup as well as the mobile hint.

## Verifying changes

There are no automated tests or linters. To check a change:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through at least one full stage to exercise spawning, boss fights, and stage transitions.
3. Check the three difficulty levels, the pause/resume flow, and game over → restart.
4. For touch changes, test with device emulation in dev tools.
5. Check reduced-motion behavior if you touched effects.

Syntax-check the inline script if you want a quick smoke test, for example:

```bash
node -e "const h=require('fs').readFileSync('index.html','utf8'); new Function(h.split('<script>')[1].split('</script>')[0])"
```

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-675xfh`).
- Write clear, descriptive commit messages.
- Do not open a pull request unless explicitly asked.
