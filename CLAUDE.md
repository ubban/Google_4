# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-file HTML5 canvas game. Everything lives in `index.html`:

- `<style>` (around lines 10–360): all CSS, including HUD, overlay card, touch controls, and theme variables.
- `<body>` (around lines 361–421): DOM scaffolding for the canvas, HUD, boss bar, touch controls, and the start/pause/game-over overlay.
- `<script>` (starts around line 422): the entire game engine, with no modules or build step.

`README.md` is a one-line placeholder. There is no package manager, test suite, linter, or build
tooling in the repo.

## Running and testing

- There is no build step. Open `index.html` directly in a browser, or serve the directory with any
  static server (for example `python3 -m http.server`) if you need to avoid `file://` quirks.
- There are no automated tests. Verify changes by playing the game in a browser. For changes to
  rendering, input, or layout, check desktop and a narrow/mobile viewport.
- Keep the file self-contained: do not add external scripts, fonts, or CDN dependencies unless the
  user asks for them.

## Architecture (inside the `<script>` block)

- **Fixed logical canvas**: `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the
  window (capped at 1.6x). All game coordinates use these base units.
- **Main loop**: `loop(now)` computes `dt` and calls `update(dt)` then `draw()`. Pause is handled by
  `togglePause()`.
- **Config tables** (edit these rather than hard-coding values elsewhere):
  - `DIFFICULTIES`: spawn rates, enemy speed, shoot chance, starting lives/bombs, boss HP multiplier.
  - `THEMES`: per-stage sky palettes, cloud and sun colors, and names. Stage cycles through them.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (multi-hit). Each has `minStage`.
  - `BOSSES`: visual designs and HP multipliers. Each boss is paired with a pattern list in
    `BOSS_PATTERNS`, indexed by boss design.
  - `KILLS_PER_STAGE` controls stage progression.
- **Weapons**: `fireWeapon()` handles Vulcan and Laser, with power levels 1–3 that set spread and
  lane counts. Bombs are handled by `useBomb()`.
- **Bosses**: `spawnBoss()` / `updateBoss()` / `killBoss()`. Bosses telegraph attacks before firing.
- **Power-ups**: `maybeSpawnPowerUp()` / `applyPowerUp()` (weapon level, bomb, extra life).
- **Scoring**: `addScore()` applies a kill-chain multiplier. `breakChain()` resets it.
- **Adaptive quality**: `trackPerf()` samples frame times and lowers `quality` (0 full, 1 medium,
  2 low) when frame rate drops. It scales particle counts and glow effects.
- **Audio**: Web Audio API via `ensureAudio()`. Sounds are synthesized (`beep`, `noiseBurst`) and
  grouped in `sfx`. Audio is created lazily on the first user gesture.
- **Accessibility**: `announce()` writes to the `#srLive` aria-live region. Reduced motion is
  honored from `prefers-reduced-motion` and can be toggled, which controls screen shake and flashes.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; if unset, the OS preference is used |

Reads and writes are not wrapped in try/catch. If you add storage access, guard it, because
`localStorage` can throw in private windows or when site data is blocked.

## Controls

- **Keyboard**: Arrow keys / WASD to move, Space to fire, X or Shift to bomb, P or Escape to pause.
- **Mouse**: move the pointer over the canvas to steer; hold the mouse button to fire.
- **Touch**: virtual joystick on the left (`#moveZone`), fire and bomb buttons on the right
  (`#fireBtn`, `#bombBtn`). Touch controls only appear on touch-capable devices.

## Conventions

- Plain ES2015+ JavaScript and CSS inside `index.html`. Match the existing style: 2-space
  indentation, single quotes in JS, `const`/`let`, and arrow functions for handlers.
- Prefer small, targeted edits. The file is large (~1,700 lines); avoid reformatting unrelated code.
- Keep the HUD and overlay accessible: keep `aria-live` announcements for meaningful state changes
  (stage changes, boss defeat, game over), and keep text legible on the existing gradients.
- Check that mobile layout still works after CSS changes (no horizontal scroll, touch controls not
  overlapping the HUD).
- Balance changes belong in the config tables above, not scattered through the update loop.

## Git

- Development happens on the branch designated by the session; do not push to other branches
  without permission.
- Write descriptive commit messages in the imperative mood, in the style of the existing history
  (for example, "Add Sky Fighter: ...").
