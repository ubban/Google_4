# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

Files:

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (no build step, no dependencies, no external assets). The canvas is fixed at a logical 480×720 (`BASE_W`, `BASE_H`) and scaled to fit the viewport.
- `README.md` — a one-line title. Keep it brief unless the user asks for more.

There is no package manager, test suite, linter, or build configuration.

## Running and verifying

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by loading the page and playing: check the title overlay, movement, firing, bombs, pause, stage transitions, boss fights, game over, and the mobile joystick/touch controls.
- Browser-side checks may be run with the pre-installed Playwright/Chromium if needed; do not reinstall Playwright.

## Architecture of `index.html`

Inside the script, the game is organized as plain functions and module-level state (no classes, no modules):

- **Persistence:** `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Keep the prefix and wrap new storage access so the game still works when storage is unavailable.
- **Tuning tables:** `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES` (with `minStage`), and `BOSSES`. Prefer adding or adjusting data here over hard-coding values in logic.
- **Game loop:** `update(dt)` advances state, `draw()` renders, and `loop(now)` drives both via `requestAnimationFrame`. `resetGame()` restores initial state for a new run.
- **Weapons and bombs:** `fireWeapon()` (Vulcan/Laser, 3 power levels), `useBomb()` (limited smart bombs that clear the screen and damage bosses), and `applyPowerUp(type)`.
- **Bosses:** `spawnBoss()`, `updateBoss(dt)`, and `killBoss()`, with bullet patterns in `BOSS_PATTERNS` and helpers such as `fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, and `fireDoubleRing`.
- **Feedback:** `beep`/`noiseBurst` for Web Audio sound effects (`ensureAudio()` must run after a user gesture), `announce()` for screen-reader text, `screenShake`, `flashHit`, `showComboPop`, and `showStageBanner`.
- **Input:** keyboard handlers on `window` (arrows/WASD to move, Space to fire, X/Shift for bomb, P/Escape to pause), plus touch/pointer joystick and fire handling.
- **Responsive layout:** `resize()` plus CSS that respects safe-area insets. Keep `touch-action: none` on the canvas and overlays.
- **Accessibility:** a reduced-motion setting toggles the `reduced-motion` body class and disables shake/pulse animations. Preserve that behavior when adding motion.

## Conventions

- Keep the game as a **single self-contained HTML file** unless the user asks for a restructure. Do not add a build step, frameworks, or CDN dependencies without a request.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, small named helper functions, and comments only where the logic is not obvious.
- Use the existing naming for storage keys, sound names, and table constants.
- Keep gameplay numbers in the tuning tables where possible so balance changes stay localized.
- Prefer `textContent` for text output. `innerHTML` is used only for generated icon markup (hearts and bombs in `renderHearts`/`renderBombs`); never feed it user-controlled strings such as player names or stored values.

## Git and workflow

- Default development branch for this session is `claude/gifted-tesla-0w64yk`. Commit and push there; do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Commit messages should be short and imperative, with a body for larger changes, as in the existing history (for example, "Add Sky Fighter: ...").
- Only commit or push when the user asks for it.
