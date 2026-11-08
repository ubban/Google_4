# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single-file HTML5 canvas game.

- `index.html` — the entire game: markup (`<body>`), CSS (`<style>`), and JavaScript (one `<script>` IIFE). No build step, no bundler, no external dependencies.
- `README.md` — minimal project readme.

There is no package manager, test suite, linter, or CI configuration in the repo.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Verifying changes

There are no automated tests. Before committing changes to `index.html`:

1. **Syntax-check the inline script** (write the extracted JS to the session scratchpad, not the repo):
   ```bash
   python3 -c "import re,sys;h=open('index.html',encoding='utf-8').read();open(sys.argv[1],'w').write(re.search(r'<script>(.*?)</script>',h,re.S).group(1))" "$SCRATCH/skyfighter_check.js" && node --check "$SCRATCH/skyfighter_check.js"
   ```
   Set `SCRATCH` to the scratchpad directory first.
2. **Play-test in a browser** for anything touching gameplay, input, audio, or rendering. Check at least desktop (keyboard/mouse) and a narrow mobile viewport (touch joystick, FIRE and BOMB buttons).

Use the scratchpad directory rather than `/tmp` for scratch files when working in a session that provides one.

## Code structure (`index.html`)

The script is a single IIFE with `'use strict'`. Major sections, in file order:

| Section | Purpose |
| --- | --- |
| DOM references | `getElementById` lookups for HUD, overlay, touch controls, boss bar |
| Storage keys | `skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion` (localStorage) |
| Adaptive performance | `trackPerf` drops render quality (`quality` 0–2) when frame time is poor |
| Config tables | `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx.*`, explosion SFX mapping |
| HUD / feedback | hearts, bombs, combo pops, stage banner, screen shake, hit flash |
| Input | keyboard (`keys` map), mouse (`canvasPointFromEvent`), touch joystick and buttons |
| Game state | `resetGame`, `update(dt)`, `loseLife`, `endGame`, `togglePause` |
| Weapons & bombs | `fireWeapon`, `useBomb`, `applyPowerUp` |
| Enemies & bosses | `spawnEnemy`, `spawnBoss`, `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, etc.), `updateBoss`, `killBoss` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Main loop | `loop(now)` via `requestAnimationFrame`; `dt` is clamped to 50 ms |

### Key game rules (keep these in mind when changing balance)

- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy shooting chance, speed multiplier, starting lives, starting bombs, and boss HP multiplier.
- After `KILLS_PER_STAGE` (18) stage kills, the boss warning fires and a boss spawns. Boss design is `BOSSES[(stage-1) % BOSSES.length]`; the theme is selected by stage.
- Tanks (`ENEMY_TYPES.tank`, `minStage: 2`) appear from stage 2.
- Score multiplier is `1 + min(9, floor(chain / 5))`, so it caps at x10. `breakChain()` resets the chain on timeout and when the player loses a life.

## Conventions

- **Single file.** Keep everything in `index.html` unless the user asks for a restructure. Do not introduce a build system or dependencies without asking.
- **Canvas coordinates** use the logical size `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to the viewport. Do not hard-code device pixels.
- **Time-based movement.** Update logic takes `dt` in seconds. New movement or timers should scale with `dt`, not frame count.
- **Persistence** uses the `STORAGE_*` key constants. Existing code calls `localStorage` directly with no try/catch; if you add new storage calls, consider wrapping them so the game still runs when storage is blocked.
- **Accessibility:** respect the reduced-motion setting (`reducedMotion` / `body.reduced-motion`) for screen shake and similar effects, and announce important state changes through `announce()`, which feeds the `srLive` region.
- **Input parity:** every action must work with keyboard, mouse, and touch. Touch uses `preventDefault` on control buttons; keep that pattern.
- **Keep the stylistic match:** 2-space indentation, single quotes in JS, short comment headers in the `// ---------- Name ----------` style.

## Git workflow

- Default working branch for this task: `claude/gifted-tesla-tude3a`. Push with `git push -u origin <branch>`.
- Write descriptive commit messages that explain the gameplay or code change.
- Do not open pull requests unless the user asks for one.
