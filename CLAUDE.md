# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html` (inline CSS, inline JavaScript, no build step). `README.md` is a stub.

Git history: `4efdad1` initial commit, `c4dc8f6` "Add Sky Fighter: Raiden-style vertical shoot-em-up game".

## Repository layout

```
index.html   # The entire game: markup (~L361-421), <style> block, and one IIFE <script> (~L422-1667)
README.md    # Placeholder title only
```

There is no `package.json`, no bundler, no test suite, no linter config, and no CI config. Do not add tooling unless asked.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by loading the page and playing: start a game, fire, use a bomb, collect power-ups, reach a boss, pause, and check the game-over overlay.
- For UI or gameplay changes, check both desktop (keyboard/mouse) and touch layouts. The game is designed for phone-width screens too.

## Architecture of `index.html`

All game code sits inside a single `(function () { 'use strict'; ... })();` IIFE. Nothing is exported to `window`, so keep new code inside that IIFE.

Main sections of the script, in order:

1. **DOM refs**: `document.getElementById` lookups for HUD, overlay, touch controls, and buttons.
2. **Persistence keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`.
3. **Config tables**: `DIFFICULTIES` (easy/normal/hard: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE = 18`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
4. **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and sound-effect maps (for example `EXPLODE_SFX`). Audio is created lazily after a user gesture.
5. **Presentation helpers**: HUD rendering (`renderHearts`, `renderBombs`), flashes, combo pop, stage banner, screen shake. These respect the reduced-motion setting.
6. **Layout and input**: `resize()` (logical canvas is `BASE_W` x `BASE_H` = 480 x 720, scaled to fit), keyboard handlers, joystick, mouse, and touch buttons.
7. **Game state**: `resetGame()` and the `update(dt)` step (movement, firing, collisions, spawning, scoring, chain/combo logic).
8. **Weapons and power-ups**: `fireWeapon`, `applyPowerUp`, `maybeSpawnPowerUp`.
9. **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`, and the attack-pattern functions (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, and so on).
10. **Lifecycle**: `loseLife`, `endGame`, `togglePause`.
11. **Rendering**: `draw()` and its sprite helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
12. **Loop**: `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 50 ms, and calls `trackPerf(dt)` for adaptive quality.

Game flow: `startBtn` click calls `resetGame()` and hides the overlay. Losing all lives calls `endGame()`, which shows the stats overlay and updates the best score in `localStorage`.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Do not introduce a framework or build step.
- **Frame-rate independence.** Update game logic with `dt` (seconds, clamped), not per-frame increments.
- **Vanilla JS and Canvas 2D.** Follow the existing style: `const` by default, arrow functions for handlers, descriptive camelCase names, and `// ---------- Section ----------` banners between sections.
- **Accessibility and comfort.** Respect the reduced-motion toggle for any new shake, flash, or pulse effect. Announce important state changes through `#srLive` (`announce()`), and keep controls keyboard-, mouse-, and touch-accessible.
- **Persistence.** Wrap new `localStorage` use the same way existing code does, and add new keys with the `skyfighter.` prefix.
- **Tuning.** Balance numbers live in the config tables near the top of the script. Prefer editing those over hard-coding values in logic.
- **Mobile.** Keep `touch-action: none` and the viewport meta intact. Do not add elements that block the joystick or action buttons.

## Things to be careful with

- Removing `'use strict'` or the IIFE wrapper would leak globals and change behavior.
- Changes to `update()` or the boss pattern functions affect difficulty; re-check all three difficulty levels.
- `keys` is keyed by `event.code`, not `event.key`. Keep that when adding controls.
- The HUD and touch control layout is responsive via `clamp()` and `vmin`/`vw` units. Test narrow widths after CSS changes.

## Git workflow

- Default work branch for automated sessions: `claude/gifted-tesla-ugj38b`.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
