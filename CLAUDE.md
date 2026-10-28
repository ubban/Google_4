# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style
vertical shoot-'em-up. There is no build step, no package manager, and no test suite.

Files:

- `index.html` (~1,670 lines): the entire game. CSS is in the `<style>` block (≈ lines 10–359),
  markup in the body (canvas + HUD + overlays + touch controls), and all game logic in one
  `<script>` block (≈ lines 422–1667).
- `README.md`: placeholder only.
- `CLAUDE.md`: this file.

## Running it

Open `index.html` directly in a browser. Nothing needs to be installed or compiled.
To serve it locally instead: `python3 -m http.server` from the repo root, then visit
`http://localhost:8000`.

There are no automated tests and no linter configured. Verify changes by playing the game in a
browser (desktop and, if touching input or layout, a mobile-sized viewport).

## Code layout inside `index.html`

The script is a single `(function () { 'use strict'; ... })();` block, so all state is private to it.
Roughly in order:

1. **Config constants**: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`,
   `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` (bullet-pattern functions for bosses).
2. **Persistence & settings**: best score, mute, difficulty, and reduced-motion preference are
   stored in `localStorage` under `skyfighter.*` keys. Every access should tolerate storage being
   unavailable.
3. **Audio**: a WebAudio synth (`ensureAudio`, `beep`, `noiseBurst`, the `sfx` object). No audio
   files. Audio is created lazily on the first user gesture.
4. **UI helpers**: hearts/bombs rendering, flashes, combo pops, stage banners, screen shake,
   `announce()` for screen readers.
5. **Input**: keyboard (`keys[e.code]`), mouse/touch drag on the canvas, a virtual joystick on
   touch devices, and the fire/bomb/pause buttons. `resize()` scales the canvas to a fixed
   `BASE_W × BASE_H` (480 × 720) logical resolution.
6. **Game state & flow**: `resetGame()`, `update(dt)` (movement, firing, spawning, collisions,
   scoring), `applyPowerUp()`, `useBomb()`, `loseLife()`, `endGame()`, `togglePause()`.
7. **Enemies & bosses**: `spawnEnemy()`, `spawnBoss()`, `updateBoss()`, `killBoss()`, and the
   `fire*` pattern helpers used by `BOSS_PATTERNS`.
8. **Rendering**: `draw()` and its helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Drawing is
   plain Canvas 2D; there are no sprite assets.
9. **Main loop**: `loop(now)` drives `update` and `draw` via `requestAnimationFrame`.

### Game rules to keep in mind

- **Difficulty** (`DIFFICULTIES`) sets spawn rate, enemy speed and fire chance, starting lives and
  bombs, and boss HP multiplier. Changing one balance number should be a deliberate decision.
- **Stages**: each stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss. The boss
  cycles through `BOSSES`, and the theme (`THEMES`) rotates with the stage.
- **Enemy types** unlock by `minStage`: grunts and interceptors from stage 1, tanks from stage 2.
- **Weapons**: `vulcan` and `laser`, each with 3 levels. A weapon power-up raises the level, and
  past level 3 it swaps weapon and resets to level 1.
- **Score multiplier**: a kill chain (`addScore`, `breakChain`) multiplies points.

### Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrows or WASD | Virtual joystick |
| Fire | Space (hold) | Fire button |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Esc | Pause button |
| Mute | Mute button | Mute button |

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Don't add a
  bundler, framework, or package manager without being asked.
- **No external dependencies.** The page uses no CDN scripts or fonts. Keep it that way.
- **Match the existing style.** Two-space indentation, `const`/`let`, semicolons, single quotes in
  JS, compact one-line object literals for config tables, and short `function` declarations for
  game logic. Follow the surrounding code's comment density (it is sparse).
- **Keep the game loop allocation-light.** Avoid creating new objects or closures per frame in
  `update`/`draw` when a reused value works. Entity arrays are pruned each frame with
  `filter`/`splice`, so keep that pattern consistent rather than introducing a second one.
- **Respect the reduced-motion setting.** Screen shake and flashes are gated by `reducedMotion`
  (and the `reduced-motion` body class). New visual effects that move or flash should honor it.
- **Accessibility.** Announce meaningful state changes (stage start, boss, game over) through the
  `announce()` helper, which writes to the `srLive` live region.
- **Storage safety.** Wrap `localStorage` reads and writes so the game still runs when storage is
  blocked (private windows, disabled site data).
- **Mobile first-class.** Touch controls, `touch-action: none`, and safe-area insets are
  intentional. Check layout at phone widths after UI changes.

## Git

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-s9gi9v`).
- Commit messages: imperative subject line, then a short body explaining *why* when it isn't
  obvious from the diff.
- Don't open a pull request unless asked.
