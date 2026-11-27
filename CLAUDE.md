# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-page HTML5 canvas vertical shoot-'em-up in the style of Raiden. Everything lives in one file:

- `index.html` — markup, inline `<style>`, and one inline `<script>` IIFE containing all game logic, rendering, audio, and input.
- `README.md` — placeholder title only.

There is no build step, package manager, bundler, framework, or test suite. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000
```

## Architecture notes

The script is wrapped in an IIFE with `'use strict'`. Key areas, in roughly file order:

- **DOM refs** — all `getElementById` lookups are at the top of the script. If you add an element, add its ref there.
- **Persistence** — `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes are wrapped only where the code already does so; keep new keys under the same prefix.
- **Difficulty** — `DIFFICULTIES` (easy/normal/hard) sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. `KILLS_PER_STAGE` controls wave progression before the stage boss.
- **Audio** — Web Audio via `ensureAudio()`, `beep()`, and `noiseBurst()`. Audio is created lazily after a user gesture; keep it that way.
- **Input** — keyboard (`keys[e.code]`), mouse on the canvas, and on-screen touch controls (joystick, fire, bomb). Keys: arrows/WASD move, Space fires, X or Shift bombs, P or Escape pauses.
- **Game loop** — `update(dt)` advances state; drawing helpers (`drawPlane`, `drawBossShip`, etc.) render it. Keep update and draw separate.
- **Weapons and bosses** — `fireWeapon()` handles the Vulcan/Laser system with three power levels. Boss attack patterns are small `fire*` functions (radial, aimed spread, columns, spiral, cross, homing, double ring) driven by `updateBoss(dt)`.
- **Scoring** — `addScore()` applies the kill-chain multiplier; `breakChain()` resets it.
- **Accessibility and motion** — a reduced-motion toggle exists and defaults to the `prefers-reduced-motion` media query; screen shake, flashes, and the vignette should respect it. Live announcements go through `announce()` into `srLive`.
- **Responsiveness** — `resize()` sizes the canvas to fit the viewport, including mobile orientation changes.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a split. No external network dependencies at runtime.
- Match the existing style: 2-space indent, single quotes in JS, `const` by default, small named helper functions.
- Keep the visual theme consistent: the HUD is a pill-style overlay, colors come from the existing palette (blues for sky, `#ffd54f` for values, `#7CFC00` for weapon, `#ff8a65` for multiplier).
- Do not introduce new global variables; state belongs inside the IIFE.
- Preserve the mobile layout: touch-friendly controls, `touch-action: none`, and safe-area insets.

## Making changes

- Read the surrounding code before editing; the file is long and state is shared across many functions.
- Test changes by loading `index.html` in a browser (desktop and a narrow/mobile viewport) and playing through at least one stage transition and a boss fight when touching gameplay code.
- When changing difficulty or spawn tuning, check all three difficulty levels.
- Commit messages should be short and imperative, describing the gameplay or UI change.

## Git

- Development branch for this session: `claude/gifted-tesla-ag71v3`. Push there with `git push -u origin <branch>`.
- Do not open pull requests unless explicitly asked.
