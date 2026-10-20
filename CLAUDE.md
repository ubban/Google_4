# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html` (inline CSS, inline JavaScript, HTML5 `<canvas>`, Web Audio API). `README.md` is a placeholder.

There is no build step, no package manager, no bundler, and no test suite. To run the game, open `index.html` in a browser or serve the directory with any static server.

## Repository layout

- `index.html`: the whole game.
  - Lines ~10–360: CSS (HUD pills, glassmorphic overlay card, touch controls, vignette, flash, boss bar, stage banner).
  - Lines ~361–421: DOM markup (`#container`, `#hud`, `#bossBarWrap`, `#touchControls`, `#overlay`).
  - Lines ~422 onward: one IIFE containing all game logic, rendering, audio, and input.
- `README.md`: placeholder, no project docs.

## Architecture of the game script

The script is a single `(function () { ... })();` block. Its main parts, in order:

1. **Persistence keys** (`STORAGE_*`): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. All `localStorage` reads and writes use these constants.
2. **Tuning tables**:
   - `DIFFICULTIES`: `easy`, `normal`, `hard`, each with spawn rates, enemy speed and fire chance, starting `lives`, starting `bombs`, and `bossHpMul`.
   - `KILLS_PER_STAGE` (18): kills needed to advance a stage, which then triggers a boss.
   - `THEMES`: six sky palettes that cycle by stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP). Each has a `minStage`.
   - `BOSSES`: six bosses (`VANGUARD`, `BEHEMOTH`, `SPECTER`, `DREADNOUGHT`, `WRAITH`, `TITAN`) with colors and `hpMul`.
3. **Audio**: `ensureAudio()` lazily creates the `AudioContext` on the first user gesture. Sound effects come from `beep()` and `noiseBurst()`. Respect `muted`.
4. **Adaptive quality**: a frame-time tracker (`trackPerf`) moves `quality` between 0 (full), 1 (medium), and 2 (low). It scales shadow blur, particle counts, and similar effects.
5. **Input**: keyboard (`keys` map on `e.code`), touch joystick plus fire and bomb zones, and mouse fire. Keys: arrows or `A`/`D`/`W`/`S` to move, `Space` to fire, `X` or `Shift` for a bomb, `P` or `Esc` to pause.
6. **Game state and loop**: `resetGame()` initializes state. `update(dt)` runs the simulation. `draw()` renders. `loop(now)` runs `update` and `draw` through `requestAnimationFrame`, with `dt` clamped to 0.05 s and updates skipped while paused or not running.
7. **Weapons and power-ups**: `fireWeapon()` implements Vulcan and Laser, each with 3 levels. `applyPowerUp()` handles weapon level, bomb, and extra life. `useBomb()` clears the screen and damages bosses.
8. **Bosses**: `spawnBoss()`, `updateBoss()`, and `BOSS_PATTERNS` (radial, aimed spread, columns, spiral, cross, seeded homing, double ring). `killBoss()` advances the stage.
9. **Scoring**: `addScore()` applies a kill-chain multiplier. `breakChain()` resets it when the player is hit.
10. **End of game**: `loseLife()` and `endGame()` persist `bestScore` when it improves.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Don't add a build step, framework, or dependencies without a request.
- Match the existing style: 2-space indentation, semicolons at statement ends, `const` and `let`, arrow functions for callbacks, and short section banners like `// ---------- Input ----------`.
- Put tunable numbers in the existing tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- Gate any new persisted setting behind a `STORAGE_*` constant, wrap access so the game still works if `localStorage` is unavailable, and keep the key namespaced under `skyfighter.`.
- Support both keyboard and touch when adding input. Touch controls are shown only when `isTouchDevice` is true.
- Respect `reducedMotion` for new screen shake, flashes, or large animations.
- Use `ctx.save()` and `ctx.restore()` around transforms, and keep draw code free of side effects on game state.

## Verifying changes

There are no automated tests or linters. Verify by:

1. Opening `index.html` in a browser, or serving the directory, and checking the console for errors.
2. Playing through at least one stage transition and one boss fight after gameplay changes.
3. Checking all three difficulties if the change touches `DIFFICULTIES`.
4. Checking touch controls with browser device emulation if the change touches input or layout.
5. Checking the layout at a narrow (phone) width, since the canvas is scaled from `BASE_W` 480 × `BASE_H` 720.

A quick syntax check of the inline script is also useful:

```bash
node -e "const s=require('fs').readFileSync('index.html','utf8');const m=s.match(/<script>([\s\S]*)<\/script>/);new Function(m[1]);console.log('ok')"
```

## Git

- Develop on the branch the session specifies. Don't push to `main` directly.
- Write descriptive commit messages in the imperative mood, with a short summary line and a body that explains the gameplay or code change.
