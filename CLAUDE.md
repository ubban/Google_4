# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html` (about 1,670 lines: inline CSS, markup, and one inline `<script>`). There is no framework, no bundler, no package manager, no external assets, and no test suite. Audio is synthesized with the WebAudio API.

`README.md` is a placeholder and carries no project documentation.

## Repository layout

```
index.html   # The whole game: <style> (lines ~10-361), markup (~362-421), <script> (~422-1668)
README.md    # Placeholder
```

Inside the script, the code is one IIFE, grouped by `// ---------- Section ----------` comments:

| Section | Contents |
| --- | --- |
| Adaptive performance | `trackPerf` / `quality` (0 full, 1 medium, 2 low) lowers particle counts when frames run slow |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, the `sfx` table, `EXPLODE_SFX` |
| Canvas sizing | `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit, capped at 1.6x |
| Input | keyboard, touch joystick + fire/bomb buttons, mouse steering |
| Game state | `resetGame`, `fireWeapon`, `update(dt)`, `updateBoss`, `fireRadial` / `fireCross` / `fireColumns` and other boss patterns, `endGame`, `togglePause` |
| Rendering | `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`, `loop(now)` |

Configuration lives in top-level constants near the top of the script: `DIFFICULTIES`, `KILLS_PER_STAGE` (18), `THEMES` (six stage palettes that cycle), `ENEMY_TYPES` (grunt, interceptor, tank), and `BOSSES` (six bosses that cycle: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN).

## Gameplay reference (for changes to game logic)

- **Difficulty** (`DIFFICULTIES`): easy, normal (default), hard. Each sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages**: after 18 kills (`KILLS_PER_STAGE`) a boss spawns. Defeating it advances `stage`, shows a stage banner, and switches the theme.
- **Enemies** unlock by stage: grunt and interceptor from stage 1, tank from stage 2.
- **Weapons**: Vulcan (spread shots) and Laser (piercing lanes). Each weapon has levels 1-3. A weapon power-up raises the level, and at level 3 it swaps to the other weapon and resets to level 1.
- **Power-ups** drop from destroyed enemies: life (6%), bomb (8%), weapon (20%).
- **Scoring** uses a combo chain and a score multiplier (`chain`, `chainTimer`, `multiplier`).
- **Persistence**: localStorage keys `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Keep these names stable, since changing them wipes players' saved state.

## Controls

- Move: arrow keys or WASD. Mouse hover on the canvas also steers; keyboard and joystick input take priority.
- Fire: Space (hold), or hold the on-screen fire button.
- Bomb: X or Shift, or the on-screen bomb button.
- Pause: P or Escape.
- Touch: left-side joystick, right-side fire and bomb buttons.

## Running and verifying

There is no build step and no test runner.

- Run it by opening `index.html` in a browser, or by serving the directory with any static server (for example `python3 -m http.server`).
- Verify changes by playing them in a browser. For automated checks, the pre-installed Playwright/Chromium can load the file and take screenshots. Don't install Playwright again.
- Check the browser console for errors after any change to the script.

## Conventions

- Keep the game in one file unless the user asks for a split. Don't add a build system, framework, or dependencies without a clear request.
- No external network assets: sounds and graphics are generated in code. Keep it that way.
- Match the existing style: 2-space indentation, `const`/`let`, section banner comments, and small helper functions.
- Keep the layout working at phone width. The page uses `touch-action: none`, safe-area insets, and a `clamp()`-sized HUD. Test touch layout when touching HUD or controls.
- Respect `prefers-reduced-motion`. Screen shake, hit flashes, and heartbeat animations are gated by `reducedMotion`.
- Keep the screen-reader hooks (`#srLive`, `announce()`) working when adding user-facing state changes.
- Frame-time-based logic uses `dt`; keep new movement and timers consistent with it.

## Git

- Development branch for this work: `claude/gifted-tesla-lg5cft`. Commit and push there; don't push to other branches without permission.
- Don't open a pull request unless the user asks for one.
