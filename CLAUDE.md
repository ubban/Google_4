# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free front-end project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5 canvas page.

| File        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: inline CSS, markup (HUD, overlays, mobile controls), and one inline `<script>` IIFE containing all game logic. |
| `README.md`  | Minimal placeholder (repo title only). Safe to expand if asked.          |

There is no build system, package manager, test suite, or linter configured. Nothing needs installing.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- For a quick manual check in the cloud environment, use the pre-installed Playwright/Chromium: `require('playwright').chromium.launch()` with no extra options.

## Code structure of `index.html`

- **Constants (top of script, ~lines 460–520):** `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and the base canvas size `BASE_W = 480`, `BASE_H = 720`.
- **Persistence:** best score, mute, difficulty, and reduced-motion are stored in `localStorage` under the `skyfighter.*` keys. Always wrap storage access so the game still works when storage is unavailable (private mode, blocked cookies).
- **Audio:** synthesized with the Web Audio API (`beep`, `noiseBurst`, `ensureAudio`). No audio files are shipped. Audio must be initialized from a user gesture.
- **Rendering:** `drawPlane`, `drawBossShip`, plus parallax cloud layers and the sun glow, all drawn on the single `#gameCanvas`. `resize()` scales the canvas to fit the viewport.
- **Simulation:** `update(dt)` is the per-frame step (movement, firing, spawning, collisions). `resetGame()` restores initial state. Boss attacks live in `BOSS_PATTERNS`.
- **Input:** keyboard (arrows/WASD move, Space fire, X/Shift bomb, P/Esc pause), mouse on the canvas, and touch (left-side virtual joystick, FIRE and BOMB buttons). Device motion is also supported via a toggle button.
- **UI feedback:** `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `flashHit`, and `announce` (screen-reader live region).

## Conventions

- Keep the game **self-contained in `index.html`** unless the user asks to split it. Do not introduce a bundler, framework, or external CDN dependency without being asked.
- Match the existing style: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), small named functions, and comments only where the intent is not obvious.
- Game tuning values belong in the constants block or the `DIFFICULTIES` table, not scattered as magic numbers.
- Preserve mobile support: the layout uses `touch-action: none`, safe-area insets, and `viewport-fit=cover`. Test any HUD or control change at phone width.
- Respect the `reducedMotion` setting for any new screen-shake, flash, or particle effect.

## Working in this repo

- Default development branch: `claude/gifted-tesla-fducgl`. Commit and push there unless told otherwise.
- Do not create pull requests unless explicitly asked.
- Before committing, check `git status` and do not add anything secret-like.
- Commit messages in this repo are descriptive, with a short summary line followed by bullet points for details.

## Verification checklist

Since there are no automated tests, verify changes by:

1. Loading `index.html` in a headless browser and confirming there are no console errors.
2. Starting a game, moving, firing, using a bomb, and pausing/resuming.
3. Checking the stage transition and boss fight still work after changes to `update()` or `BOSS_PATTERNS`.
4. Checking the layout at a phone-sized viewport if HUD or overlay markup changed.
