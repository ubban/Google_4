# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game (HTML, CSS, and JavaScript) lives in a single file, `index.html` (~1,670 lines). `README.md` is a placeholder and has no build or setup instructions.

There is no package manager, build step, test suite, or linter. No external assets are loaded; sound is synthesized with the WebAudio API.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Repository layout

| Path         | Purpose                                      |
|--------------|----------------------------------------------|
| `index.html` | The whole game: markup, `<style>`, `<script>` |
| `README.md`  | Placeholder README                           |

Inside `index.html`, the sections run in roughly this order:

1. **Head / styles** (`<style>`, lines ~1–359): meta tags, layout, HUD, touch controls, and reduced-motion overrides (`body.reduced-motion ...`, ~line 349).
2. **Markup** (lines ~360–421): canvas, HUD, pause/menu overlays, touch joystick, fire and bomb buttons, and the motion toggle (`#motionBtn`).
3. **Script** (starts at line 422):
   - Constants: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`.
   - Audio: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map.
   - HUD and feedback: `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `screenShake`, `announce` (screen-reader text).
   - Input: keyboard (`keys`), touch joystick (`joystickStart/Move/End`), fire and bomb buttons, and mouse steering (`canvasPointFromEvent`).
   - Game state: `resetGame`, `update(dt)`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `loseLife`, `killBoss`, `endGame`, `togglePause`.
   - Boss attacks: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, and `updateBoss`.
   - Rendering: `draw`, `drawPlane`, `drawBossShip`, `drawClouds`.
   - Main loop: `loop(now)`, which is driven by `requestAnimationFrame`.

## Conventions

- **Keep it a single file.** Don't split out modules, add a bundler, or introduce dependencies unless the user asks for that change.
- **Use no external assets.** Graphics are drawn on the canvas and sound comes from WebAudio. Don't add image or audio files, or CDN links, without an explicit request.
- **Use the existing naming scheme.** Constants are `UPPER_SNAKE_CASE`, `STORAGE_*` keys use the `skyfighter.*` prefix, and functions are `camelCase`.
- **Persist settings through `localStorage`** only via the `STORAGE_*` keys (best score, mute, difficulty, reduced motion).
- **Tune gameplay in the data tables.** Difficulty values live in `DIFFICULTIES`, and enemy and boss stats live in `ENEMY_TYPES` and `BOSSES`. Prefer editing these to changing logic.
- **Support touch and keyboard.** Any new control needs both a keyboard binding and a touch or click path. Current keys: arrows or WASD to move, Space to fire, X or Shift to bomb, P or Escape to pause.
- **Respect reduced motion.** The `reducedMotion` flag (initialized from `prefers-reduced-motion` or the saved setting) scales down `screenShake`, and the `body.reduced-motion` CSS class disables flashes, pulses, and banners. New visual effects should check both.
- **Keep the canvas responsive.** The game uses a fixed logical size (`BASE_W = 480`, `BASE_H = 720`) scaled by `resize()`. Don't hard-code pixel positions outside that coordinate space.

## Verifying changes

There are no automated tests. After editing:

1. Open `index.html` in a browser (or serve it locally) and confirm there are no console errors.
2. Play through the main flow: start, fire, collect power-ups, use a bomb, lose a life, reach a boss, and reach the game-over screen.
3. Check the touch controls at phone width (or with device emulation) and the keyboard controls on desktop.
4. Confirm that difficulty and mute settings persist across a reload.

For visual or interactive checks, the `run` skill can launch the app.

## Git workflow

- Develop on the branch assigned to the session, and don't push to other branches without permission.
- Write short, imperative commit messages that describe the gameplay or code change (e.g. `Add Sky Fighter: Raiden-style vertical shoot-em-up game`).
