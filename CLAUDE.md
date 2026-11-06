# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

The entire game is a single self-contained file, `index.html` (~1,670 lines). It contains the HTML, inline CSS, and one inline `<script>` block that runs the game in a `<canvas>`. There is no build step, no package manager, no bundler, and no test suite.

Other files:
- `README.md`: a one-line title. Keep it short unless asked to expand it.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- There is nothing to install or compile.

## Repository layout

```
index.html   # Entire game: markup, CSS (<style>), and logic (<script>)
README.md    # Project title
CLAUDE.md    # This file
```

Inside `index.html`, the layout is roughly:
- `<style>`: dark-theme CSS, HUD, overlay/menu, touch controls, responsive sizing via `clamp()` and `env(safe-area-inset-*)`.
- `<body>`: `#container` holding `#gameCanvas`, the HUD (`#hud`, `#hearts`, `#bombs`, `#mult`, `#weaponLvl`), boss bar (`#bossBarWrap`), banners (`#comboPop`, `#stageBanner`), top buttons (`#muteBtn`, `#motionBtn`, `#pauseBtn`), touch controls (`#moveZone`, `#fireBtn`, `#bombBtn`), and the start/overlay card (`#overlay`, `#startBtn`, `#diffRow`).
- `<script>` (single IIFE): constants → audio (WebAudio `beep` / `noiseBurst`) → input → game state (`resetGame`) → spawning → `update(dt)` → boss patterns → `draw()` → `loop(now)` driven by `requestAnimationFrame`.

## Key game data (edit here to tune gameplay)

All tuning constants live near the top of the script:
- `DIFFICULTIES`: `easy` / `normal` / `hard` — spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
- `KILLS_PER_STAGE` (18): kills needed to advance a stage.
- `THEMES`: per-stage sky gradient, cloud/sun colors, and names (cycles per stage).
- `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+).
- `BOSSES`: six bosses (`VANGUARD` → `TITAN`), each with colors and `hpMul`; attack patterns are in `BOSS_PATTERNS`.

Persistent settings use `localStorage` with the `skyfighter.*` prefix (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Read and write these defensively, since storage can be unavailable.

## Controls

- **Keyboard:** Arrow keys / WASD to move; Space to fire (hold); `X` or Shift to drop a bomb; `P` or Escape to pause.
- **Mouse:** move over the canvas to steer; hold the mouse button to fire.
- **Touch:** virtual joystick on the left (`#moveZone`), FIRE and BOMB buttons on the right.

## Conventions

- **Keep it a single file.** Do not split `index.html` into modules or add a build step unless the user asks. The no-dependency, open-and-play design is intentional.
- **No external dependencies** unless explicitly requested. Everything (sound, drawing, input) uses browser built-ins.
- **Match the existing style:** 2-space indentation, single quotes in JS, `const`/`let`, arrow functions for handlers, and `UPPER_SNAKE_CASE` for tuning constants.
- **Preserve accessibility hooks:** the `#srLive` ARIA live region (`announce()`), the reduced-motion toggle (screen shake / flash / pulse), and `aria-label`/`title` on icon buttons.
- **Preserve mobile support:** keep `touch-action`, safe-area insets, and responsive canvas sizing (`BASE_W` 480 × `BASE_H` 720, scaled by `resize()`) working.
- Use `dt` (seconds, clamped to 0.05) for any time-based movement so the game stays frame-rate independent.

## Verifying changes

There are no automated tests or linters configured. To verify a change:
1. Open `index.html` in a browser (or serve it locally) and check the console for errors.
2. Play through start, pause/resume, a stage transition, a boss fight, and game over; confirm the HUD and best score update.
3. Check the touch layout at phone width if the change touches layout or controls.

If you add a test or lint setup, document the commands here.

## Git

- Default development branch for this work: `claude/gifted-tesla-0ssdfb`.
- Write clear, descriptive commit messages in the imperative mood.
- Do not open pull requests unless explicitly asked.
