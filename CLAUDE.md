# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, browser-based vertical shoot-'em-up in the style of Raiden.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>`. No build step, no bundler, no framework, no external dependencies.
- `README.md` — minimal placeholder.

There is no `package.json`, test suite, linter, or CI config. Verify changes by opening `index.html` in a browser (or serving the directory with any static server).

## Architecture of `index.html`

- **Rendering:** a single `<canvas id="gameCanvas">` drawn each frame by `draw()`. The main loop is `loop(now)` driven by `requestAnimationFrame`; game logic is in `update(dt)`, separate from drawing.
- **Entities:** plain objects held in arrays (player, bullets, enemies, boss, power-ups, explosions). Collisions use `rectsOverlap`.
- **Tunables:** `DIFFICULTIES` (around line 485), `ENEMY_TYPES` (around line 502), and `BOSSES` (around line 508) hold balance values. Prefer editing these constants over changing logic.
- **Boss attacks:** `BOSS_PATTERNS` plus helpers such as `fireRadial`, `fireAimedSpread`, `fireCross`, and `fireHomingSeeded`. Each boss has a `hpMul` multiplier.
- **Weapons:** `player.weapon` is either `'vulcan'` or `'laser'` (toggled by a power-up in `applyPowerUp`). `fireWeapon()` handles the per-level spread.
- **Audio:** Web Audio synthesized in-browser (`beep`, `noiseBurst`, `sfx.*`). Audio is initialized lazily via `ensureAudio()` on the first user gesture; do not start audio on page load.
- **Persistence:** `localStorage` only, with keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). All reads and writes are direct, so keep them guarded for environments where storage is unavailable.
- **Accessibility:** respects `prefers-reduced-motion` (user-toggleable and persisted). The page is always dark (`color-scheme: dark`).

## Controls (for manual testing)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys / WASD | Drag in the left-side zone (joystick) |
| Steer | Mouse move over canvas | — |
| Fire | Hold Space, or hold mouse button on canvas | Hold FIRE button |
| Smart bomb | `X` or Shift | BOMB button |
| Pause | `P` or Escape | Pause button |

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a restructure. Do not introduce a build system, package manager, or external script/CDN dependencies without asking.
- Keep the existing code style: 2-space indentation, single quotes in JS, `const`/`let`, and small named helper functions.
- Preserve mobile behavior: the layout uses `touch-action: none`, `viewport-fit=cover`, and safe-area insets. Check phone-width layouts after changing HUD or control markup.
- Keep `localStorage` keys stable so existing players keep their saved best score and settings.
- When changing balance, edit the constants in `DIFFICULTIES` or `BOSSES` rather than hard-coding numbers in update logic.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-xelbx8`). Do not push to other branches without explicit permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.
