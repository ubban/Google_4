# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`: markup, CSS, and one inline `<script>` block. There is no build step, no package manager, no bundler, and no test suite. `README.md` is a placeholder.

## Running the game

Open `index.html` directly in a browser. No server is required. For a quick local server:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There are no automated tests or linters configured. Verify changes by playing the game in a browser, including on a narrow/touch viewport when touching input or layout.

## File layout (`index.html`)

- **`<head>`**: viewport meta (locks zoom, `viewport-fit=cover`), `theme-color`, title.
- **`<style>`** (lines 10–359): all styling, including the `.reduced-motion` body class and touch-control layout.
- **HTML body** (from about line 363): `#gameCanvas`, the HUD (score, best, multiplier, weapon, hearts, bombs, boss bar), the `#overlay` start/pause/game-over screen, the `#touchControls` joystick and fire/bomb buttons, and a hidden `#srLive` region for screen-reader announcements.
- **`<script>`** (from about line 422): one IIFE wrapping everything, with `'use strict'`. Nothing is exported to `window`.

Within the script, the code is grouped under section comments such as `// ---------- Adaptive performance ----------`. Keep new code in the matching section, and add a new section comment when adding a new subsystem.

### Main script subsystems

| Area | Key identifiers |
|---|---|
| Config tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS` |
| Persistence | `STORAGE_*` constants, `bestScore`, `muted`, `difficulty`, `reducedMotion` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx`, `EXPLODE_SFX` (Web Audio, synthesized; no audio files) |
| Input | `keys` (keyboard by `e.code`), joystick helpers, `firing`, mouse handlers on the canvas |
| Game state | `resetGame`, `update(dt)`, `draw()`, `loop(now)` |
| Entities/spawning | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `spawnExplosion`, `applyPowerUp` |
| Weapons | `fireWeapon`, `useBomb`, `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing` |
| Bosses | `updateBoss`, `killBoss`, `BOSS_PATTERNS` |
| Lifecycle | `loseLife`, `endGame`, `togglePause` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw` |
| Adaptive quality | `quality` (0 full / 1 medium / 2 low), `trackPerf`, driven by a 45-frame rolling average |

## Controls

- **Keyboard:** Arrow keys or WASD to move, Space to fire (it is `preventDefault`-ed), X or Shift for a smart bomb, P or Escape to pause.
- **Touch:** left-side joystick zone to move, fire button, bomb button. The touch layer is enabled only when `ontouchstart` or `maxTouchPoints` indicates a touch device.
- **Mouse:** moving over the canvas steers the ship; mouse-down fires.

## Persistence

All state is stored in `localStorage` under these keys. Keep the names stable, because they are player data.

| Key | Value |
|---|---|
| `skyfighter.bestScore` | number |
| `skyfighter.muted` | `'1'` or `'0'` |
| `skyfighter.difficulty` | `easy`, `normal`, or `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; when absent, follows `prefers-reduced-motion` |

`localStorage` can throw or be unavailable (private mode, blocked storage). The current reads and writes do not guard against this, so if you add a new one, wrap it in `try/catch` and make sure the game still runs without it.

## Conventions

- **Single file.** Do not split `index.html` into modules, add a build step, or add dependencies unless the user asks. Everything should keep working by opening the file directly.
- **Game-loop timing.** `dt` is clamped to 50 ms in `loop`. Movement and timers should scale with `dt` (seconds), not frame counts. `frame` is a counter used for patterns that intentionally tick per frame.
- **Fixed logical resolution.** The canvas is always `480×720` (`BASE_W`, `BASE_H`). `resize()` only changes CSS size, so write draw and collision code in these logical coordinates.
- **Tuning lives in tables.** Prefer adding or adjusting values in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, and `BOSS_PATTERNS` over hard-coding numbers in logic.
- **Audio is synthesized.** Use the existing `beep`/`noiseBurst` helpers and `sfx` table. Respect `muted`, and call `ensureAudio()` from user-gesture handlers only.
- **Accessibility.** Gameplay status changes should go through `announce(text)` so screen readers hear them. Animations that shake or flash should check `reducedMotion` (for example, `screenShake` and the `.reduced-motion` class).
- **Match the local style.** The file uses 2-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow callbacks, and short section-divider comments. Follow that rather than introducing a new style.
- **Comments.** Keep them short and explain *why* (e.g., why a constant exists), not what the next line does.

## Making changes

1. Read the relevant section first; most changes touch one subsystem plus, sometimes, a HUD element and its CSS.
2. Keep edits minimal and within the existing structure. Don't reformat unrelated code.
3. Test in a browser: start, play through at least one stage transition and one boss, pause/resume, game over, and best-score persistence across reloads. Check all three difficulties if you touched `DIFFICULTIES` or spawn logic, and check touch controls if you touched input or layout.
4. Check the browser console for errors.

## Git

- Development happens on the feature branch named for the task (currently `claude/gifted-tesla-uk4su1`). Do not push to `main` directly.
- Write descriptive commit messages in the imperative mood, summarizing what changed and why.
- Do not open a pull request unless the user explicitly asks for one.
