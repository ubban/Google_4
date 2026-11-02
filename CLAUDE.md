# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based Raiden-style vertical shoot-'em-up. There is no build step, no package manager, no framework, and no test suite.

Files:

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (starts around line 422 and runs to the end of the file).
- `README.md` — a stub (title only).

## Running the game

Open `index.html` directly in a browser. No server or install is needed. For mobile testing, serve the folder with any static server and load it from the device.

## Architecture (inside `index.html`)

The script is an IIFE-style block of plain JavaScript using the 2D canvas API.

- **Setup**: DOM handles are grabbed at the top of the script (`canvas`, `overlay`, `#hud` elements, touch controls, etc.).
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` — `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access is a direct call, so keep reads/writes defensive if you add new ones (private mode or blocked storage can throw).
- **Difficulty**: `DIFFICULTIES` object (Easy / Normal / Hard), applied via `setDifficulty()`.
- **Stages**: `stage` advances after `KILLS_PER_STAGE` (18) kills. Each stage ends with a boss from the `BOSSES` table; boss attack patterns live in `BOSS_PATTERNS`.
- **Weapons**: fire logic is in `fireWeapon()` and the helpers `fireAimedSpread`, `fireColumns`, `fireCross`, `fireDoubleRing`, `fireHomingSeeded`, `fireRadial`, `fireSpiralBurst`. Power-ups are spawned by `maybeSpawnPowerUp()` and applied by `applyPowerUp()`.
- **Bombs**: `useBomb()` / `flashBomb()`; the HUD bomb icons are rendered by `renderBombs()`.
- **Main loop**: `update()` for state, `draw()` for rendering, driven by `requestAnimationFrame(loop)`. Pause is `togglePause()`.
- **Audio**: a lazily-created `AudioContext` (`ensureAudio()`), with synthesized sounds (`beep`, `noiseBurst`, `panFor`). Mute is toggled by `updateMuteBtn()`.
- **Accessibility**: reduced-motion mode (`updateMotionBtn()`) disables screen shake, flashes, and pulses. Screen-reader announcements go through `announce()` into `#srLive`.
- **Perf tracking**: `trackPerf()`.

## Controls

- Desktop: mouse to steer, click to fire; Arrows/WASD to move; Space to fire; X or Shift for bomb; P or Esc to pause.
- Touch: drag in the left zone (virtual joystick), tap FIRE, tap BOMB.

Keyboard state is tracked in a `keys` map keyed by `e.code`. Pointer and touch handlers use `canvasPointFromEvent()` and the joystick functions.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Don't introduce a build step, bundler, or dependencies without asking.
- Match the existing style: 2-space indentation, `const`/`let`, single quotes in JS, and small named helper functions.
- Keep CSS responsive. The layout uses `clamp()`, `vw`, `env(safe-area-inset-*)`, and `touch-action: none`. Check changes at phone width.
- Don't remove the `aria-live` region or the `title`/`aria-label` attributes on controls.
- Preserve the `skyfighter.*` localStorage key names; changing them resets players' saved best scores.

## Verifying changes

There is no automated test or lint setup. To verify, open `index.html` in a browser and play through:

1. Start on each difficulty, confirm the HUD updates (score, best, multiplier, weapon, hearts, bombs).
2. Clear a stage to reach the boss, and check the boss bar and stage banner.
3. Test pause, mute, and reduced-motion toggles, and confirm they persist after reload.
4. On a narrow viewport (or device emulation), check the touch controls and HUD wrap.

Syntax-check the inline script if you touch it heavily, for example by extracting the `<script>` block and running `node --check`.

## Git

- Develop on the branch named in the session instructions (currently `claude/gifted-tesla-kufvxo`).
- Commit with clear messages and push with `git push -u origin <branch>`.
