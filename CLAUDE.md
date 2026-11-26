# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game (HTML, CSS, and JavaScript) lives in `index.html` (~1,670 lines). There is no build step, no package manager, and no test suite.

Files:

- `index.html`: the full game.
- `README.md`: a one-line title.
- `CLAUDE.md`: this file.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file server, for example:

```bash
python3 -m http.server 8000
```

There is nothing to install or compile. Keep it that way unless the user asks for a build system.

## Code layout inside `index.html`

- **Styles** (inline `<style>`, roughly lines 1–360): CSS custom layout for the canvas container, HUD, stage banner, boss bar, touch controls, overlay, and a `body.reduced-motion` class that disables shake, flash, and pulse effects.
- **Markup** (roughly lines 360–420): `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#score`, `#best`, `#mult`, `#weaponStat`, `#hearts`, `#bombs`), the boss bar, the combo pop, the stage banner, the top bar (`#muteBtn`, `#motionBtn`, `#pauseBtn`), the touch controls (`#moveZone`, `#fireBtn`, `#bombBtn`), and the start/pause `#overlay`. `#srLive` is an ARIA live region for screen-reader announcements.
- **Script** (one IIFE starting around line 423):
  - **Tuning constants**: `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier per `easy` / `normal` / `hard`), `KILLS_PER_STAGE` (18), `THEMES`, `ENEMY_TYPES`, `BOSSES`.
  - **Persistence**: `localStorage` keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Every read and write should tolerate storage being unavailable.
  - **Audio**: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on the first user gesture.
  - **Input**: keyboard (`keys` map), touch joystick, touch and mouse fire/bomb buttons, and canvas mouse aiming.
  - **Game state and loop**: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`, `fireWeapon`, and the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`). Drawing helpers are `drawPlane` and `drawBossShip`.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks to split it. Do not add external dependencies or CDN links without a clear reason.
- Preserve the existing style: `const` for tuning values, camelCase identifiers, and 2-space indentation.
- Gameplay numbers belong in the constants at the top of the script (`DIFFICULTIES`, `KILLS_PER_STAGE`, and so on), not scattered through update logic.
- Any new setting that should persist goes under the `skyfighter.*` localStorage namespace and must be wrapped so the game still works when storage is blocked.
- Keep touch and keyboard controls working together. Any input change should be checked on both.
- Respect `reducedMotion`: new screen shake, flashes, or pulsing animations must check `reducedMotion` and the `body.reduced-motion` class.
- Keep the UI accessible: buttons need `title` or text labels, and important status changes should go through `#srLive` via `announce()`.
- Keep the layout responsive. The canvas scales to `BASE_W` × `BASE_H` (480 × 720) through `resize()`, and the page uses `overflow: hidden` with `touch-action: none`. Do not add scrolling content to the game screen.

## Verifying changes

There are no automated tests or linters configured. Verify changes by:

1. Opening `index.html` in a browser (or a static server) and checking the console for errors.
2. Playing through start, a stage clear, a boss fight, bomb use, pause/resume, and game over on at least one difficulty.
3. Checking keyboard and touch-control paths if input code changed.
4. Checking mute, difficulty, and reduced-motion toggles persist across a page reload.

For visual or layout changes, take a screenshot at both desktop and phone-sized viewports.

## Git

- Development branch for this session: `claude/gifted-tesla-y6rbma`.
- Commit messages should be short and imperative, in the style of the existing history (for example, `Add Sky Fighter: Raiden-style vertical shoot-em-up game`).
- Do not open pull requests unless the user asks.
