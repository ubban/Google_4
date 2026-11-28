# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up.

- The entire game lives in `index.html` (~1,670 lines): inline `<style>`, HTML markup for the HUD and overlays, and one inline `<script>` wrapped in an IIFE.
- There is no build step, no package manager, no bundler, and no test suite. Open the file in a browser to run it.
- `README.md` is a placeholder and contains only the project title.

## Repository layout

```
.
├── index.html   # The whole game: CSS, markup, and JavaScript
├── README.md    # Placeholder
└── CLAUDE.md    # This file
```

## Running the game

No install is needed. Either:

- Double-click `index.html`, or
- Serve the directory with any static server, for example `python3 -m http.server 8000`, then open `http://localhost:8000/`.

For a headless check, Playwright with the pre-installed Chromium works. Load the page, then inspect the canvas or console output.

## Code structure (`index.html`)

The script is a single IIFE. Key sections, in file order:

| Area | Notes |
| --- | --- |
| Storage keys | `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`. |
| `DIFFICULTIES` | Per-difficulty tuning: spawn rate, enemy speed, enemy fire chance, lives, bombs, boss HP multiplier. Values: `easy`, `normal` (default), `hard`. |
| `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` | Stage progression and enemy/boss definitions. |
| Audio | Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`) that synthesize sound effects. No audio files. |
| Input | Keyboard handlers on `window`, plus a touch joystick and pointer handling for the canvas. |
| Game state | `resetGame()` builds fresh state. `update(dt)` advances the simulation each frame. |
| Weapons and bombs | `fireWeapon()`, `useBomb()`, `applyPowerUp()`. |
| Boss patterns | `BOSS_PATTERNS` and the `fire*` helpers (radial, spread, spiral, cross, homing, double ring). |
| Rendering | `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`). Canvas 2D. |
| Loop | `loop(now)` drives `update` and `draw` via `requestAnimationFrame`. |

### Canvas

- Logical resolution is `BASE_W = 480` by `BASE_H = 720`. `resize()` scales the canvas to fit the viewport.
- Keep gameplay coordinates in logical units, not CSS pixels.

### Controls

- Move: arrow keys or `WASD`. Touch devices get an on-screen joystick.
- Fire: `Space` (hold), or pointer/touch hold on the canvas.
- Bomb: `X` or either `Shift`.
- Pause: `P` or `Escape`.

## Conventions

- **Keep it one file.** Do not split `index.html` into modules, add a build step, or add dependencies unless the user asks. The no-build layout is deliberate.
- **No external assets.** Graphics are drawn with the Canvas 2D API and audio is synthesized with Web Audio. Do not add image, sound, or font files, or CDN links, without asking.
- **Preserve the IIFE.** Top-level names should stay inside the IIFE so the page adds no globals.
- **Respect reduced motion.** The `reducedMotion` setting dampens screen shake (`screenShake`) and skips some flash effects. New visual effects that move the screen or flash should honor it.
- **Respect mute.** Every new sound must go through the existing audio helpers so the mute toggle covers it.
- **Persist settings through the existing keys.** Use the `skyfighter.*` `localStorage` keys. Wrap storage access so the game still runs when storage is unavailable, as it does now.
- **Touch and small screens.** The layout must work on phones: keep `touch-action` and the viewport meta intact, and check HUD wrapping at narrow widths.
- **Match the existing style.** Two-space indentation in the `<style>` and `<script>` blocks, single quotes in JavaScript, `const` and `let` (no `var`), and short descriptive function names.

## Making changes

1. Read the relevant section of `index.html` before editing. The file is large, so search by function name rather than reading it all.
2. Balance changes belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `BOSS_PATTERNS`. Prefer changing those tables over hard-coding numbers in the update logic.
3. After a gameplay change, run the game in a browser and check at least: a full play-through start, a stage transition, a boss fight, a bomb, a power-up, pause/resume, and game over. Check both a desktop viewport and a phone-sized viewport.
4. Confirm there are no console errors on load or during play.

## Git

- Development branch: `claude/gifted-tesla-lzoshu`. Do not push to other branches without explicit permission.
- Remote: `origin` at `https://github.com/ubban/Google_4`.
- Do not open a pull request unless the user asks for one.
- Commit messages: short imperative subject line, with a body when the reason is not obvious from the diff.
