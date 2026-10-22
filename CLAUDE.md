# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up built on an HTML5 `<canvas>`. There is no build step, no package manager, and no test suite.

- `index.html` — the entire game: markup, CSS (`<style>` block), and JavaScript (one IIFE in a `<script>` block near the end of the file).
- `README.md` — minimal project title only.

To run the game, open `index.html` in a browser. Nothing needs to be installed.

## Repository layout

```
index.html   # all game code (HTML + CSS + JS)
README.md    # project title
CLAUDE.md    # this file
```

Keep the game as a single self-contained file unless the user asks to split it. Do not introduce a bundler, framework, or external dependencies without asking.

## Code structure inside `index.html`

The JavaScript is wrapped in `(function () { 'use strict'; ... })()`. Within it, roughly in order:

1. **DOM references** — `const xxxEl = document.getElementById(...)`. Every element the script touches is looked up once at the top.
2. **Storage keys** — `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`, all prefixed `skyfighter.`.
3. **Tuning tables** — `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`. Balance changes usually belong here rather than in logic.
4. **Audio** — `ensureAudio()`, `beep()`, `noiseBurst()`, Web Audio based; `EXPLODE_SFX` maps enemy types to sounds.
5. **HUD and effects** — `renderHearts`, `renderBombs`, `flashHit`, `showComboPop`, `showStageBanner`, `screenShake`.
6. **Input** — `keys` map keyed by `e.code`, joystick and touch handlers for mobile, `canvasPointFromEvent`.
7. **Game state** — `resetGame()` builds fresh state; `update(dt)` runs per-frame logic; `fireWeapon()`, `useBomb()`, `applyPowerUp()`.
8. **Bosses** — `spawnBoss()`, `BOSS_PATTERNS` (an array of bullet-pattern functions such as `fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`), `updateBoss(dt)`, `killBoss()`.
9. **Rendering** — `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
10. **Main loop** — `loop(now)` drives `update` and `draw` with a delta-time cap.

When adding a feature, follow this pattern: add tuning values to the relevant table, put logic in the matching section, and keep rendering code in the rendering section.

## Conventions

- **Language and style:** plain ES2015+ JavaScript, no transpilation. Use `const` by default and `let` when reassigned. Arrow functions for callbacks, `function` declarations for named helpers. Single quotes for strings. Two-space indentation.
- **Comments:** section headers use the form `// ---------- Name ----------`. Add a short comment only where the intent is not obvious from the code.
- **Naming:** `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constant tables, `STORAGE_*` for localStorage keys.
- **Frame timing:** all movement and timers are multiplied by `dt`. Do not hard-code per-frame increments.
- **Scope:** everything lives inside the IIFE. Do not add globals, and expose nothing on `window` except the existing listeners.
- **Persistence:** localStorage is the only persistence layer. New settings should use a `skyfighter.`-prefixed key and be read with a fallback default, as the existing ones are.

## Accessibility and responsiveness

These behaviors are already implemented. Preserve them when changing related code.

- **Reduced motion:** `reducedMotion` is set from the OS `prefers-reduced-motion` setting unless the player has toggled it (`motionBtn`). It adds the `reduced-motion` class to `<body>`, and the CSS under `body.reduced-motion` suppresses the vignette, hit flash, combo pop, stage banner, and overlay animations. Screen shake is not gated in JS today; if you add motion effects, check `reducedMotion` in code as well as in CSS.
- **Screen reader announcements:** use `announce(text)` for game events that matter to assistive tech. It writes to the `#srLive` live region.
- **Adaptive quality:** `trackPerf(dt)` averages the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low). Expensive effects should check `quality` before drawing.
- **Mobile:** the layout uses `touch-action: none`, a virtual joystick in `#moveZone`, and fire and bomb buttons. Touch handlers call `preventDefault` and `ensureAudio()` on first interaction, because browsers require a user gesture before audio can start.
- **Viewport:** `resize()` scales the canvas to the window while keeping the `BASE_W` × `BASE_H` (480 × 720) logical resolution. Game coordinates always use the base size, never the CSS size.

## Controls (for reference when editing input code)

| Action        | Keyboard                        | Touch           |
|---------------|---------------------------------|-----------------|
| Move          | Arrow keys or WASD              | Virtual joystick|
| Fire          | Space (hold)                    | Fire button     |
| Smart bomb    | X or Shift                      | Bomb button     |
| Pause         | P or Escape                     | Pause button    |
| Mute          | Mute button                     | —               |

## Gameplay model (summary)

- **Weapons:** `vulcan` and `laser`, each with power levels (`player.level`). A power-up toggles between them (`applyPowerUp`). Fire delay is 0.1 s for laser and 0.15 s for vulcan.
- **Lives and bombs:** starting values depend on difficulty. Bombs clear the screen and damage bosses.
- **Stages:** each stage spawns waves until `KILLS_PER_STAGE` (18) kills, then a boss fights. Stage numbers cycle through `THEMES`.
- **Score:** kills award points scaled by a kill-chain multiplier (`addScore`, `breakChain`).
- **Best score:** saved to localStorage when a game ends (`endGame`).

## Testing and verification

There is no automated test suite. To verify a change:

1. Open `index.html` in a browser, or serve the directory with any static server.
2. Play through at least one full stage, including a boss fight, on each difficulty you touched.
3. Check the browser console for errors.
4. For UI or layout changes, check both a desktop viewport and a phone-sized viewport (the game is designed for touch).
5. For audio changes, confirm the mute toggle still works and that sound starts only after a user gesture.

If you add a test harness later, document how to run it here.

## Git workflow

- Develop on the branch specified for the task. Do not push to other branches without permission.
- Write clear, descriptive commit messages in the imperative mood, with a summary line and an optional body for context. The existing history uses the form "Add Sky Fighter: ..." for the initial feature commit.
- Do not create pull requests unless explicitly asked.

## Guidelines for Claude

- Keep changes small and scoped to the request. Do not refactor the IIFE structure or reorder sections unless asked.
- Do not add dependencies, build tooling, or new files unless the user requests them.
- When changing game balance, edit the tuning tables instead of hard-coding numbers in logic.
- Preserve the existing accessibility and mobile behavior listed above.
- If a change affects gameplay, mention in the final message which values changed so the user can playtest them.
