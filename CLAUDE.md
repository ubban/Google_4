# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The whole game ships as one file, `index.html` (~1,670 lines: inline CSS, markup, and one inline `<script>` IIFE). `README.md` is a stub.

There is no build step, package manager, bundler, linter, or test suite. Do not add one unless asked.

## Running the game

- Open `index.html` directly in a browser, or serve the folder with any static server, e.g. `python3 -m http.server 8000`.
- Verify behavior by actually playing it (desktop and a narrow/touch viewport), since there are no automated tests.

## File layout

| Path | Purpose |
| --- | --- |
| `index.html` | Entire game: styles, HUD/overlay markup, touch controls, and game logic |
| `README.md` | Project title only |

## Structure of `index.html`

- **Styles** (`<style>`, roughly lines 10–360): dark color scheme via `:root`, HUD pills (`#hud .stat`), hearts/bombs, vignette and flash overlays, touch controls, and a `body.reduced-motion` override.
- **Markup** (roughly lines 363–420): `#gameCanvas`, `#hud`, `#overlay` (title/start/game-over screen with difficulty buttons `.diffBtn`), `#touchControls` (joystick, FIRE, BOMB), and `#srLive` for screen-reader announcements.
- **Script** (roughly lines 422–1669): a single `(function () { 'use strict'; ... })()` that holds all state and logic. Key sections, in file order:
  - Config tables: `DIFFICULTIES` (easy/normal/hard: spawn timing, enemy speed, shot chance, lives, bombs, boss HP multiplier) and `KILLS_PER_STAGE` (18).
  - Audio: a small Web Audio synth (`beep`, `noiseBurst`, `sfx.*`), created lazily by `ensureAudio()` on user gesture.
  - Input: keyboard (`keys[e.code]`), mouse on the canvas, touch joystick and buttons.
  - Game state and loop: `resetGame`, `update(dt)`, `draw()`, `loop(now)`. Weapons are `vulcan` and `laser` (`fireWeapon`), toggled by a power-up.
  - Enemies and bosses: `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, and boss bullet patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, etc.). Stages advance after a boss is killed.
  - Player damage and end: `loseLife`, `endGame`, `togglePause`.

## Controls (keep the help text in sync)

- Desktop: mouse moves and fires (click); Arrows/WASD move; Space fires; X or Shift drops a bomb; P or Esc pauses.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

The instructions are in the `#controlsHint` element and the `#overlaySub` paragraph. Update them whenever controls change.

## Persistence

Settings and high score use `localStorage` with these keys (defined near the top of the script):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every access should keep working when storage is unavailable or throws (private mode, blocked site data). Wrap new reads and writes in `try/catch` if you add more.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Preserve the single-file, no-dependency design.
- Keep the existing style: 2-space indentation, `const`/`let`, the IIFE scope, and no new globals.
- Game tuning values live in the `DIFFICULTIES` table and in constants next to the code that uses them. Prefer changing those over hard-coding new magic numbers.
- Respect `reducedMotion` and `muted` when adding effects or sounds.
- Touch and small-screen layouts matter: the canvas scales with `resize()`, and the HUD wraps. Check phone-width layout after UI changes.
- Accessibility: announce important state changes through `announce()` / `#srLive`.

## Git

- Development branch for this work: `claude/gifted-tesla-uz573k`. Commit and push there (`git push -u origin <branch>`).
- Write clear, descriptive commit messages in the imperative mood.
