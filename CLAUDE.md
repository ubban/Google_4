# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-em-up built as a single self-contained HTML5 canvas page.

| File        | Purpose                                                            |
|-------------|--------------------------------------------------------------------|
| `index.html`| The whole game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file (~1,670 lines). |
| `README.md` | Minimal project readme (title only).                               |

There is no build system, package manager, test suite, linter, or server-side code.
Nothing needs to be installed.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server
  (e.g. `python3 -m http.server`) and visit `http://localhost:8000`.
- Use the Playwright/Chromium setup in the environment for any automated check; do not
  install Playwright again.

## Code structure of `index.html`

- **Markup**: a `<canvas id="gameCanvas">` plus HUD elements (score, best, multiplier,
  weapon, hearts, bombs, boss bar, combo/stage banners, pause/mute/motion buttons,
  touch controls) and an overlay card for the title, game-over, and stats.
- **CSS**: one `<style>` block near the top. Keep visual changes here rather than
  inline styles.
- **JavaScript**: one IIFE (`(function () { 'use strict'; ... })();`) with no globals.
  Keep it that way: new code should live inside the IIFE.
  Major sections, in order of appearance:
  - DOM references and `STORAGE_*` keys
  - Adaptive performance (`quality` level driven by frame timing)
  - Config tables: `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`
  - Audio (`ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX`), created lazily on first user gesture
  - HUD helpers (hearts, bombs, combo pops, stage banners, screen shake)
  - Input (keyboard, mouse, touch joystick and buttons)
  - Game state (`resetGame`, `update`, `draw`) and entities (enemies, bosses, power-ups, bullets)
  - Boss attack patterns (`BOSS_PATTERNS`, `fire*` helpers) and boss lifecycle (`updateBoss`, `killBoss`)
  - Lives, game over, pause, and the main `loop(now)` driven by `requestAnimationFrame`

## Game rules and tuning (where to change them)

- **Difficulty** (`DIFFICULTIES`): spawn rate, enemy speed, enemy fire chance, starting
  lives and bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
- **Stages**: `KILLS_PER_STAGE` kills advance the stage; stage number selects the theme
  and boss (`BOSSES` cycles through six bosses).
- **Weapons**: `fireWeapon` implements Vulcan and Laser with power levels 1–3; the
  weapon-level power-up is applied in `applyPowerUp`.
- **Scoring**: `addScore` applies a kill-chain multiplier that `breakChain` resets.
- **Persistence** (`localStorage`, all keys prefixed `skyfighter.`):
  `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are wrapped
  defensively; keep them that way, since storage can be unavailable (private mode).

## Controls (keep in sync with `controlsHint` in the overlay)

- Keyboard: Arrows / WASD move, Space fires, X or Shift uses a bomb, P or Esc pauses.
- Mouse: move to steer, click to fire.
- Touch: drag in the left zone for the virtual joystick, tap FIRE, tap BOMB.
- Input is read from `keys[e.code]` for movement and from event handlers for actions.

## Conventions

- Plain ES2015+ JavaScript, no frameworks, no modules, no external dependencies. Do not
  add a bundler, package.json, or CDN script unless the user asks for it.
- Keep it a single-file page unless the user asks to split it. If you do split it, keep the
  paths relative and update this document.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let`
  (no `var`), arrow functions for callbacks, and short comment banners
  (`// ---------- Section ----------`) between sections.
- Prefer small, targeted edits with the `Edit` tool over rewriting the file. Diffs
  against `index.html` are hard to review when large blocks move.
- Gameplay numbers are tuned by hand. When changing one, state the old and new value
  in the commit message or summary.
- Accessibility features matter: keep the `srLive` announcements, the reduced-motion
  toggle, and the `prefers-reduced-motion` check working when touching UI or effects.
- Performance matters: the loop caps `dt` at 50 ms and adapts quality. Avoid per-frame
  allocations in hot paths (`update`, `draw`, particle code) where practical.

## Verifying changes

There are no automated tests. Before reporting a gameplay or UI change as done:

1. Open the page in headless Chromium (Playwright, `executablePath:
   '/opt/pw-browsers/chromium'` if needed) and confirm there are no console errors.
2. Start a game, play briefly, and exercise the code path you changed (for example, a
   boss pattern, a power-up, or pause/resume).
3. Check the layout at phone width (the page has touch controls) if you touched CSS or HUD.

If you could not run the game, say so explicitly rather than claiming it works.

## Git and workflow

- Default working branch for this session: `claude/gifted-tesla-po0jli`. Push only to
  that branch unless the user says otherwise.
- Commit messages: short imperative summary line, then a body explaining the gameplay or
  code change. Match the style of the existing history.
- Do not create pull requests unless the user explicitly asks.
