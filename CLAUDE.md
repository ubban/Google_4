# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-em-up
built as an HTML5 canvas app. The entire game lives in `index.html`: markup, CSS
(`<style>`, roughly lines 10–362), and one IIFE-wrapped `<script>` (roughly lines 422–1668).
`README.md` is a placeholder with no setup instructions.

There is no build step, no package manager, no bundler, and no test suite. Open
`index.html` in a browser to run it.

## Layout of `index.html`

- **Markup** (~lines 1–421): HUD (score, best, multiplier, weapon name/level, hearts,
  bombs, stage banner, combo pop), start/pause overlay with difficulty buttons,
  boss health bar, touch controls (joystick `moveZone`, `fireBtn`, `bombBtn`), and
  mute / motion / pause buttons. `#srLive` is an ARIA live region used by `announce()`.
- **Styles** (~lines 10–362): HUD pill styling, glassmorphic overlay, animations for
  hit flash, bomb flash, screen shake, and low-health vignette. Reduced motion is
  applied via the `body.reduced-motion` class.
- **Script** (~lines 422–1668), in this order:
  1. DOM lookups and `STORAGE_*` keys (`skyfighter.*` in `localStorage`).
  2. `announce()` and adaptive quality (`trackPerf`, `quality` 0–2) that throttles
     rendering when frame time is high.
  3. Config tables: `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`, `THEMES`
     (sky palettes cycled per stage), `ENEMY_TYPES` (grunt, interceptor, tank),
     `BOSSES` (six boss designs).
  4. Audio: `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` helpers. Audio
     is created lazily on first user gesture.
  5. Input: keyboard (`keys[e.code]`), touch joystick, fire/bomb buttons, and mouse
     on the canvas.
  6. Game state: `resetGame()`, `update(dt)`, `draw()`, `loop(now)`. `dt` is clamped
     to 0.05 s.
  7. Systems: spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), weapons
     (`fireWeapon`, 3 power levels for Vulcan/Laser), bombs (`useBomb`), scoring with
     a kill-chain multiplier (`addScore`, `breakChain`), power-ups (`applyPowerUp`).
  8. Bosses: `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, homing,
     double ring) driven by `updateBoss`; `killBoss` advances the stage.
  9. Lifecycle: `loseLife`, `endGame`, `togglePause`, and high score persistence.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split.
  Do not add a build step or external dependencies without a clear reason.
- **No external network calls or CDN assets.** The game is self-contained.
- **Vanilla JS, ES2015+ syntax**, `'use strict'` inside the IIFE. Match the existing
  style: 2-space indent, `const` by default, arrow functions for callbacks, and
  `function` declarations for game systems.
- **Game tuning lives in constant tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`,
  `THEMES`, `BOSS_PATTERNS`). Prefer adjusting those over hard-coding numbers in
  update logic.
- **Coordinates use a fixed logical canvas** of `BASE_W = 480` × `BASE_H = 720`,
  scaled by `resize()`. Positions and sizes in game logic are in logical pixels.
- **Persist only through the existing `STORAGE_*` keys.** Wrap new storage reads and
  writes in `try/catch` if they can run in a private window or with blocked storage.
  The current code does not guard `localStorage`, so treat that as a known gap rather
  than a pattern to copy.
- **Accessibility:** respect `prefers-reduced-motion` and the `reducedMotion` toggle
  for any new visual effects that shake or flash. Announce important state changes
  through `announce()`.
- **Audio** must start from a user gesture. Call `ensureAudio()` from input handlers
  before playing sounds.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` directly in a browser, or serve the directory with any static
   server (for example `python3 -m http.server`).
2. Click **Start** and play through at least one kill, a power-up pickup, a bomb, and
   a stage transition. Reach a boss if the change touches bosses or stages.
3. Check the console for errors. Test the touch controls with device emulation if the
   change affects input.
4. Confirm that pause (`P` / `Escape` / pause button), bombs (`X` / `Shift`), mute, difficulty selection, and best-score
   persistence still work after reload.

Describe in the PR or final message which of these you actually ran. Do not claim a
browser check passed unless you ran it.

## Git workflow

- Develop on the branch the session specifies, and commit with descriptive messages.
- Do not open pull requests unless the user asks.
