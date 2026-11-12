# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The entire game lives in a single file, `index.html` (~1,670 lines), with inline `<style>` and `<script>` blocks. `README.md` is a placeholder.

There is no build step, package manager, test suite, or linter configured.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server` and visit `http://localhost:8000`).
- No install is required.

## File layout

- `index.html` — the whole game:
  - `<style>` (lines 10–359): dark-theme styles, HUD, touch controls, and the canvas layout.
  - `<body>` (lines 361–421): the `#gameCanvas` element, HUD elements, touch controls, and the `#controlsHint` help text.
  - `<script>` (lines 422–1667): all game logic, rendering, input, and audio.
- `README.md` — project name only.

## Code architecture (inside `index.html`)

The script is a single `(function () { ... })();` block with `const`/`function` declarations; there are no modules. Major sections, in rough order:

- **Persistence keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` use the `skyfighter.*` prefix in `localStorage`.
- **Accessibility helpers** — `announce()` writes to a live region for screen readers; `prefers-reduced-motion` is read via `matchMedia` and can be toggled in-game.
- **Config tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `EXPLODE_SFX`.
- **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, `panFor()`. Audio is created lazily on first user input (browser autoplay rules).
- **HUD** — `renderHearts()`, `renderBombs()`, `flashHit()`, `showStageBanner()`, `showComboPop()`.
- **Layout and input** — `resize()` scales a fixed `BASE_W` canvas; keyboard state is kept in `keys[e.code]`; touch uses `joystickStart/Move/End` and `canvasPointFromEvent()`.
- **Game state** — `resetGame()`, `setDifficulty()`, `endGame()`, `togglePause()`.
- **Entities and rendering** — `drawPlane()`, `drawBossShip()`, `spawnEnemy()`, `spawnBoss()`, `spawnExplosion()`, `maybeSpawnPowerUp()`, `applyPowerUp()`.
- **Weapons** — `fireWeapon()` dispatches to pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
- **Bosses** — `updateBoss()` drives phase logic from `BOSS_PATTERNS`; `killBoss()` handles defeat.
- **Main loop** — `update(dt)` advances the simulation; `loop` is driven by `requestAnimationFrame`, and `trackPerf(dt)` tracks frame timing.

### Controls

| Action | Keyboard | Mouse | Touch |
| --- | --- | --- | --- |
| Move | Arrow keys / WASD (read in `update()` from `keys`) | Move pointer over canvas (`mousemove`) | Drag left side (`moveZone`) |
| Fire | `Space` (held) | Hold button on canvas (`mousedown`/`mouseup`) | `#fireBtn` |
| Smart bomb | `X` or either `Shift` (`useBomb()`) | — | `#bombBtn` |
| Pause | `P` or `Escape` (`togglePause()`) | — | — |

When changing controls, update the keyboard handler (`window.addEventListener('keydown', ...)`), the touch controls, and the `#controlsHint` text in `index.html`.

## Conventions

- **Keep it a single file** unless the user asks for a split. Don't introduce a bundler, framework, or npm dependencies without discussion.
- **Match existing style:** 2-space indentation, single quotes in JS, `const` by default, small named helper functions, and comments that use `// ---------- Section ----------` dividers.
- **Constants over magic numbers:** tune gameplay through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`) rather than hard-coding values in update logic.
- **Persist only through the `skyfighter.*` keys** in `localStorage`, and wrap storage access so the game still works when storage is unavailable (private mode, blocked site data).
- **Accessibility:** respect `prefers-reduced-motion` for screen shake and flashes, and announce important state changes through `announce()`.
- **Mobile:** the viewport disables zoom and the page uses `touch-action: none`; keep layouts working with `env(safe-area-inset-*)` values and the touch joystick.

## Testing and verification

No automated tests exist. To verify a change:

1. Open `index.html` in a browser (or a static server) and check the console for errors.
2. Play through at least one full stage and one boss fight, covering weapon pickup, bomb use, pause/resume, and game over.
3. Check the touch layout with a narrow viewport or device emulation.
4. Confirm `localStorage` persistence (best score, mute, difficulty, reduced motion) across reloads.

If you add tests or tooling, document the commands here.

## Git workflow

- Default working branch for this environment: `claude/gifted-tesla-xdwikr`.
- Commit with clear, imperative messages describing the gameplay or code change.
- Don't open pull requests unless explicitly asked.
