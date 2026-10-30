# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas app.

- `index.html` is the entire game: markup, CSS, and JavaScript in one file (~1,670 lines). There is no build step, no package manager, and no test suite.
- `README.md` is a placeholder and contains only the repo title.
- Git branch: work lands on the designated feature branch (currently `claude/gifted-tesla-2rdwm3`). Do not push to other branches without explicit permission.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout inside `index.html`

- **`<style>`**: HUD pills, overlay card, touch controls, vignette/flash effects, and keyframes. Responsive sizing uses `clamp()` and `vw`, and `env(safe-area-inset-*)` handles notched devices.
- **`<body>` markup**: canvas `#gameCanvas`, HUD (`#hud`), boss bar, stage banner, top-bar buttons (mute, reduced motion, pause), touch controls, and the start/pause overlay (`#overlay`).
- **`<script>`**: one IIFE. Sections, in rough order:
  - DOM references and `STORAGE_*` keys (persisted in `localStorage` under `skyfighter.*`)
  - `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`: tunable game data, kept near the top
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table, all built on Web Audio
  - HUD helpers: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`
  - Input: keyboard, mouse, and touch joystick handlers (`joystickStart/Move/End`, `canvasPointFromEvent`)
  - Game flow: `resetGame`, `update(dt)`, `draw()`, `loop(now)`, `loseLife`, `endGame`, `togglePause`
  - Weapons and bosses: `fireWeapon`, `useBomb`, `applyPowerUp`, the `fire*` boss pattern functions, `updateBoss`, `killBoss`
  - Rendering helpers: `drawPlane`, `drawBossShip`, `drawClouds`

Keep changes inside the existing structure. Do not split the file into modules unless the user asks for it.

## Game mechanics worth knowing

- **Difficulty** (`DIFFICULTIES`) sets spawn rate, enemy shooting chance, starting lives, starting bombs, and boss HP multiplier. Easy gives 4 lives and 3 bombs; Normal and Hard give 3 lives and 2 bombs.
- **Stages** advance after `KILLS_PER_STAGE` (18) kills. Each stage ends with a boss, using patterns from `BOSSES` and the `fire*` functions.
- **Weapons**: Vulcan and Laser (toggled by a power-up), each with a weapon level. Power-ups can raise the level, add a bomb, or add an extra life (capped at `maxLives`).
- **Score**: a kill-chain multiplier builds during combos and resets via `breakChain`.
- **Persistence**: best score, mute, difficulty, and reduced-motion preference are saved to `localStorage`. Reads use `|| default` fallbacks; keep that pattern if you add new keys.

## Conventions

- Plain ES2015+ JavaScript and vanilla DOM. Do not add frameworks or dependencies.
- Use `const`/`let`, camelCase names, and single quotes for strings, matching the existing code.
- Game-time logic uses the `dt` passed into `update(dt)`. Do not use wall-clock time for gameplay.
- Respect the reduced-motion setting (`reducedMotion`) for any new screen shake, flashes, or pulsing effects.
- Keep touch and keyboard controls in parity. If you add an action, wire it to both (keyboard handlers, mouse/touch handlers, and the on-screen buttons).
- Browsers block audio until a user gesture; call `ensureAudio()` before playing sound, and check `muted` before playing.

## Verifying changes

There is no automated test suite or linter. To verify a change:

1. Open `index.html` in a browser (desktop and, if you touched input or layout, a narrow mobile viewport).
2. Play through at least one stage, including a boss fight, and check the HUD updates.
3. Check the pause (`P`/`Esc` on keyboard, the ⏸ button otherwise), mute, and reduced-motion toggles.
4. Confirm best score persists after a reload.

If you can run a headless browser, a quick smoke test that loads the page and checks for console errors is a good baseline.

## Git and pull requests

- Commit messages: short imperative summary line, followed by a body that explains the why when it is not obvious.
- Do not open a pull request unless the user asks for one.
- Keep commits focused. Do not mix gameplay tuning with visual or refactoring changes in one commit.
