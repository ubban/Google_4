# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its main deliverable is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs entirely in the browser:

- `index.html` — the whole game: markup (~lines 1–420), CSS (`<style>`, ~lines 10–362),
  and one IIFE-wrapped `<script>` (~lines 422–1668) containing all game logic.
- `README.md` — a minimal project readme.

There is no build step, bundler, package manager, framework, or test suite. Do not add
any of these unless the user asks. To run the game, open `index.html` in a browser
(or serve the directory with any static file server).

## Architecture of the game script

The script is a single `(function () { 'use strict'; ... })();` closure. Key sections,
in source order:

| Section | Purpose |
| --- | --- |
| DOM refs (top of script) | `getElementById` lookups for HUD, overlay, touch controls, etc. |
| `STORAGE_*` keys | `localStorage` persistence keys (see below). |
| Adaptive performance (`trackPerf`) | Adjusts a `quality` level (0–2) from rolling frame times. |
| `DIFFICULTIES` | Per-difficulty tuning: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`. |
| `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` | Stage pacing, sky palettes, enemy and boss definitions. |
| Audio (`ensureAudio`, `beep`, `noiseBurst`) | WebAudio synthesis; no audio asset files. |
| Input | Keyboard (`keys` map), mouse/pointer steering and firing, touch joystick, fire and bomb buttons. |
| `resize()` | Fits a `BASE_W × BASE_H` (480 × 720) logical canvas to the viewport. |
| `resetGame`, `update(dt)`, `draw()` | Game state reset, fixed-step-style simulation, and rendering. |
| Boss patterns (`BOSS_PATTERNS`, `fire*` helpers) | Bullet pattern generators used by `updateBoss`. |
| `loop(now)` | `requestAnimationFrame` loop; clamps `dt` to 0.05 s; calls `update` then `draw` when running and not paused. |

Conventions to follow when editing:

- Keep everything inside the existing IIFE; do not introduce globals.
- Tune gameplay through the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `BOSS_PATTERNS`) rather than hard-coding magic numbers in logic.
- Time-based motion uses `dt` (seconds); do not assume a fixed frame rate.
- Visual effects should respect `reducedMotion` (body class `reduced-motion`), and
  screen shake goes through `screenShake`.
- Screen-reader announcements go through `announce()` into the `#srLive` region.
- Preserve the `'use strict'` mode and existing indentation style (2 spaces).

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

- `skyfighter.bestScore` — high score, written on game over.
- `skyfighter.muted` — `'1'` when audio is muted.
- `skyfighter.difficulty` — `easy` | `normal` | `hard`.
- `skyfighter.reducedMotion` — `'1'` / `'0'`; if absent, falls back to the
  `prefers-reduced-motion` media query.

Storage access is not wrapped in try/catch today. If you add more storage use, guard it
so the game still runs when storage is blocked (private browsing, disabled site data).

## Controls (must stay in sync with the on-screen hint)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows / WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- Touch: drag in the left zone to fly (joystick), tap FIRE, tap BOMB.

If you change a binding, update the `#controlsHint` text (~line 417) as well.

## Working in this repo

- **Verify changes by running the game.** There are no automated tests or linters, so
  open the page in a browser (or use the `run` skill / a headless browser) and check
  gameplay, the HUD, and the overlay screens after edits. At minimum, confirm there are
  no console errors and that the script still parses.
- Keep changes focused. Do not reformat unrelated code in `index.html`; the file is
  large and the diff should stay reviewable.
- Keep the game self-contained: no external scripts, stylesheets, or asset URLs unless
  the user asks for them. Do not commit secrets or credentials.

## Git and branches

- Default development branch for this session: `claude/gifted-tesla-mrasf7`.
  Push there with `git push -u origin <branch>`; do not push to other branches without
  explicit permission.
- Do not open a pull request unless the user explicitly asks for one.
- Commit messages are short, imperative, and describe the user-visible change
  (for example, "Add Sky Fighter: Raiden-style vertical shoot-em-up game").
