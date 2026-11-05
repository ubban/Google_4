# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game lives in one file, `index.html`: inline CSS (`<style>`), markup, and a single IIFE `<script>` that draws everything on a `<canvas id="gameCanvas">` with the 2D context. `README.md` is a placeholder and carries no project documentation.

There is no build step, no package manager, no bundler, and no test suite. The repo has no `package.json`.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests or linters configured. Verify changes by playing the game in a browser, and check both desktop (keyboard and mouse) and touch layouts when touching HUD, overlay, or control code.
- Preview with a headless browser when useful. Capture screenshots at a sensible size; the project has no image assets to compare against.

## Code layout inside `index.html`

- `<head>`: meta tags, an inline SVG favicon, and the stylesheet. Colors and sizes are hard-coded per element; there are no CSS custom properties beyond `color-scheme: dark`.
- `<body>` markup: `#container` wraps the canvas, `#hud` (score, best, multiplier, weapon, hearts, bombs), `#vignette` and `#flash` overlays, `#overlay` (title card, difficulty buttons, start button, stats row), `#touchControls` (joystick, FIRE, BOMB), and the icon buttons `#muteBtn`, `#motionBtn`, `#pauseBtn`.
- Script: all state and logic sit inside one IIFE. Keep new code inside that scope rather than adding globals.

Key constants and tables in the script:

- `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, enemy shot chance, starting lives and bombs, and a boss HP multiplier.
- `KILLS_PER_STAGE` (18): kills needed to advance to the next stage and its boss.
- `THEMES`: six sky palettes, cycled per stage, each with cloud and sun colors and a display name.
- `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), and `tank` (multi-hit). `tank` unlocks at stage 2 via `minStage`.
- `BOSSES`: six bosses, one per stage, each with its own colors and `hpMul`.

Persistence uses `localStorage` under the `skyfighter.*` prefix: `bestScore`, `muted`, `difficulty`, and `reducedMotion`. Read and write these through the existing `STORAGE_*` constants.

## Controls (keep the in-game hint in sync)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift to use a bomb, P or Escape to pause.
- Mouse: move to steer, click to fire.
- Touch: drag in the left zone to fly, tap FIRE, tap BOMB.

The hint text lives in `#controlsHint`. Update it whenever a control changes.

## Conventions

- Plain HTML, CSS, and JavaScript only. Do not add frameworks, npm dependencies, or external network calls unless the user asks for them.
- Keep the file self-contained and openable from `file://`.
- Respect the existing reduced-motion and mute settings when adding effects or sounds. Audio goes through the shared `ensureAudio()` context and the `beep` / `noiseBurst` helpers.
- Keep the HUD and overlay usable at phone widths (`viewport-fit=cover`, safe-area insets, `clamp()` font sizes). Do not introduce horizontal page scroll.
- Performance: the game tracks frame times and lowers `quality` when frames run slow. New per-frame work should respect that path.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in JS, and short comments only where the intent is not obvious.

## Git

- Develop and push on the branch the session names (currently `claude/gifted-tesla-dhv0jb`). Do not push to other branches without explicit permission.
- Write descriptive commit messages that explain the gameplay or UI change, not just the file touched.
- Do not open a pull request unless the user asks for one.
