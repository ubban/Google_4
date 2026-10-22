# CLAUDE.md

Guidance for working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-em-up built as a single self-contained
HTML5 canvas page. There is no build step, no package manager, and no test suite.

- `index.html` — the whole game: inline `<style>`, markup, and one inline `<script>`
  wrapped in an IIFE with `'use strict'`.
- `README.md` — minimal project title only.

To run it, open `index.html` in a browser. Nothing needs to be installed.

## Code layout (inside `index.html`)

The script is organized into commented sections. Keep new code in the matching section:

- **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is created
  lazily on the first user gesture, so keep it that way (browsers block autoplay).
- **Config tables** — `DIFFICULTIES`, `THEMES` (stage backdrops), `ENEMY_TYPES`,
  `BOSSES`, and `BOSS_PATTERNS`. Tuning changes usually belong here, not in logic.
- **Input** — `keys` map keyed by `KeyboardEvent.code`, touch joystick/fire/bomb
  handlers, and mouse/pointer steering via `canvasPointFromEvent`.
- **Game state and loop** — `resetGame`, `update(dt)`, `draw()`, and `loop(now)`.
  `dt` is clamped to 50 ms so pausing or tab switches do not cause teleporting.
- **Weapons and bombs** — `fireWeapon` (Vulcan/Laser, 3 power levels), `useBomb`
  (limited, clears the screen and damages bosses), and `applyPowerUp`.
- **Bosses** — `spawnBoss`, `updateBoss`, `killBoss`, and the bullet-pattern helpers
  (`fireRadial`, `fireAimedSpread`, `fireCross`, etc.) that `BOSS_PATTERNS` composes.
- **Persistence** — `localStorage` keys are prefixed `skyfighter.`: `bestScore`,
  `muted`, `difficulty`, `reducedMotion`. Reads fall back to defaults when unset.

## Conventions

- Plain ES2015+ JavaScript and CSS. No frameworks, no bundlers, no external dependencies
  unless there is a strong reason to add one.
- Keep everything in `index.html` unless the user asks to split it. If a split is
  requested, keep the page loadable from the filesystem (no module imports that
  require a dev server).
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default,
  short section banners like `// ---------- Input ----------`.
- Game-facing coordinates use the `BASE_W × BASE_H` logical space (480×720). Scaling to the
  window is handled in `resize()`; do not hard-code CSS pixel sizes for game objects.
- Respect the reduced-motion and mute settings when adding effects or sound. Screen shake
  and flashes should go through the existing helpers (`screenShake`, `flashHit`, etc.) so
  they honor the toggle.
- Accessibility: screen-reader announcements go through `announce()` into the `srLive`
  region; keep new status text routed the same way.

## Controls (for reference when changing input)

- Desktop: mouse to steer and click to fire; arrows/WASD to move; Space to fire;
  X or Shift to bomb; P or Esc to pause.
- Touch: drag on the left zone to fly; FIRE and BOMB buttons on the right.

## Verifying changes

There is no automated test or lint command. After editing:

1. Open `index.html` in a browser and check the browser console for errors.
2. Play through a few waves, a boss fight, a bomb, a power-up, and a game over.
3. Check pause (P/Esc), difficulty switching, mute, and reduced-motion toggles.
4. For layout changes, check at a phone-sized viewport as well as desktop.

Use the Chromium browser tooling for this if it is available. Note in your summary which
checks you ran and which you could not.

## Git

- Development happens on the branch specified for the session; do not push elsewhere.
- Commit messages in this repo are descriptive and sentence-cased, with a short summary
  line followed by a body explaining the gameplay or code change.
