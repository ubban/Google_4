# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game lives in one file, `index.html` (~1,670 lines: inline CSS, markup, and one inline `<script>`). `README.md` is a placeholder and has no project documentation.

There is no build system, package manager, bundler, test suite, linter, or CI configuration. Nothing is installed or compiled. The game runs by opening `index.html` in a browser.

## Repository layout

```
index.html   # The whole game: <style> (lines ~10-360), <body> markup (~361-421), <script> (~422-end)
README.md    # Placeholder title only
CLAUDE.md    # This file
```

## Running and verifying changes

- Open `index.html` directly in a browser. No server is needed.
- For automated checks, Playwright with the pre-installed Chromium works. Load the file with a `file://` URL and drive it through the DOM and keyboard events.
- There are no unit tests. Verify gameplay changes by playing or scripting the game in a real browser, and check the console for errors.

## Architecture of `index.html`

The script is one IIFE with `'use strict'`. Its sections, roughly in file order:

- **DOM references and storage keys.** Elements are looked up once by ID. `localStorage` keys are prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`.
- **Tuning tables** (near the top of the script):
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed, enemy shoot chance, starting `lives`, starting `bombs`, and `bossHpMul`.
  - `KILLS_PER_STAGE = 18`. Reaching this count triggers the boss.
  - `THEMES`: six sky palettes that cycle with stage.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, from stage 2).
  - `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN), each with its own colors and `hpMul`.
- **Audio.** Synthesized with the Web Audio API through `beep()` and `noiseBurst()`. No audio files. `ensureAudio()` must run on a user gesture.
- **Adaptive quality.** `trackPerf()` samples frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low) to keep frame rate up.
- **Layout.** `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the viewport.
- **Input.**
  - Keyboard: arrows or WASD to move, Space or hold to fire, X or Shift for a bomb, P or Esc to pause.
  - Mouse: move to steer, click to fire.
  - Touch: a virtual joystick on the left (`moveZone`), plus FIRE and BOMB buttons.
- **Weapons.** `fireWeapon()` has two weapons, `vulcan` (bullets, with spread at levels 2-3) and `laser` (pierce at level 3). `player.level` runs 1-3.
- **Power-ups.** `maybeSpawnPowerUp()` drops power-ups from killed enemies. The types are `life` (6%), `bomb` (8%), and `weapon` (20%). `applyPowerUp()` applies them. A `weapon` power-up raises `player.level` until 3. At level 3 it switches between vulcan and laser and resets the level to 1.
- **Bosses.** `BOSS_PATTERNS` is one array of three attack functions per boss. `updateBoss()` cycles through them. Attack helpers include `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, and `fireDoubleRing`.
- **Game state.** `resetGame()` sets the initial state. `update(dt)` advances the simulation and `draw()` renders it. `loop(now)` is the `requestAnimationFrame` driver. `loseLife()` grants 1.5 s of invulnerability. `endGame()` writes the best score to `localStorage`. `togglePause()` pauses the loop.
- **Accessibility.** `announce()` writes to the `#srLive` live region. A reduced-motion setting turns off screen shake, hit flashes, and pulses. It defaults to the `prefers-reduced-motion` media query and can be toggled in the UI.

## Conventions

- Keep the game as a single self-contained file. Do not add external scripts, stylesheets, or fonts unless the user asks. The page makes no network requests today.
- Keep all code inside the existing IIFE. Do not add new globals.
- Keep tuning values in the tables at the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) instead of hard-coding them in logic.
- Match the existing style: `const` and `let`, arrow functions, two-space indentation, single quotes in JS.
- If you add a setting, persist it under the `skyfighter.` prefix and wrap storage reads and writes so the game still works when `localStorage` is unavailable.
- Any new UI text or state change should be announced through `announce()` for screen readers.
- Preserve the mobile layout and touch controls when changing HUD or overlay markup.

## Gotchas

- `index.html` is large. Use targeted `grep` or line ranges rather than reading the whole file at once.
- `setDifficulty()` and `updateMuteBtn()` run during initialization, before the code that follows them in the IIFE. Keep that in mind when reordering the script.
- `THEMES` cycles by stage using `(stage - 1) % THEMES.length`, so the palette repeats after six stages.
- Boss attacks use `Math.random()`, so the attack patterns are not deterministic.
- Game-over and pause both use the shared `#overlay` element. Changing one flow can affect the other.

## Git

- Development branch: `claude/gifted-tesla-ic3g2m`. Do not push to other branches without explicit permission.
- Make focused commits with clear messages.
