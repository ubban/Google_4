# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- The entire game lives in a single file: `index.html` (inline `<style>` and one inline `<script>`, drawn on a `<canvas>`).
- There is no build step, no package manager, no bundler, no test suite, and no external assets. Audio is synthesized with WebAudio.
- `README.md` is a placeholder; treat `index.html` as the source of truth.

To run it, open `index.html` in a browser (or serve the directory with any static file server).

## Repository layout

```
index.html   # The whole game: markup, CSS, and JS
README.md    # Minimal placeholder
CLAUDE.md    # This file
```

## Architecture (inside `index.html`)

The script is organized into sections marked with `// ---------- Name ----------` comments:

- **Storage keys** (`STORAGE_*`): `localStorage` keys for best score, mute, difficulty, and reduced motion.
- **Adaptive performance** (`trackPerf`): frame-time tracking used to adjust effects.
- **Config tables**: `DIFFICULTIES` (easy / normal / hard: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE` (18), `THEMES` (six stage sky palettes, cycled per stage), `ENEMY_TYPES`, and `BOSSES`.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and per-enemy explosion sounds. Audio is created lazily on first user gesture.
- **HUD / feedback**: hearts, bombs, flash effects, combo pop-ups, stage banner, screen shake.
- **Canvas sizing**: fixed logical size `BASE_W = 480`, `BASE_H = 720`, scaled to fit via `resize()`.
- **Input**: keyboard `keys` map, mouse/touch steering, and a touch joystick plus on-screen FIRE/BOMB buttons.
- **Game state**: `resetGame`, `update(dt)` (main simulation step), `loseLife`, `endGame`, `togglePause`.
- **Weapons and bosses**: `fireWeapon` (vulcan and other weapons with power-up levels), `useBomb`, boss attack patterns (`BOSS_PATTERNS`, `fire*` helpers), `updateBoss`, `killBoss`.
- **Rendering**: `draw()` and helper draw routines (`drawPlane`, `drawBossShip`, `drawClouds`).
- **Main loop**: `loop(now)` drives `update` and `draw` with a delta-time step.

## Controls (keep the in-game hint in sync)

- Keyboard: Arrow keys / WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses.
- Mouse: move to steer, hold click to fire.
- Touch: drag in the left zone to fly, tap FIRE, tap BOMB.

The on-screen controls hint lives in `#controlsHint` in the markup. If you change a binding, update that text too.

## Working conventions

- Keep the game self-contained. Do not add a build system, framework, or npm dependency unless the user asks for one.
- Prefer small, targeted edits to the existing functions over restructuring the file.
- Preserve the existing style: `const`/`let`, arrow callbacks, two-space indentation, section comment banners.
- Keep the game playable on phone-sized viewports (`touch-action: none`, safe-area insets, responsive HUD clamps). Test layout changes at narrow widths.
- Respect the reduced-motion setting when adding new screen shake, flashes, or animations.
- Persisted values go through the `STORAGE_*` keys and should be wrapped in try/catch, since `localStorage` can be unavailable.
- Use the WebAudio synth for any new sound effects; do not add audio files.

## Verifying changes

There is no automated test suite. To verify a change:

1. Open `index.html` in a desktop browser and play through at least one stage, including a boss fight, a bomb, a pause/resume, and a life loss.
2. Check the HUD and stage banner at a narrow (phone) viewport, for example with browser devtools device emulation.
3. Confirm the console shows no errors on load or during play.

## Git

- Development branch for this work: `claude/gifted-tesla-nes0i2`.
- Write clear, descriptive commit messages that explain the why.
- Do not open pull requests unless the user explicitly asks.
