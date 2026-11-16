# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only app is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs in a single HTML file with no
build step, no package manager, and no external dependencies.

| File        | Purpose                                                    |
|-------------|------------------------------------------------------------|
| `index.html`| The whole game: inline CSS (`<style>`), markup, and JS (`<script>`) |
| `README.md` | Minimal project readme                                     |

There is no test suite, linter, or CI configured in the repo.

## Running the game

No install step is needed. Open `index.html` directly in a browser, or serve
the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code layout inside `index.html`

- **`<style>` (about lines 10–359):** dark color scheme (`color-scheme: dark`),
  HUD pills, vignette/flash overlays, boss bar, stage banner, touch controls,
  and a `reduced-motion` body class that disables animations.
- **Markup (about lines 360–421):** `#gameCanvas`, HUD elements (`#score`,
  `#best`, `#mult`, `#weaponName`, `#hearts`, `#bombs`), overlays, a hidden
  `#srLive` live region for screen readers, and touch controls (`#moveZone`,
  `#fireBtn`, `#bombBtn`).
- **`<script>` (about lines 422–1667):** a single IIFE (`'use strict'`) that
  holds all game state. Key sections, roughly in file order:
  - Config constants: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES`,
    `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
  - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table.
  - Input: keyboard handlers, mouse steering/firing, touch joystick and buttons.
  - Game logic: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`,
    `fireWeapon`, `useBomb`, `applyPowerUp`, `updateBoss`, `killBoss`,
    `loseLife`, `endGame`, `togglePause`.
  - Rendering: `draw()` and helpers such as `drawPlane`, `drawBossShip`,
    `drawClouds`.
  - Main loop: `loop(now)`, driven by `requestAnimationFrame`.

## Game rules and tuning (source of truth: `index.html`)

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets
  spawn rates, enemy shoot chance, starting lives, starting bombs, and boss HP
  multiplier. The choice persists in `localStorage`.
- **Enemies:** `grunt` and `interceptor` appear from stage 1; `tank` (4 HP)
  appears from stage 2.
- **Stages:** a stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a
  boss fight. Bosses cycle through `BOSSES` by stage, and their HP scales with
  stage number. Stage backgrounds cycle through `THEMES`.
- **Power-ups:** `life`, `bomb`, and `weapon` (toggles `vulcan` / `laser`).
- **Controls:** mouse, arrows/WASD, or touch to move; Space or click/fire button
  to shoot; X or Shift to bomb; P or Esc to pause.

## Persistence

Settings and progress are stored in `localStorage` under these keys. Keep the
names stable, because changing them resets players' saved data:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

## Conventions

- Keep the project dependency-free and single-file unless the user asks for a
  restructure. Do not add a build step or framework.
- Preserve accessibility hooks: keep the `#srLive` announcements (via
  `announce()`) and the `reduced-motion` handling. Add `aria-label`s to any new
  controls; the current buttons do not have them yet.
- Preserve mobile support: the viewport meta, `touch-action: none`, and the
  touch control layer must keep working at phone widths.
- Keep the game loop frame-rate independent by using the `dt` value passed into
  `update`.
- Use the adaptive `quality` level when adding expensive visual effects, so
  they scale down on slow devices.

## Verifying changes

Since there are no automated tests, verify changes by opening `index.html` in
a browser (desktop and a mobile-sized viewport) and playing through at least
one stage boss, one bomb use, and one game over. Check the browser console for
errors.
