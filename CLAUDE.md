# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single self-contained HTML5 canvas page.

- `index.html` — the whole game: markup, CSS (`<style>`, ~lines 10–361), and JavaScript (`<script>`, ~line 422 to end). There is no build step, bundler, package manager, or test suite.
- `README.md` — minimal placeholder; not a source of truth for behavior.

To run the game, open `index.html` in a browser (or serve the directory with any static file server). Nothing needs to be installed.

## Repository layout and history

- History is two commits: an initial commit, then `Add Sky Fighter: ...`, which introduced the game.
- Development branch for this task: `claude/gifted-tesla-gsw205`.

## Architecture of `index.html`

The script is one IIFE (`(function () { 'use strict'; ... })()`). Everything is module-private; nothing is exposed on `window`.

Key sections, in file order:

1. **DOM references** — elements are looked up once by id at the top (`gameCanvas`, `overlay`, HUD elements such as `score`, `hearts`, `bombs`, `weaponName`, `bossBarFill`, and the touch controls).
2. **Persistence keys** — `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. All reads and writes go directly to `localStorage`; keep it that way unless adding a key here too.
3. **Adaptive quality** — `trackPerf` samples the last 45 frame times and steps `quality` (0 full, 1 medium, 2 low) to keep frame rate up.
4. **Config tables** — `DIFFICULTIES` (spawn rates, enemy speed, shot chance, starting lives/bombs, boss HP multiplier), `THEMES` (six sky palettes that cycle per stage), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`, with `minStage` gating), `BOSSES` (six named bosses cycling by stage), `KILLS_PER_STAGE` (18).
5. **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` primitives and a `sfx` object built on them. Audio is unlocked on the first user gesture.
6. **HUD and effects** — `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, `flashBomb`, and `announce` (writes to an `aria-live` region for screen readers).
7. **Input** — keyboard (`keys` map keyed by `e.code`), mouse (canvas position follows the pointer), and touch (virtual joystick in `moveZone`, plus fire and bomb buttons). The fire control is Space or the fire button; bomb is X or Shift; pause is P or Escape.
8. **Game state and loop** — `resetGame`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`. Update and draw are separate; `update` handles movement, spawning, collisions, scoring and stage progression, and `draw` renders everything from state.
9. **Bosses** — `BOSS_PATTERNS` is an array of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring). `updateBoss` picks patterns; `killBoss` and `endGame` close out a run.

### Game rules worth knowing

- Lives come from difficulty (`easy` 4, `normal` 3, `hard` 3). Bombs clear the screen and damage bosses.
- Score is multiplied by a kill-chain multiplier (`breakChain`, up to ×10).
- Stages advance after `KILLS_PER_STAGE` kills; the boss appears at the end of each stage.
- Power-ups (`applyPowerUp`) raise weapon level (Vulcan/Laser, 3 levels), grant bombs, or add an extra life.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce a build system, frameworks, or npm dependencies without an explicit request.
- **No external network dependencies.** The page currently loads nothing remote. Keep it that way.
- **Vanilla JS, ES2017+ style** consistent with the existing code: `const`/`let`, arrow functions, template literals, `'use strict'`.
- **Match the existing style:** two-space indentation, single quotes in JS, short section headers using `// ---------- Name ----------`.
- **Game tuning lives in the config tables.** Prefer editing `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, or `KILLS_PER_STAGE` over hard-coding numbers in logic.
- **Respect reduced motion.** Anything that shakes or flashes must honor `reducedMotion` (see `screenShake`).
- **Accessibility:** announce important state changes through `announce()` and keep controls keyboard- and touch-reachable.
- **Mobile:** the layout scales to the viewport via `resize()` (`BASE_W` 480 × `BASE_H` 720). Test changes at phone widths as well as desktop.

## Verifying changes

There are no automated tests or linters. Verify by:

1. Opening `index.html` in a browser (or serving the directory) and checking the console for errors.
2. Playing through start, a stage clear, a boss fight, pause/resume, bomb use, and game over.
3. Checking persistence: best score, mute, difficulty, and reduced-motion survive a page reload.
4. Checking touch controls with a touch-emulated viewport for layout and input changes.

If a change cannot be exercised in a browser, say so explicitly rather than claiming it works.

## Git workflow

- Develop on `claude/gifted-tesla-gsw205` and push there with `git push -u origin claude/gifted-tesla-gsw205`.
- Do not push to other branches without explicit permission.
- Do not open a pull request unless explicitly asked.
- Keep commits focused. Describe the gameplay or UI effect in the commit message, not just the code change.
