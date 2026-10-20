# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is one self-contained file, `index.html`, with inline `<style>` and a single inline `<script>` that draws everything on a `<canvas>`.

Other files:
- `README.md`: placeholder project readme.
- `CLAUDE.md`: this file.

There is no package manager, build step, bundler, test suite, or linter configured. Do not invent npm scripts or test commands. If you add tooling, document it here.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server` from the repo root. No install step is needed.

To verify a change, load the game in a browser and play through the affected path. Check the desktop controls, and check touch controls if the change touches input, layout, or the HUD.

## Code layout (`index.html`)

- **Markup** (about lines 1–361 are `<head>` and CSS): viewport meta disables user scaling and uses `viewport-fit=cover`. Colors and layout are in the `<style>` block.
- **DOM** (about lines 361–421): `#container` holds the canvas, the `#hud`, the `#topBar` buttons (mute, reduced motion, pause), the `#touchControls` (joystick and FIRE/BOMB), and the `#overlay` start/game-over card. A visually hidden `#srLive` region announces events to screen readers.
- **Script** (about lines 422–1667): one IIFE with `'use strict'`. Sections are marked with `// ---------- Name ----------` comments. Keep that convention when adding a section.

Main sections in the script, in order:
1. DOM references and storage keys
2. Adaptive performance (`quality` 0–2, lowered when frames run slow)
3. Tuning tables: `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`
4. Audio (Web Audio API via `ensureAudio`, `beep`, `noiseBurst`)
5. HUD and feedback helpers (hearts, bombs, combo pop, stage banner, screen shake)
6. Resizing: logical resolution is `BASE_W = 480` x `BASE_H = 720`, scaled to fit the window
7. Input: keyboard, touch joystick, fire and bomb buttons, mouse steering on the canvas
8. Game state: `resetGame`, `update(dt)`, spawning, collisions, weapons, power-ups, bombs, chain/score
9. Boss system: `BOSS_PATTERNS` (bullet patterns), `updateBoss`, `killBoss`
10. Lifecycle: `loseLife`, `endGame`, `togglePause`
11. Rendering: `draw()` and the `drawPlane` / `drawBossShip` / cloud helpers
12. `loop(now)` driven by `requestAnimationFrame`, with `dt` capped at 0.05 s

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. If you split files, update the layout notes above.
- Match the existing style: 2-space indent, single quotes, `const` by default, `let` for mutable state, UPPER_SNAKE_CASE for tuning constants, and short comments that explain intent.
- Gameplay numbers live in the tuning tables near the top of the script. Prefer changing those over hard-coding values in update or draw logic.
- Use `update(dt)` for simulation and `draw()` for rendering. Keep them separate. Frame-rate-dependent behavior should scale with `dt`.
- Any new user-facing state change should be announced through `announce(text)` so screen readers hear it.

## Persistence

The game stores four values in `localStorage`. Use the same key prefix for any new key.

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score |
| `skyfighter.muted` | `'1'` if muted |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; defaults to the `prefers-reduced-motion` media query when unset |

Wrap new storage access so the game still works if `localStorage` throws, such as in private browsing.

## Controls (keep the start-screen hint in sync)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move over the canvas to steer, click to fire.
- Touch: drag on the left side to fly, tap FIRE, tap BOMB.

If you change a binding, update the `#controlsHint` text in the markup too.

## Accessibility and responsiveness

- Respect the reduced-motion setting. Screen shake, hit flashes, and the low-health pulse should be suppressed or softened when it is on.
- The layout must work on phones: keep the touch controls reachable and avoid horizontal scrolling.
- Keep interactive buttons labeled. Use `title` or visible text, and keep `aria-live` announcements for important events.

## Git workflow

- Develop on the branch assigned for the session. Do not push to other branches without permission.
- Write clear commit messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.
