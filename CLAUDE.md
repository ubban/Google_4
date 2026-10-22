# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` holds **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page. There is no build step, no bundler, no package manifest, and no test suite.

Files:

- `index.html` (~1,670 lines): all markup, CSS (`<style>` at ~line 10), and game logic (one IIFE in `<script>` at ~line 422).
- `README.md`: a one-line title. Keep it short.

To run the game, open `index.html` in a browser, or serve the directory with any static server. Nothing needs to be installed.

## Verifying changes

There are no automated tests or linters configured. Verify by loading `index.html` in a browser and playing. Check the affected path, for example:

- Start, pause (P/Esc), and game over flows.
- Stage transitions and the boss fight, if you touched spawning, scoring, or boss patterns.
- Touch controls, if you touched input (test at phone width).
- Mute, reduced-motion, and difficulty toggles, if you touched them.

If you add a test or lint setup, document the command here.

## Code layout inside `index.html`

The script is one `(function () { 'use strict'; ... })()` block. Keep new code inside it. Do not add globals.

Sections, in file order:

1. **DOM references**: `const xEl = document.getElementById(...)` at the top.
2. **Storage keys**: `STORAGE_*` constants. Persisted keys use the `skyfighter.` prefix.
3. **Adaptive performance**: `trackPerf` lowers the `quality` level (0 full, 1 medium, 2 low) when frame times are poor.
4. **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here, not in logic.
5. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` helpers. Audio is built with the Web Audio API and starts only after a user gesture.
6. **HUD and effects**: hearts, bombs, combo pops, stage banners, screen shake, and flashes.
7. **Input**: keyboard (`keys`), touch joystick and buttons, and mouse. Input maps to `firing`, `useBomb()`, and movement.
8. **Game state**: `resetGame()`, then `update(dt)` for the fixed per-frame simulation, and `draw()` for rendering.
9. **Boss system**: `BOSS_PATTERNS` (a list of bullet-pattern functions), `updateBoss`, `killBoss`.
10. **Flow**: `loseLife`, `endGame`, `togglePause`, and the `loop(now)` requestAnimationFrame loop.

Conventions observed in the code:

- Use `const` by default and `let` only for values that change.
- Use arrow functions for callbacks and `function` declarations for named helpers.
- Keep comments to section headers in the form `// ---------- Name ----------`, plus short notes where the logic is not obvious.
- Use the existing `sfx.*` helpers for sounds and `announce()` for screen-reader text. Do not create new audio contexts.
- Respect `reducedMotion` for any new shake, flash, or pulse effect.
- Keep the layout working at phone width (`BASE_W` 480 × `BASE_H` 720 is the logical play field, scaled by `resize()`).

## Persistence

`localStorage` holds four values, all with the `skyfighter.` prefix:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; falls back to the OS `prefers-reduced-motion` setting if unset |

Storage reads and writes are not wrapped in `try`/`catch`. If you add more persisted state, wrap it so the game still works when storage is blocked (private windows, disabled site data).

## Gameplay constants to know

- Lives and bombs per run depend on difficulty (`DIFFICULTIES`).
- Each stage ends after `KILLS_PER_STAGE` (18) kills and then spawns a boss. The boss is `BOSSES[(stage - 1) % BOSSES.length]` and the background theme is `THEMES[(stage - 1) % THEMES.length]`.
- Score is multiplied by a kill-chain multiplier (`addScore`, `breakChain`).
- Weapons are `'vulcan'` and `'laser'` (`player.weapon`). Power-ups change the weapon level and can toggle the weapon type (`applyPowerUp`). `fireWeapon` handles firing.

## Editing guidelines

- Make the smallest change that does the job. The file is one large script, so avoid restructuring it unless asked.
- Put tuning values in the existing tables rather than hard-coding them in logic.
- Do not add frameworks, a build step, or external dependencies unless the user asks. The project is intentionally a single file that works from `file://`.
- Do not change the storage key names. Existing players' saved data depends on them.
- Keep the `<meta name="viewport">` and touch-control markup intact unless the change is about mobile layout.

## Git

- Default development branch for this workspace: `claude/gifted-tesla-8y5v3x`.
- Commit with clear messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.
