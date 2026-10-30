# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

Files:

- `index.html` — the whole game: markup, CSS (`<style>`), and JavaScript (`<script>`) in one file.
- `README.md` — placeholder title only.

There is no build step, package manager, test suite, or linter configured.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server` from the repo root) and visit
`http://localhost:8000`. No dependencies need to be installed.

## Code layout inside `index.html`

- The page's `<style>` block defines the HUD (pill-style score/lives/bombs), the
  glassmorphic overlay card, touch controls, and the low-health vignette.
- The `<script>` block is one IIFE wrapped in `'use strict'`. Everything is
  private to that closure; nothing is exposed on `window`.
- Key constant tables near the top of the script:
  - `DIFFICULTIES` — spawn rate, enemy speed, shoot chance, starting lives and
    bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
  - `THEMES`, `ENEMY_TYPES`, `BOSSES` — per-stage visuals, enemy stats, and boss definitions.
  - `BOSS_PATTERNS` — the bullet-pattern functions bosses cycle through
    (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireSpiralBurst`, and others).
- Game loop: `update(dt)` advances state each frame, `loop(now)` drives it via
  `requestAnimationFrame`, and `draw*` helpers (`drawPlane`, `drawBossShip`) render.
- Audio is synthesized with the Web Audio API (`beep`, `noiseBurst`); `ensureAudio()`
  must run from a user gesture before sound plays.
- Touch input uses a joystick on the left (`moveZone`) and fire/bomb buttons on the
  right. Keyboard and mouse input are handled separately.

## Persistence

The game stores four values in `localStorage`, all under the `skyfighter.` prefix:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | `'1'` when audio is muted |
| `skyfighter.difficulty` | Selected difficulty name |
| `skyfighter.reducedMotion` | `'1'`/`'0'` reduced-motion override |

If you rename or repurpose a key, existing players lose their saved data, so
call that out in the change.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks
  for a split. Do not add a bundler, framework, or external CDN dependency
  without asking.
- Match the existing style: `const` and `let`, two-space indentation,
  single quotes in JavaScript, and short section comments.
- Tuning values belong in the constant tables (`DIFFICULTIES`, `BOSSES`, etc.)
  rather than inline magic numbers.
- Preserve the `'use strict'` IIFE. Do not leak globals.
- Keep accessibility hooks intact: the `srLive` live region announces game
  events through `announce()`, and the reduced-motion toggle should gate screen
  shake and flashes.
- Keep the layout working at phone width. The canvas scales via `resize()` against
  `BASE_W` × `BASE_H` (480 × 720).

## Verifying changes

There is no automated test suite. After editing `index.html`:

1. Open the page in a browser and start a run on each difficulty.
2. Check that firing, bombs, power-ups, stage transitions, and the boss fight all work.
3. Check the mobile layout and touch controls, either with a device or browser dev tools.
4. Confirm the high score and settings persist across a reload.

If you can run a headless browser (Playwright/Chromium is often available),
it is a good way to smoke-test the start-to-game-over flow.

## Git

- Development happens on feature branches named `claude/...`. Do not push to
  `main` directly.
- Write commit messages in the imperative mood, with a short subject line and
  a body that explains the why for larger changes.
