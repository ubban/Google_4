# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML file.

- `index.html` (~1,670 lines): the entire game. HTML markup, inline `<style>`, and one inline `<script>` (starts around line 422). It uses a `<canvas>` and the Web Audio API for sound.
- `README.md`: a one-line title. Keep it short; do not let it drift from the code.

There is no build step, no package manager, no bundler, no test suite, and no server requirement. The game runs by opening `index.html` directly in a browser.

## Repository layout

```
.
├── index.html   # Entire game: markup, CSS, and JS
├── README.md    # Project title
└── CLAUDE.md    # This file
```

Do not add a build system, framework, or npm dependencies unless the user asks for one. Keep the project a single self-contained page unless there is a clear reason to split it.

## Running and verifying

- **Run:** open `index.html` in a browser, or serve the directory with any static server (for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`).
- **Verify changes:** there are no automated tests. After editing, load the page and check the affected behavior by hand. For gameplay changes, start a game, play through the affected path, and confirm the console shows no errors.
- **Syntax check:** you can extract the inline script and run `node --check` on it, but a browser load is the real check.
- The pre-installed Playwright/Chromium in the cloud environment can be used for a headless smoke test (load the page, look for console errors, take a screenshot at `deviceScaleFactor: 2` when a visual check is needed).

## Code structure inside `index.html`

The inline script is organized as one large IIFE-style block with these areas, roughly in file order:

1. **Constants and storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`. Keys are namespaced `skyfighter.*`.
2. **Persisted settings**: best score, mute, difficulty, and reduced-motion preference are read from and written to `localStorage`. Always wrap storage access so that a blocked or unavailable `localStorage` does not break the game.
3. **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, `panFor()`. Audio is created lazily on the first user gesture, so keep it that way (browsers block autoplay).
4. **HUD and feedback**: `renderHearts()`, `renderBombs()`, `flashHit()`, `flashBomb()`, `showComboPop()`, `showStageBanner()`, `screenShake()`, `announce()` for screen readers.
5. **Settings UI**: `setDifficulty()`, mute and reduced-motion toggles.
6. **Input**:
   - Keyboard: `keys[e.code]` map. Move with arrows or WASD. Fire with Space. Bomb with X or Shift. Pause with P or Escape.
   - Touch: a virtual joystick (`joystickStart/Move/End`), a fire button, and a bomb button.
   - Mouse: the canvas tracks the pointer (`canvasPointFromEvent`), and mouse-down fires.
7. **Game state and lifecycle**: `resetGame()`, `update(dt)`, `loseLife()`, `endGame()`, `togglePause()`.
8. **Entities and spawning**: enemies (`spawnEnemy`), bosses (`spawnBoss`, `updateBoss`, `killBoss`), power-ups (`maybeSpawnPowerUp`, `applyPowerUp`), explosions, scoring and combo chain (`addScore`, `breakChain`).
9. **Weapons and boss patterns**: `fireWeapon()`, `useBomb()`, and the boss bullet patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
10. **Rendering**: `draw()` and its helpers `drawPlane()`, `drawBossShip()`, `drawClouds()`.
11. **Main loop**: `loop(now)` drives `update` and `draw` through `requestAnimationFrame`. `trackPerf(dt)` monitors frame timing.
12. **Resize**: `resize()` fits the canvas to the viewport and respects safe-area insets.

When changing behavior, find the owning section first and keep the change local to it.

## Conventions

- **Plain JavaScript, ES2017+ syntax, no transpilation.** Target current evergreen browsers. Avoid adding libraries.
- **Keep it in one file** unless the user asks for a split. If a split is requested, move CSS and JS into sibling files referenced with relative paths.
- **Naming:** `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constants, `skyfighter.`-prefixed keys for storage.
- **Frame-rate independence:** game logic takes a `dt` argument. Scale movement and timers by `dt`, not by frame count.
- **Accessibility and comfort:** respect the reduced-motion setting for screen shake and flashes. Keep `announce()` updated for significant state changes (level, game over, boss). The viewport meta disables zoom (`user-scalable=no`) on purpose for the fullscreen game; do not loosen it without a reason, and mention the trade-off to the user if you do.
- **Mobile:** the layout must work at phone width with no horizontal scroll. Touch controls must not conflict with page scroll (`touch-action: none` is set on the game).
- **Error handling:** guard `localStorage` and audio calls. Do not let a failure in either stop the game loop.

## Git and workflow

- The default working branch for this task is `claude/gifted-tesla-iy68i8`. Commit and push there. Do not push to other branches without explicit permission.
- Do not create a pull request unless the user asks for one.
- Keep commits focused. Use imperative, present-tense subject lines (for example, `Add boss health bar to HUD`).
- Before committing, confirm that `index.html` still loads and that the game starts without console errors.

## Things to avoid

- Do not add `node_modules`, lockfiles, or build output to the repo.
- Do not commit secrets, API keys, or personal data.
- Do not remove the `skyfighter.*` storage keys or change their names without a migration plan, because they hold players' saved best score and settings.
- Do not change the `<meta name="viewport">` settings or the `touch-action`/`user-select` rules without checking the mobile layout.
