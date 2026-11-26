# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas.

- `index.html` (~1,670 lines): the entire game. HTML, CSS, and one inline
  `<script>` IIFE. No external assets, libraries, or network calls.
- `README.md`: a one-line placeholder. It is not documentation for the game.

There is no package manager, build step, bundler, test suite, linter, or CI
configuration. Do not invent commands for these.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server`). No install step is needed.

## Structure of `index.html`

Top to bottom:

1. **`<style>`** (about lines 10–360): all CSS, including the HUD pill bar, the
   glassmorphic overlay card, touch controls, the vignette and hit-flash
   effects, and `body.reduced-motion` overrides.
2. **Markup** (about lines 362–420): `#gameCanvas`, HUD elements (`#score`,
   `#best`, `#mult`, `#weaponName`, `#weaponLvl`, `#hearts`, `#bombs`), boss bar
   (`#bossBarWrap`), `#overlay` start/pause/game-over card, mute/motion/pause
   buttons, and touch controls (`#touchControls`, `#fireBtn`, `#bombBtn`).
3. **`<script>`** (from about line 422): one `(function () { 'use strict'; ... })()`
   IIFE. Nothing is exported to `window`. Sections, in order:
   - DOM lookups and `STORAGE_*` keys (`skyfighter.*` in `localStorage`).
   - Config tables: `DIFFICULTIES` (easy/normal/hard), `KILLS_PER_STAGE`,
     `THEMES` (sky palettes cycled per stage), `ENEMY_TYPES`, `BOSSES`,
     `BOSS_PATTERNS`.
   - Audio: a lazily created WebAudio context (`ensureAudio`) and synthesized
     `beep` / `noiseBurst` sound effects. No audio files.
   - Input: a `keys` map from `keydown`/`keyup` (keyed by `e.code`), mouse and
     pointer steering, and a virtual joystick for touch devices.
   - Game state and logic: `resetGame`, `spawnEnemy`, `spawnBoss`,
     `fireWeapon`, `update(dt)`, `applyPowerUp`, `updateBoss`, `killBoss`,
     `loseLife`, `endGame`, `togglePause`.
   - Rendering: `draw()` and the drawing helpers (`drawPlane`, `drawBossShip`,
     `drawClouds`).
   - Main loop: `loop(now)` driven by `requestAnimationFrame`, with a fixed
     `BASE_W` × `BASE_H` (480 × 720) logical resolution scaled by `resize()`.

## Game rules worth knowing before editing

- **Controls:** arrows/WASD or mouse to move, Space or click to fire, X or
  Shift to drop a bomb, P or Esc to pause. Touch uses the left drag zone and
  the FIRE/BOMB buttons.
- **Difficulty** changes spawn rates, enemy speed, enemy fire chance, starting
  lives, starting bombs, and boss HP multiplier. The choice persists in
  `localStorage`.
- **Stages:** after `KILLS_PER_STAGE` (18) kills in a stage, a boss is spawned
  (about 1.2 s later). Bosses cycle through `BOSSES` and use the
  `BOSS_PATTERNS` attacks. Defeating the boss (`killBoss`) starts the next stage.
- **Scoring:** kill chains raise a score multiplier, which `breakChain` resets.
- **Persistence:** best score, mute, difficulty, and reduced-motion preference
  are stored in `localStorage` under the `skyfighter.*` keys. Every access must
  stay inside the existing try/fallback pattern. Storage can be blocked, for
  example in private windows.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a
  split. Do not add a build system, framework, or external CDN dependency
  without an explicit request.
- Keep the code inside the existing IIFE. Do not add globals.
- Game tuning lives in the config tables (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`). Prefer changing those over hard-coding numbers in logic.
- Preserve accessibility hooks: `#srLive` announcements through `announce()`,
  `aria` labels on controls, and the reduced-motion checks. `screenShake()`
  dampens shake when `reducedMotion` is on, and the hit/bomb flashes return
  early when it is on.
- Preserve touch support. The `isTouchDevice` check toggles `#touchControls`.
- Animation and time-based logic use the `dt` passed to `update`. Keep new
  motion frame-rate independent.

## Verifying changes

There are no automated tests. After changing game code:

1. Open `index.html` in a browser and check the console for errors.
2. Start a game, play through at least one stage transition and one boss fight,
   then check game over and the best-score update.
3. Check pause (P/Esc), bomb (X/Shift), and mute.
4. For UI or layout changes, check a phone-width viewport and the touch controls.

## Git

- Default development branch for this session: `claude/gifted-tesla-pn4khb`.
- Commit messages in this repo use an imperative subject line, with a body
  describing the user-visible change, as in the existing `Add Sky Fighter` commit.
