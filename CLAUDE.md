# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a small, dependency-free browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. The entire game ships as a single file, `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>` IIFE). `README.md` is a stub.

There is no build step, package manager, test suite, or linter configured.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The whole game: HTML overlay/HUD, CSS, and JavaScript on a 2D `<canvas>`. |
| `README.md` | Minimal project title stub. |

## Running the game

No tooling is required. Open the file directly, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code structure of `index.html`

- **Markup** (`<body>`): `#container` holds `#gameCanvas` plus overlays (`#overlay`, `#hud`, `#touchControls`, `#vignette`, `#flash`, `#stageBanner`, `#comboPop`, `#bossBarWrap`).
- **Styles** (inline `<style>`): dark color scheme, full-viewport flex layout, `touch-action: none` to prevent page gestures during play.
- **Script** (inline `<script>`, wrapped in an IIFE with `'use strict'`). Major sections, in order:
  - DOM references and constants: `DIFFICULTIES` (easy/normal/hard tuning), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, name and color data), and `STORAGE_*` localStorage keys.
  - Audio: `ensureAudio()`, `beep()`, `noiseBurst()` on a lazily created `AudioContext`. Sound is synthesized, with no audio assets.
  - Input: keyboard (`keys` map), mouse on canvas, touch joystick (`joystick*`), and on-screen FIRE/BOMB buttons.
  - Game state and loop: `resetGame()`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`. `dt` is clamped to 50 ms.
  - Gameplay systems: enemy and boss spawning, weapon firing (`fireWeapon`), power-ups (`applyPowerUp`), combo and score (`addScore`, `breakChain`), bombs (`useBomb`), lives (`loseLife`), and game over (`endGame`).
  - Boss attack patterns: `BOSS_PATTERNS` and helpers such as `fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`.
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, plus HUD helpers (`renderHearts`, `renderBombs`, `showStageBanner`, `screenShake`).
  - Persistence: best score, mute, difficulty, and reduced-motion preference, all stored in `localStorage`.

## Controls (for reference when testing)

- **Desktop:** mouse to steer and click to fire; Arrow keys or WASD to move; Space to fire; X or Shift for a bomb; P or Esc to pause.
- **Mobile:** drag on the left to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game self-contained in `index.html`. Do not add npm dependencies, bundlers, or external network assets unless the user asks for them.
- Preserve the single-IIFE structure and `'use strict'`. Put new constants alongside the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than as magic numbers inside logic.
- Use `dt`-scaled movement for anything frame-dependent so behavior stays consistent across refresh rates.
- Wrap `localStorage` access so the game still works when storage is blocked (current code calls it directly; new code should be defensive).
- Respect the `reducedMotion` setting for new screen-shake, flash, or particle effects.
- Keep the layout working on phones (`viewport-fit=cover`, safe-area insets, `touch-action: none`).

## Verifying changes

There are no automated tests. After editing gameplay or rendering code:

1. Load `index.html` in a browser and check the browser console for errors.
2. Start a run, exercise the change (firing, bombs, bosses, pause, difficulty switch), and confirm the HUD updates.
3. Test a narrow viewport and touch controls if the change touches layout or input.

## Git

- Development branch for this work: `claude/gifted-tesla-rs1gc1`.
- Commit messages should describe the gameplay or code change in plain words.
