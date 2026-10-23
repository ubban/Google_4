# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game (markup, CSS, and JavaScript) lives in one file, `index.html`. `README.md` is a placeholder.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless the user asks for it.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Because there is no tooling, verification means loading the page in a browser and playing it. Check the canvas, HUD, and controls after any change.

## File layout (`index.html`)

- **Lines ~1–359**: `<head>` meta tags, the inline favicon (SVG data URI), and the `<style>` block. HUD and canvas styles are here.
- **Lines ~361–421**: `<body>` markup. A `#gameCanvas` element, the HUD, on-screen control buttons (`pauseBtn`, `bombBtn`, `fireBtn`, `muteBtn`, `motionBtn`), and overlay screens (`startBtn` starts the game).
- **Lines ~422–1667**: one inline `<script>` with the game logic, organized roughly in this order:
  - Storage keys, settings, and the audio helpers (`ensureAudio`, `beep`, `noiseBurst`, `sfx`).
  - Difficulty, theme, enemy, and boss configuration tables (`DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`).
  - HUD helpers (`renderHearts`, `renderBombs`, `showStageBanner`, `announce`).
  - Input handling: keyboard listeners, joystick, and mouse/touch-to-canvas mapping.
  - Game state and reset (`resetGame`), spawning, bullets, bombs, and power-ups.
  - Boss attack patterns (`BOSS_PATTERNS`) and boss lifecycle (`updateBoss`, `killBoss`).
  - Update (`update`), rendering (`draw`, `drawPlane`, `drawBossShip`, `drawClouds`), and the main `loop`.

Line numbers are approximate and will drift. Search for the function or constant name instead of relying on them.

## Gameplay and configuration reference

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rates, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, then a boss fights. Stage backgrounds cycle through `THEMES` by `(stage - 1) % THEMES.length`.
- **Enemies** (`ENEMY_TYPES`): `grunt`, `interceptor` (weaves), and `tank` (4 HP, appears from stage 2).
- **Bosses** (`BOSSES`, `BOSS_PATTERNS`): six bosses, each with its own hull colors, HP multiplier, and attack patterns.
- **Power-ups**: `life`, `bomb`, and `weapon`. Weapon upgrades cycle through levels 1–3, then switch between `vulcan` and `laser`.

### Controls

| Action | Keyboard | Mouse | Touch |
| --- | --- | --- | --- |
| Move | Arrow keys or WASD | Mouse moves the ship toward the pointer | Virtual joystick |
| Fire | Hold Space | Hold mouse button on canvas | Hold on-screen Fire button |
| Smart bomb | X or Shift | On-screen Bomb button | On-screen Bomb button |
| Pause | P or Escape | On-screen Pause button | On-screen Pause button |

Mute and motion-reduction toggles are on-screen buttons (`muteBtn`, `motionBtn`).

## Persistence

The game uses `localStorage` with keys that all start with `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every `localStorage` access can throw (private mode, blocked storage). Keep reads and writes defensive, and make sure the game still runs without them. If you add a new setting, give it a `skyfighter.`-prefixed key.

## Conventions

- **Vanilla JavaScript only.** No frameworks or external dependencies. The page loads no remote scripts.
- **Keep it one file.** Put new markup, styles, and script into `index.html` unless the user asks for a split.
- **Match the existing style.** Two-space indentation, `const` and `let` (no `var`), arrow functions for event handlers, and short, descriptive helper names.
- **Accessibility:** route status messages through `announce()` so screen readers pick them up. Respect the `reducedMotion` setting when adding screen shake, flashes, or other motion effects.
- **Performance:** the loop tracks frame time with `trackPerf` and steps through `quality` tiers (full, medium, low). New visual effects should degrade under the lower tiers instead of dropping frames.
- **Input:** the game calls `preventDefault` on Space to stop page scrolling. Keep new key bindings from clashing with the existing ones listed above.

## Making changes

- Make small, targeted edits. Avoid restructuring the single-file layout unless asked.
- After a gameplay change, test it in a browser: start a game, play through at least one kill streak, use a bomb, and check that pause, game over, and restart still work. Check the layout at phone width as well as desktop, since the HUD and canvas scale with the viewport.
- Update the **Gameplay and configuration reference** section of this file if you change difficulty values, controls, stage rules, or storage keys.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-3yr70j`). Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages.
- Do not open a pull request unless the user explicitly asks for one.
