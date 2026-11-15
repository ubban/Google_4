# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-page HTML5 canvas app.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one file (~1,670 lines). |
| `README.md` | One-line project title. |

There is no build system, package manager, bundler, test suite, linter, or CI configuration. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000
```

No install step is needed.

## Structure of `index.html`

The file has three sections, in this order:

1. **`<style>`** (starts around line 10): all styling. The HUD is pill-shaped, and the start/pause overlay is a glassmorphic card. The `prefers-reduced-motion` and in-game "reduced motion" toggle both affect screen shake, flash, and pulse effects.
2. **Markup** (around lines 362–420): `#container` holds the `<canvas id="gameCanvas">`, the HUD (`#hud`), boss health bar, stage banner, top bar buttons (mute, reduced motion, pause), touch controls (`#touchControls`), and the `#overlay` start/game-over card.
3. **`<script>`** (starts around line 422): all game logic, with no modules and no external dependencies.

Key parts of the script, in roughly the order they appear:

- **Persistence keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`. All are `skyfighter.*` strings in `localStorage`. Reads and writes are direct; keep them that way unless adding a storage helper.
- **Tuning tables**: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BOSS_PATTERNS`. Most balance changes belong in these tables, not in the update loop.
- **Audio**: a Web Audio API layer (`ensureAudio`, `beep`, `noiseBurst`, and per-enemy `EXPLODE_SFX`). Audio is created lazily, after a user gesture, so browsers allow it.
- **Input**: keyboard (`keydown`), mouse (canvas `mousemove`/`mousedown`, plus `fireBtn` mouse events), and touch (a left-side virtual joystick in `moveZone`, plus FIRE and BOMB buttons).
- **Game state and loop**: `resetGame()` initializes state, `update(dt)` advances it, `draw()` renders it, and `loop(now)` drives both through `requestAnimationFrame`. `trackPerf` monitors frame time.
- **Gameplay systems**: weapons (`fireWeapon`, with Vulcan and Laser at three power levels), smart bombs (`useBomb`), power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), a kill-chain score multiplier (`addScore`, `breakChain`), stages (`KILLS_PER_STAGE` kills per stage, then `spawnBoss`), and lives (`loseLife`, `endGame`, `togglePause`).
- **Rendering helpers**: `drawPlane`, `drawBossShip`, `drawClouds`, `spawnExplosion`, plus the DOM-based effects `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, and `screenShake`.

The canvas is a fixed logical size (`BASE_W = 480`, `BASE_H = 720`) that `resize()` scales to the viewport. Game coordinates should always be expressed in this logical space.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce a framework, bundler, or npm dependencies.
- **Vanilla JS, ES2015+ syntax.** Match the existing style: `const`/`let`, arrow functions, template strings, and semicolons at line ends.
- **Naming.** Constants are `UPPER_SNAKE_CASE`. Functions and variables are `camelCase`. Storage keys are prefixed `skyfighter.`.
- **Keep the frame loop allocation-light.** Avoid creating objects or closures inside `update`/`draw` on every frame where a reused structure works. Time-based motion uses the `dt` passed to `update`.
- **Accessibility.** Preserve the `aria-live` announcer (`srLive` / `announce`), `title`/labels on icon buttons, and the reduced-motion path. Any new visual shake or flash must respect `reducedMotion`.
- **Mobile.** Touch controls must keep working. Preserve `preventDefault` on the touch handlers for FIRE and BOMB, which is intentional.

## Making changes

- Balance tweaks (speeds, spawn rates, boss patterns, score values) should go in the tuning tables where possible.
- After editing, test by loading the page in a browser and checking at least: starting a game, firing, collecting a power-up, using a bomb, reaching a boss, pausing, muting, and switching difficulty. Confirm that the best score persists across reloads.
- Check the layout at phone width (the touch controls and overlay) as well as desktop width.
- Keep the Git history clean: one logical change per commit, with a descriptive message.

## Things to avoid

- Do not add new `localStorage` access without a `try/catch`. Storage can throw in private windows or when site data is blocked, and the game must still run without it.
- Do not change the logical canvas size (`BASE_W`/`BASE_H`) casually. Positions, enemy spawn lanes, and boss layouts all assume it.
- Do not add analytics, network calls, or third-party scripts to the game.
- Do not commit generated or temporary files. The repo currently contains only source files.
