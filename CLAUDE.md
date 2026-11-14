# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file browser game: a Raiden-style vertical shoot-'em-up with weapons, smart bombs, power-ups, stages, and bosses. There is no build system, package manager, test suite, or backend.

Files:

- `index.html` (~1,670 lines): all markup, CSS (`<style>`), and game logic (`<script>`) in one file.
- `README.md`: a placeholder title only.

## Running locally

No build step. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code structure (`index.html`)

- **Markup**: a `#container` holds a `<canvas id="gameCanvas">`, HUD elements (`#hud`, `#hearts`, `#bombs`, `#weaponStat`, `#multStat`), overlay and start button, pause/mute/motion buttons, touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), a boss bar, and an `aria-live` region (`#srLive`) used by `announce()`.
- **Styles**: a dark color scheme (`color-scheme: dark`). The game uses a canvas and fixed-size HUD overlays. Animations include `heartBeat` and screen-shake/flash effects. A `.reduced-motion` body class disables motion effects.
- **Script**: one IIFE in strict mode. Everything is private to it. Key sections, roughly in file order:
  - Constants: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six bosses, each with a `hpMul`), `BOSS_PATTERNS`.
  - Persistence via `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`).
  - Audio: a lazily created `AudioContext` (`ensureAudio()`), with `beep()` and `noiseBurst()` synthesizers. No audio files.
  - Adaptive quality: `trackPerf()` steps `quality` (0 full, 1 medium, 2 low) from a rolling frame-time average.
  - Game state and loop: `resetGame()`, `update(dt)`, `updateBoss(dt)`, `draw()`, `loop(now)`. `dt` is clamped to 50 ms.
  - Input: keyboard (`keys[e.code]`), touch joystick and fire/bomb buttons, and mouse/pointer on the canvas.
  - Power-ups: `maybeSpawnPowerUp()` and `applyPowerUp()` handle `life`, `bomb`, and `weapon` types.

## Controls (for reference when testing)

- Move: Arrow keys or WASD. Touch: drag in the left zone.
- Fire: Space (hold), or hold the on-screen fire button.
- Bomb: X or either Shift key, or the bomb button.
- Pause: P or Escape.

## Conventions

- Keep the game in a single self-contained `index.html` unless the user asks for a restructure. Do not add a build step or dependencies without asking.
- Keep the script inside its IIFE. Do not add globals.
- New DOM references go at the top of the script with the existing `getElementById` lookups.
- Persistent settings use the `skyfighter.` localStorage prefix. Wrap new storage reads and writes so they still work if storage is unavailable (private windows, blocked site data), since the current code does not guard them.
- Respect the `reducedMotion` setting and `prefers-reduced-motion` when adding visual effects such as shake, flash, or pulsing.
- Announce important game events through `announce()` for screen readers.
- Preserve mobile support: touch controls, `touch-action: none`, and the `resize()` logic that scales the canvas to the viewport.
- Use the existing constants and tables (`ENEMY_TYPES`, `BOSSES`, `DIFFICULTIES`, `THEMES`) when adding enemies, bosses, stages, or difficulty tweaks, rather than hard-coding values in logic.

## Testing

There are no automated tests. Verify changes by loading `index.html` in a browser and playing through the affected path: start, fire, bomb, pause, a stage transition, a boss fight, game over, and the best-score save. Check both desktop keyboard and a touch-sized viewport. For visual or gameplay changes, a headless browser screenshot is a reasonable check.

## Git

- Development branch for this work: `claude/gifted-tesla-e8k0yl`.
- Commit messages: short imperative summary, followed by an optional body.
