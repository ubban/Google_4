# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 `<canvas>`. There is no build step, no package manager, no test suite, and no external dependencies.

- `index.html` — the entire game: markup (~lines 1–360 head/CSS, ~361–421 body/overlays/HUD), and one inline `<script>` IIFE (~lines 422–1668) holding all game logic.
- `README.md` — minimal placeholder; currently only a title line.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There are no automated tests or linters configured. Verify changes by playing the game in a browser (desktop keyboard and, where relevant, touch layout).

## Code layout inside the script

The script is organized by `// ---------- Section ----------` comments. Keep new code in the matching section:

| Section | Purpose |
| --- | --- |
| DOM refs and `STORAGE_*` keys | Cached element lookups; `localStorage` keys prefixed `skyfighter.` |
| Adaptive performance | `trackPerf` drops render `quality` (0/1/2) when frame time is poor |
| Config tables | `DIFFICULTIES`, `THEMES` (stage backgrounds), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE` |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX` — Web Audio synthesized sounds, no asset files |
| HUD and effects | `renderHearts`, `renderBombs`, `flashHit`, `showStageBanner`, `screenShake` |
| Layout | `BASE_W = 480`, `BASE_H = 720` logical resolution; `resize()` scales the canvas |
| Input | Keyboard `keys` map, virtual joystick and buttons for touch, mouse via canvas |
| Game state | `resetGame`, `update(dt)`, spawners, power-ups, bombs, weapons |
| Bosses | `BOSS_PATTERNS` (bullet pattern functions), `updateBoss`, `killBoss` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()` |
| Main loop | `loop(now)` with `dt` clamped to 50 ms; `requestAnimationFrame` driven |

## Controls

- Move: arrow keys or WASD; or mouse/touch on the canvas.
- Fire: Space (hold), or the FIRE button / mouse-down on touch.
- Bomb: X or Shift, or the BOMB button.
- Pause: P or Escape, or the pause button.
- Mute, reduced-motion toggle, and difficulty (Easy / Normal / Hard) are in the overlay and HUD.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not add a bundler, framework, or npm dependencies without being asked.
- **Match the existing style.** Two-space indentation, single quotes in JS, `const`/`let` (strict mode is on), arrow functions for callbacks, and `function` declarations for named helpers.
- **Tune with the config tables.** Balance changes (spawn rates, enemy speeds, HP, lives, bomb counts) belong in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` rather than as magic numbers in logic.
- **Time-based updates.** Movement and timers use `dt` in seconds. Do not tie behaviour to frame counts.
- **Persistence.** Only the keys in the `STORAGE_*` constants are stored. Add new keys with the same `skyfighter.` prefix, and guard against unavailable storage if you add new reads.
- **Accessibility.** Respect `prefers-reduced-motion` (the `reducedMotion` flag gates screen shake, flashes, and pulses). Announce important state changes through `announce()` into the `srLive` live region.
- **Audio** is synthesized with Web Audio and only starts after a user gesture (`ensureAudio()`). Do not add audio files without a reason.

## Workflow

- Develop on the branch the task specifies (currently `claude/gifted-tesla-lnyv7o`). Do not push to other branches without permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user explicitly asks for one.
- When a change touches gameplay balance or visuals, say in the summary what was verified in a browser and what was only checked by reading the code.
