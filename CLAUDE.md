# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

Files:

- `index.html` — the whole game: markup, CSS, and JavaScript in one file (~1,670 lines).
- `README.md` — a one-line title. Nothing else documents the project.

There is no build system, package manager, bundler, test suite, linter, or CI config. Do not add any of these unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000
```

There are no install or build steps.

## Architecture of `index.html`

The file has three parts, in order:

1. **`<style>`** (about lines 10–360) — all CSS. HUD, overlay card, touch controls, and the `.sr-only` helper for screen-reader text.
2. **`<body>`** (about lines 361–421) — static DOM. Key IDs: `gameCanvas`, `hud` (`score`, `best`, `mult`, `weaponName`, `weaponLvl`, `hearts`, `bombs`), `bossBarWrap`, `comboPop`, `stageBanner`, `topBar` (`muteBtn`, `motionBtn`, `pauseBtn`), `srLive`, `touchControls` (`moveZone`, `joystickBase`, `fireBtn`, `bombBtn`), and `overlay` (`startBtn`, `diffRow`, `statsRow`).
3. **Inline `<script>`** (about lines 422–1668) — one IIFE, `(function () { ... })();`. Everything is private to the closure. Nothing is exposed on `window`.

Inside the IIFE, the code is grouped roughly like this:

- **Config tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
- **Persistence** — `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read currently assumes storage works.
- **Audio** — a lazily created `AudioContext` (`ensureAudio`) with procedural `beep` and `noiseBurst` effects. Nothing is loaded from disk.
- **Input** — keyboard (`keys[e.code]`), touch joystick plus fire/bomb buttons, and mouse steering and firing on the canvas. Keyboard and mouse controls are listed in `#controlsHint`.
- **Game state** — module-level `let` variables (`score`, `lives`, `stage`, `chain`, `multiplier`, `running`, `paused`, `gameOver`, and so on). `resetGame()` initializes them.
- **Entities and spawning** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`.
- **Weapons and boss patterns** — `fireWeapon` and the `fire*` helpers (`fireRadial`, `fireAimedSpread`, `fireCross`, and others). Boss attacks are data-driven through `BOSS_PATTERNS`.
- **Update and draw** — `update(dt)` runs the simulation, `draw()` renders it, and `loop(now)` drives both through `requestAnimationFrame`. `dt` is clamped to 0.05 s.
- **Adaptive quality** — `trackPerf(dt)` lowers a `quality` level (0 full, 1 medium, 2 low) when frames are slow.
- **Accessibility** — `announce()` writes to `#srLive`. Reduced motion (`reducedMotion`, saved under `skyfighter.reducedMotion`, defaulting to the `prefers-reduced-motion` setting) suppresses hit/bomb flashes and shrinks screen shake.

## Conventions

- **Keep the single-file structure.** Put new CSS in `<style>`, markup in `<body>`, and code inside the existing IIFE. Do not split into modules or separate files unless asked.
- **Use plain browser JavaScript.** Use `const` and `let`, arrow functions, and template literals as the existing code does. Do not add a framework, a bundler, or a CDN dependency.
- **Match the local style.** Use 2-space indentation, single quotes in JS, and short one-line helpers where the surrounding code does. Add comments only where the logic is not obvious, which matches the existing density.
- **Tune numbers in the config tables.** Difficulty, enemy stats, theme colors, and boss HP multipliers belong in `DIFFICULTIES`, `ENEMY_TYPES`, `THEMES`, and `BOSSES`. Avoid scattering magic numbers through the update loop.
- **Use the existing `localStorage` keys** and prefix any new key with `skyfighter.`.
- **Keep the game working without storage.** Reads and writes to `localStorage` can throw in private windows or when site data is blocked. Wrap new storage calls in `try`/`catch` and fall back to in-memory values.
- **Keep the game playable on phones.** Any new control must work with touch, and the layout must not scroll horizontally.

## Verifying changes

No automated tests exist. Verify by hand:

1. Open `index.html` in a browser (or serve it locally) and check the browser console for errors.
2. Play through the start, a stage transition, a boss fight, a bomb, a pause/resume, and game over.
3. Check each difficulty (Easy, Normal, Hard) and each mute and reduced-motion toggle.
4. For layout changes, check a narrow viewport and a touch-emulated viewport.

If you add a test harness or tooling, document the commands here.

## Git

- Work on the branch you were assigned. Do not push to `main` unless asked.
- Write short, descriptive commit messages that say what changed and why.
