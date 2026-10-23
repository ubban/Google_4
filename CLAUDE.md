# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game. The entire game lives in one file, `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>` wrapped in an IIFE).

- `README.md` — project title only.
- `index.html` — the whole game. Nothing else.

There is no build step, no package manager, no bundler, no framework, no test suite, and no CI configuration. Do not introduce any of these unless the user asks for them.

## Running the game

Open `index.html` directly in a browser, or serve the directory statically, for example:

```sh
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Architecture of `index.html`

The file has three sections, in order:

1. **`<style>`** — all CSS. Colors and layout are inline; there are no external stylesheets or fonts. The page uses `color-scheme: dark`. Reduced-motion styling is driven by a `body.reduced-motion` class.
2. **Markup** — a `<canvas id="gameCanvas">` plus HUD (`#hud`), boss bar (`#bossBarWrap`), top buttons (`#topBar`: mute, reduced-motion, pause), touch controls (`#touchControls`), and the start/pause/game-over overlay (`#overlay`).
3. **`<script>`** — a single `(function () { 'use strict'; ... })();` block. Nothing is exposed on `window`. Keep new code inside this IIFE.

Within the script, code is grouped by `// ---------- Section ----------` comments in this order: adaptive performance, constants, audio, HUD helpers, input, game state, entities and spawning, update logic, boss patterns, draw, and the main loop. Follow the same grouping when adding code.

### Key constants (search for these names)

| Constant | Purpose |
| --- | --- |
| `BASE_W`, `BASE_H` (480 × 720) | Logical canvas size. The canvas is scaled to fit the window (capped at 1.6×). |
| `DIFFICULTIES` | Per-difficulty spawn timing, enemy speed, shoot chance, starting lives and bombs, and boss HP multiplier. |
| `KILLS_PER_STAGE` | Kills needed to advance a stage. |
| `THEMES` | Sky palettes; the theme cycles with stage. |
| `ENEMY_TYPES` | `grunt`, `interceptor` (weaves), and `tank` (multi-hit). Each has size, HP, speed, score, and `minStage`. |
| `BOSSES` | Boss names and colors, with an HP multiplier. Patterns live in `BOSS_PATTERNS`. |

### Game loop and state

- `loop(now)` is the `requestAnimationFrame` driver. It caps `dt` at 0.05 s, calls `trackPerf`, then runs `update(dt)` and `draw()` only when the game is running and not paused.
- Game state is a set of module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, and so on). `resetGame()` reinitializes all of them; any new state must be reset there too.
- `quality` (0 full, 1 medium, 2 low) is set automatically by `trackPerf` from average frame time. Expensive effects (shadow blur, glow, particle counts) should branch on `quality` rather than hard-coding.
- Times and speeds are in seconds and pixels per second, scaled by `dt`. Do not use frame counts for gameplay timing; `frame` exists only for patterns that need a tick counter.

### Persistence

`localStorage` keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore` — written only in `endGame()`, when the score beats the stored best.
- `skyfighter.muted` — `'1'` or `'0'`.
- `skyfighter.difficulty` — `'easy'`, `'normal'`, or `'hard'`.
- `skyfighter.reducedMotion` — `'1'` or `'0'`. If absent, the `prefers-reduced-motion` media query decides the default.

Reads and writes are not wrapped in try/catch. If you add storage access, consider that it can throw in private or restricted browsing contexts.

### Input

- Keyboard: arrows or WASD to move; Space fires (held); X or Shift triggers a bomb; P or Escape pauses.
- Mouse: moving the cursor over the canvas steers the ship toward it; mousedown fires.
- Touch: a left-side joystick zone (`#moveZone`), a FIRE button, and a BOMB button. Touch controls appear only when `isTouchDevice` is true.
- Keyboard state is tracked in the `keys` object, keyed by `KeyboardEvent.code`.

### Accessibility

- `announce(text)` writes to the `#srLive` live region. Use it for important state changes such as game over, new best, or lives remaining.
- Reduced motion disables screen shake, flashes, and pulses. Any new motion effect must respect `reducedMotion`.
- Keep `aria` labels and `title` attributes on icon-only buttons.

### Audio

Audio is synthesized with the Web Audio API (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object). No audio files are loaded. `ensureAudio()` must run from a user gesture; the existing handlers already call it.

## Conventions

- **Vanilla JavaScript only.** No dependencies, no modules, no transpilation. Match the existing style: `const`/`let`, arrow functions for callbacks, 2-space indentation, single quotes, and semicolons.
- **Keep it in one file** unless the user asks for a split. If a split is requested, update this file.
- **Comments** are short and used sparingly, mainly as section headers and to explain non-obvious numbers. Match the density already in the file.
- **Don't add build tooling, test frameworks, or new config files** without an explicit request.
- **Preserve the mobile layout.** The page is `overflow: hidden` and scales the canvas to the viewport; check changes to HUD or overlay CSS at narrow widths.

## Verification

No automated tests exist. To verify a change:

1. Load the game in a browser (see "Running the game") and check the browser console for errors.
2. Play through at least one stage transition and, if touched, the boss fight (`BOSSES`, `BOSS_PATTERNS`, `updateBoss`).
3. Check that pause (P/Esc), mute, difficulty switching, and reduced-motion toggling still work.
4. For HUD or overlay changes, check both a narrow (phone-sized) and a desktop viewport.
5. Confirm best-score persistence by reloading the page after a game over.

Report in the final message what was actually exercised and what was not.

## Git

- Development branch for this work: `claude/gifted-tesla-r2ie2t`.
- Commit messages describe the change in the imperative mood, with a short summary line and an optional body explaining why.
- Do not open a pull request unless the user asks for one.
