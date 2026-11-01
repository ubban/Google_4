# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up. The entire game lives in a single file, `index.html`
(~1,670 lines: inline CSS, markup, and one IIFE-wrapped `<script>`). `README.md` is a
placeholder with no setup instructions.

There is no build step, package manager, test suite, linter, or CI config.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server
(for example `python3 -m http.server`). Nothing needs to be installed.

## Repository layout

```
index.html   # the whole game: <style>, <body> overlay/HUD/touch markup, <script>
README.md    # placeholder
```

Git history: `Initial commit`, then `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.
Development happens on feature branches; do not push to `main` without being asked.

## Architecture of `index.html`

The script is one IIFE. Major sections, roughly in file order:

- **DOM refs and storage keys** (~L426–466). Storage keys are prefixed `skyfighter.`:
  `bestScore`, `muted`, `difficulty`, `reducedMotion`.
- **Perf governor** (`trackPerf`, `quality` 0/1/2). Lowers particle counts and effect
  cost when frame time is high.
- **Tuning tables**
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed and fire chance,
    starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE = 18`.
  - `THEMES`: six sky palettes, cycled per stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+).
  - `BOSSES`: six bosses (VANGUARD → TITAN), cycled by `(stage - 1) % 6`, with HP
    scaled by `baseHp = 70 + stage * 35`.
- **Audio**: lazily created `AudioContext` (`ensureAudio`), synthesized with
  `beep` / `noiseBurst`. The `sfx` map holds named sound effects. `muted` persists.
- **Rendering helpers**: `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`,
  `screenShake`, `showStageBanner`, `showComboPop`. The canvas is fixed at
  `BASE_W × BASE_H = 480 × 720` and scaled by `resize()` (max 1.6×).
- **Input** (`keys`, touch joystick, fire and bomb buttons, mouse on canvas). Details in
  the Controls section below.
- **Game state** (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`,
  `powerUps`, `boss`, plus `score`, `lives`, `bombCount`, `stage`, `chain`,
  `multiplier`, and others). `resetGame()` initializes it.
- **Weapons**: `fireWeapon()` with `vulcan` and `laser` types and levels 1–3.
- **Boss attacks**: `BOSS_PATTERNS` and `fire*` helpers (radial, aimed spread,
  columns, spiral, cross, homing, double ring). `updateBoss` drives phases and
  telegraphs.
- **Main loop**: `update(dt)` → `draw()` via `requestAnimationFrame`. `dt` is clamped to
  50 ms. Pause is handled by `togglePause()`.
- **Persistence and end of run**: `bestScore` is written to `localStorage` on game over.

## Controls

| Action        | Keyboard                  | Touch / mouse                       |
|---------------|---------------------------|-------------------------------------|
| Move          | Arrow keys / WASD         | Virtual joystick / mouse on canvas  |
| Fire          | Space (hold)              | FIRE button (hold)                  |
| Bomb          | X or Shift                | BOMB button                         |
| Pause         | P or Escape               | Pause button                        |
| Mute          | Mute button               | Mute button                         |
| Reduced motion| Motion button (🎆)        | Motion button (🎆)                  |

Reduced motion defaults to the `prefers-reduced-motion` media query until the user
toggles it. It turns off screen shake, flash, and pulse effects.

## Conventions

- **Keep it one file.** The project is intentionally a single self-contained HTML file.
  Do not add a bundler, package.json, or external assets unless the user asks for that.
- **No external network calls.** The page loads nothing remote. The favicon is an inline
  SVG data URI. Keep it that way.
- **Preserve the mobile layout.** The viewport meta disables zoom, touch controls are
  always present on touch devices, and `touch-action: none` is set. Check touch changes
  at phone widths.
- **Use the existing tuning tables** when changing balance. Prefer editing
  `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `THEMES` over hardcoding numbers in
  `update()`.
- **Storage access is best-effort.** Wrap new `localStorage` reads and writes in
  `try/catch`, since storage can be blocked in private windows. Existing calls are not
  wrapped yet.
- **Accessibility:** keep the `srLive` announcements (`announce()`) for state changes.
  Current announcements cover reduced-motion toggles, boss defeat and stage entry, lives
  remaining, and game over.

## Verifying changes

There are no automated tests. For a change:

1. Open `index.html` in a browser (or serve the directory) and check for console errors.
2. Play through at least one full stage boundary (18 kills) and a boss fight to exercise
   spawn, boss, and stage transition code.
3. Check pause, mute, reduced-motion, and difficulty toggles, and that the best score
   persists across reloads.
4. If the change touches layout or input, check it at a phone-sized viewport.

A quick syntax check of the inline script is also useful:

```bash
python3 -c "
import re, sys
html = open('index.html', encoding='utf-8').read()
open(sys.argv[1], 'w', encoding='utf-8').write(re.search(r'<script>(.*)</script>', html, re.S).group(1))
" "$SCRATCH/skyfighter_check.js" && node --check "$SCRATCH/skyfighter_check.js"
```

Set `SCRATCH` to a scratch directory outside the repo.

## Git

- Work on the designated feature branch. Commit with clear messages, then push with
  `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
