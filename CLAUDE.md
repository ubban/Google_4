# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. Switch weapons, drop smart bombs, and take down stage bosses.

The whole game lives in `index.html`: inline `<style>`, the markup, and one inline `<script>` (starts around line 422). There is no build step, no package manager, no bundler, and no test suite. `README.md` is a stub.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

There are no install or build commands. Verify changes by playing the game in a browser, including on a narrow/touch viewport, because the game has touch controls.

## Code layout (`index.html`)

- **Markup and CSS (top of file):** HUD (`#hud`, `#hearts`, `#bombs`, `#weaponStat`, `#multStat`), overlay/menu (`#overlay`, `#startBtn`, `.diffBtn`), boss bar (`#bossBarWrap`), touch controls (`#touchControls`, `#moveZone`, `#fireBtn`, `#bombBtn`), and a `#vignette`/`#flash` damage/bomb effect layer.
- **DOM references (~line 426):** every element the script uses is fetched once into a `const` at the top of the script. Add new element lookups there.
- **Persistence keys (~line 463):** `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Keep the `skyfighter.` prefix for any new key.
- **Tuning data (~lines 485–516):**
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed before a boss spawns.
  - `THEMES`: six stage backgrounds, cycled with `(stage - 1) % THEMES.length`.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). `minStage` gates when each type appears.
  - `BOSSES`: six bosses (VANGUARD → TITAN), cycled the same way, each with an `hpMul`.
- **Audio (~line 543+):** a lazily created `AudioContext` (`ensureAudio`), with `beep`/`noiseBurst` primitives and a `sfx` table. Audio starts only after a user gesture.
- **Input (~line 768+):** keyboard (`keys[e.code]`), mouse (`mouseTarget`, click-to-fire via `firing`), and touch (virtual joystick plus Fire/Bomb buttons).
- **Rendering and sizing:** a fixed logical canvas of `BASE_W × BASE_H` = 480 × 720, scaled to fit the window by `resize()`. Coordinates in game logic are in the logical space.
- **Game loop:** `update(dt)` (~line 1148) → `draw()` (~line 1520), driven by `loop(now)` (~line 1640) via `requestAnimationFrame`. `trackPerf` adjusts a `quality` level (full / medium / low) when frame times are slow.
- **Boss patterns:** `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss`.
- **Lifecycle:** `resetGame()` (~line 875) resets state, `endGame()` (~line 1469) shows the game-over overlay and saves the best score.

## Controls (keep the in-game hint in sync)

| Action | Keyboard | Mouse / Touch |
| --- | --- | --- |
| Move | Arrows / WASD | Mouse position / drag left zone |
| Fire | Space (held) | Click / hold FIRE button |
| Bomb | X or Shift | BOMB button |
| Pause | P or Esc | Pause button |

The hint text lives in `#controlsHint` (~line 417). Update it whenever controls change.

## Conventions

- **Match the existing style.** The script uses 2-space indentation, single quotes, `const`/`let`, and short comments only where the logic is not obvious. Use the same naming (`camelCase` for functions and variables, `UPPER_SNAKE` for config tables).
- **Keep it one file.** Do not split the game into modules or add a build step unless the user asks for it.
- **Put tuning in the data tables.** New enemy, boss, theme, or difficulty values belong in `ENEMY_TYPES`, `BOSSES`, `THEMES`, or `DIFFICULTIES`, not hard-coded in the update loop.
- **Respect the user's settings.** Mute and reduced-motion preferences are persisted and should keep working. Screen shake and flash effects must honor `reducedMotion` (see `screenShake`, `flashHit`, and `flashBomb`), which dampens shake and skips flashes.
- **Accessibility.** Announcements go through `announce()` (the `#srLive` region). Keep the overlay and buttons keyboard- and screen-reader-friendly.
- **Mobile first.** Viewport meta disables zoom, and `touch-action: none` is set on the body. Do not reintroduce page scrolling or zoom behavior.
- **Performance.** The loop runs every frame. Avoid allocating large objects or arrays per frame, and respect the existing `quality` tiers when adding effects.

## Git

- Develop on the branch the session specifies. Do not push to `main` directly.
- Write descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless the user asks for one.

## Verification checklist for changes

Because there are no automated tests, check manually before committing:

1. The page loads with no console errors (open DevTools).
2. Start a game on each difficulty; confirm HUD, score, and best score update.
3. Confirm keyboard and touch controls work, and that bombs, weapon upgrades, and pause all function.
4. Reach a boss (about 18 kills) and confirm the boss bar, attack patterns, and stage transition.
5. Toggle mute and reduced motion, and confirm they persist after a reload.
6. Resize the window to a phone-sized viewport and confirm the canvas still fits.
