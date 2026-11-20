# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — minimal project readme.

There is no package manager, build step, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000/`.

## Code layout inside `index.html`

- **`<style>`** (around lines 10–420): HUD pills, overlay card, control buttons, vignette, and keyframe animations. Dark color scheme only (`color-scheme: dark`).
- **`<body>`** markup: `#container`, `#gameCanvas`, `#hud`, overlay/title screen, pause and control buttons, and the `#srLive` live region for screen readers.
- **`<script>`** (starts around line 422): wrapped in a single IIFE. Sections, in order:
  1. DOM lookups and `STORAGE_*` localStorage keys (`skyfighter.*`).
  2. Adaptive performance (`trackPerf` adjusts a `quality` level 0–2 from frame times).
  3. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
  4. Audio (`ensureAudio`, `beep`, `noiseBurst`), which is synthesized with WebAudio. No audio files.
  5. Input: keyboard, touch joystick (`joystickStart/Move/End`), fire and bomb buttons, and mouse steering on the canvas.
  6. Game state (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`) and `resetGame()`.
  7. Spawning and combat: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `addScore`/`breakChain`.
  8. `update(dt)` (game logic), `draw()` and the `draw*` helpers (rendering), then `loop(now)` driving both via `requestAnimationFrame`.

## Conventions

- **Keep it one file.** New game code goes in the existing `<script>` IIFE and `<style>` block unless the user asks for a split.
- **Use `const` for tuning data** and keep balance numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- **Time-based movement.** `update(dt)` receives seconds, clamped to 0.05 s in `loop`. Multiply per-frame motion by `dt`.
- **Persistence goes through the `STORAGE_*` keys** and must not throw. Wrap new `localStorage` access in `try/catch` if it could run in a private window or with blocked storage.
- **Respect reduced motion.** Screen shake, flashes, and similar effects should check `reducedMotion` (preference from `prefers-reduced-motion`, overridable in-game).
- **Mobile and desktop both matter.** Preserve the touch controls (`#moveZone`, `#fireBtn`, `#bombBtn`) and the `viewport-fit=cover` / safe-area handling when changing layout.
- **Accessibility.** Announce important state changes through `announce()` (the `#srLive` region).
- **Match existing style:** 2-space indent, single quotes in JS, short comments that explain intent.

## Verifying changes

There is no automated test suite. After changing `index.html`:

1. Load the page in a browser and check the console for errors.
2. Exercise the affected path: start a game, fire, use a bomb, pause/resume, reach a boss if you touched boss logic, and change difficulty.
3. Check the HUD and overlay at a phone-sized viewport as well as desktop.
4. If a headless browser is available, a short Playwright script that loads the page and checks for `pageerror` events is a quick smoke test.

Say explicitly in your summary which of these you ran and which you could not.

## Git

- Development branch for this work: `claude/gifted-tesla-1fc8k3`. Push there, not to other branches, unless the user says otherwise.
- Do not open a pull request unless the user asks for one.
