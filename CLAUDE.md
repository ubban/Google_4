# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-page, Raiden-style vertical shoot-'em-up built with HTML5 Canvas and vanilla JavaScript. There is no build step, no package manager, and no test suite.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html`| The entire game: markup (`<body>`), CSS (`<style>`), and JS (`<script>`), about 1,670 lines |
| `README.md` | Minimal placeholder                                            |

To run the game, open `index.html` in a browser. No server is required.

## Code layout inside `index.html`

- **`<head>`**: meta tags (viewport locked to phone-friendly settings, theme color), an inline SVG favicon, and the page `<title>`.
- **`<style>` (about lines 10–360)**: all styling. HUD pills, glassmorphic overlay card, touch controls, boss health bar, and keyframe animations.
- **`<body>` (about lines 361–421)**: a `<canvas id="gameCanvas">`, HUD elements, overlay (title, difficulty buttons, stats), touch controls, and a live region (`srLive`) for screen readers.
- **`<script>` (about lines 422 onward)**, in roughly this order:
  1. DOM element lookups (`const xxxEl = document.getElementById(...)`).
  2. Storage keys (`STORAGE_*`) and persisted settings (best score, mute, difficulty, reduced motion).
  3. Game data tables: `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
  4. Audio: lazily created `AudioContext`, `beep()`, `noiseBurst()`, and the `sfx` map. All sound is synthesized; there are no audio files.
  5. HUD and effects helpers: hearts/bombs rendering, flash, combo popup, stage banner, screen shake.
  6. Input: keyboard (`keys`), a virtual joystick and fire/bomb buttons for touch, and mouse targeting.
  7. Game state: `resetGame()` and module-level `let` variables (`player`, `bullets`, `enemies`, `boss`, `score`, `stage`, and so on).
  8. Drawing: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
  9. Simulation: `update(dt)`, `updateBoss(dt)`, and spawning (`spawnEnemy`).
  10. Main loop: `loop(now)` calls `update` and `draw` each frame via `requestAnimationFrame`.

Everything lives in one global-scope script. Helper functions and state are declared with `const`/`let`/`function` at the top level of that script, so keep new code in the same style unless there is a clear reason to split it out.

## Game mechanics to keep in mind

- **Stages**: each stage ends after `KILLS_PER_STAGE` (18) kills and then spawns a boss. Stage number also selects the `THEMES` entry (modulo the theme count) and gates enemy types through `minStage`.
- **Enemies**: `grunt`, `interceptor` (fast, weaves), and `tank` (high HP, from stage 2).
- **Bosses**: `BOSSES` lists six designs with `hpMul` scaling. A boss telegraphs attacks before firing, and its health bar is shown in the HUD.
- **Weapons**: Vulcan and Laser, each with three power levels. Weapon power-ups are type `'weapon'`.
- **Power-ups**: `'life'`, `'bomb'`, and `'weapon'`, handled in `applyPowerUp()`.
- **Bombs**: limited per life, clear the screen, and damage bosses. Triggered by `X` or Shift, or by the touch bomb button.
- **Combo multiplier**: kills in quick succession build a chain, which raises the score multiplier.
- **Difficulty**: `easy`, `normal`, and `hard`, set in `DIFFICULTIES` (spawn rate, enemy speed, enemy fire chance, lives, bombs, boss HP). The choice persists in `localStorage`.
- **Adaptive quality**: `trackPerf()` averages the last 45 frame times. If the average is slower than about 40 fps, `quality` increases (0 full, 1 medium, 2 low), which reduces effects. If it is faster than about 56 fps, it steps back down.

## Controls

- **Keyboard**: arrow keys or WASD to move, Space to fire, `X` or Shift for a bomb, `P` or Escape to pause.
- **Touch**: drag on the left side to fly with the virtual joystick, tap FIRE, tap BOMB. Shown on touch devices.
- **Mouse**: move the pointer to steer the ship, and click or hold to fire. The in-game hint at `index.html:417` lists these.

## Persistence

These keys are stored in `localStorage` and read through guarded accessors. Keep the names stable, because changing them resets players' saved state:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

## Accessibility and motion

- Respect `prefers-reduced-motion`. When `reducedMotion` is on, screen shake is strongly reduced and hit/bomb flashes are skipped. The initial value comes from the saved choice if one exists, otherwise from the system preference. Users toggle it with the motion button.
- Screen-reader announcements go through `announce(text)`, which writes to the `srLive` region. Use it for important state changes such as stage start, boss warnings, pause, and game over.
- Controls need labels. Buttons and the canvas wrapper should keep `aria-*` attributes and visible focus styles.

## Conventions

- Keep it a single self-contained HTML file with no external dependencies unless the user asks for one. Don't add a bundler, framework, or package manager without discussion.
- Use `const` for data tables and constants, and `let` for mutable game state. Constants are UPPER_SNAKE_CASE for tables such as `DIFFICULTIES` and `STORAGE_*`.
- Keep the visual style consistent with the existing palette: blues for the HUD and sky, and per-type colors for enemies and bosses.
- Scale-independent: the canvas uses `BASE_W` × `BASE_H` (480 × 720) and `resize()` scales it to fit. Write gameplay coordinates in base units, not CSS pixels.
- Time-based movement uses the `dt` passed to `update(dt)`. Don't hardcode per-frame speeds where `dt` is available.
- Guard all `localStorage` and `AudioContext` access in `try`/`catch` or existence checks, since both can be unavailable (private windows, autoplay policies).

## Testing and verification

There is no automated test suite. Verify changes by:

1. Opening `index.html` in a browser and checking the console for errors.
2. Playing through at least one stage transition and one boss fight after gameplay changes.
3. Checking the layout at phone width (touch controls visible, no horizontal scroll) after any CSS or HUD change.
4. Testing with reduced motion enabled and with audio muted.

If you add a test harness, document how to run it here.

## Git workflow

- The default working branch for this repo is set per task. Commit with clear, descriptive messages that say what changed and why.
- Keep commits focused. Gameplay tuning, visual polish, and bug fixes are good candidates for separate commits.
- Don't open a pull request unless the user asks for one.
