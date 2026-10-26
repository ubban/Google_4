# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas app.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS, and JavaScript in one file (~1,700 lines). |
| `README.md` | Placeholder; only contains the project title. |

There is no build system, package manager, bundler, test suite, or lint configuration. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

Verifying changes means loading the page and playing it. There are no automated tests to run.

## Code layout inside `index.html`

- **`<style>`** (around lines 10–420): all CSS. Colors, HUD pills, overlay card, touch controls, and the `.reduced-motion` class live here.
- **`<body>`** (around lines 363–420): HUD, overlay, touch controls, and the `<canvas id="gameCanvas">`. Element IDs are referenced by `getElementById` in the script, so renaming an ID means updating the script too.
- **`<script>`** (starts around line 422): one IIFE (`'use strict'`) containing all game logic. Major sections, in order:
  - DOM references and `STORAGE_*` keys
  - Adaptive performance (`trackPerf`, `quality` 0–2)
  - Config tables: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`
  - Audio (`ensureAudio`, `beep`, `noiseBurst`), synthesized with the Web Audio API; no audio files
  - Input: keyboard (`keys` map), touch joystick, fire and bomb buttons, mouse
  - `resize()` scales the canvas to a fixed `BASE_W × BASE_H` (480 × 720) logical resolution
  - Game state: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `updateBoss`, `killBoss`, `loseLife`, `endGame`
  - Weapons and bosses: `fireWeapon`, `useBomb`, `applyPowerUp`, and the `fire*` boss pattern helpers
  - Rendering: `draw()`, `drawPlane`, `drawBossShip`, `drawClouds`
  - Main loop: `loop(now)`

## Game conventions

- **Persistence** uses `localStorage` with the keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, and `skyfighter.reducedMotion`. Don't rename these keys without a migration, since they hold players' saved state. Existing reads and writes are not uniformly wrapped in `try/catch`; wrap any new storage access, because storage can be blocked in private windows or by site settings.
- **Difficulty** is set by `DIFFICULTIES` (`easy` / `normal` / `hard`), which controls spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages:** once `stageKills` reaches `KILLS_PER_STAGE` (18), a boss spawns. Defeating it advances `stage`, and the theme is `THEMES[(stage - 1) % THEMES.length]`. `BOSSES` has six entries, and `BOSS_PATTERNS` supplies each boss's attack patterns.
- **Controls:** arrows or WASD to move, Space or hold-to-fire (mouse/touch fire button) to shoot, X / Shift to drop a bomb, P / Escape to pause. Touch devices get an on-screen joystick and buttons.
- **Accessibility:** `prefers-reduced-motion` is respected by default and can be toggled in the UI. Reduced motion turns off or softens screen shake and hit flashes. Announcements for screen readers go through the `srLive` live region via `announce()`.
- **Performance:** `trackPerf` lowers the `quality` level when frame time is high. Keep per-frame allocations low in `update` and `draw`, since they run every frame.

## Editing guidelines

- Keep everything in `index.html` unless there is a clear reason to split files. The single-file layout is deliberate.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, short section headers written as `// ---------- Name ----------`, and comments only where the logic is not obvious.
- Don't introduce external dependencies, CDNs, or build steps. The game is meant to run from a plain file open.
- When you change a tuning value, check its effect on all three difficulty levels and on the boss/stage progression, not just the default one.
- Test in a browser for both keyboard and touch paths when you change input, and at a narrow phone-sized viewport when you change layout or CSS.

## Git

- Develop on the branch assigned for the task (currently `claude/gifted-tesla-owyyqz`); don't push to `main` directly.
- Write descriptive commit messages that explain the gameplay or code change.
