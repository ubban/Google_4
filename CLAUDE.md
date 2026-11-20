# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is one self-contained file:

- `index.html` (~1,670 lines): markup, inline `<style>`, and one inline `<script>` wrapped in an IIFE (`'use strict'`).
- `README.md`: a placeholder. It only has the repo name and a timestamp.

There is no package manager, build step, bundler, test suite, linter, or CI config. Nothing needs to be installed. To run the game, open `index.html` in a browser, or serve the directory with any static server.

## Repository layout

```
index.html   # The entire game (HTML + CSS + JS)
README.md    # Minimal placeholder
CLAUDE.md    # This file
```

## Architecture of `index.html`

**Markup (lines ~362–420).** Key element IDs:
- `#gameCanvas` is the single 2D canvas.
- `#hud`, `#score`, `#best`, `#mult`, `#weaponName`, `#weaponLvl`, `#hearts`, `#bombs` form the HUD.
- `#bossBarWrap` and `#bossBarFill` are the boss health bar.
- `#comboPop` and `#stageBanner` show transient text.
- `#touchControls` (`#moveZone`, `#joystickBase`, `#fireBtn`, `#bombBtn`) is the mobile UI.
- `#overlay` (`#overlayCard`, `#startBtn`, `#diffRow` with `.diffBtn` buttons) is the title, pause, and game-over screen.
- `#muteBtn`, `#motionBtn`, `#pauseBtn` are the toggle controls.

**Script structure.** Everything lives in one IIFE. The sections, in order, are:

1. **Element references and constants.** `STORAGE_*` keys are namespaced `skyfighter.*`. They are `bestScore`, `muted`, `difficulty`, and `reducedMotion`.
2. **Tuning tables.**
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy shoot chance, starting `lives`, `bombs`, and `bossHpMul`.
   - `KILLS_PER_STAGE = 18`.
   - `THEMES`: six sky palettes that cycle per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP, from stage 2).
   - `BOSSES`: six bosses in order (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN).
   - `BOSS_PATTERNS` (~line 1372): bullet-pattern generators such as radial, aimed spread, columns, spiral, cross, homing, and double ring.
3. **Audio.** `ensureAudio()` creates the `AudioContext` lazily, on first user gesture. `beep()`, `noiseBurst()`, and the `sfx` object synthesize every sound. There are no audio files.
4. **Adaptive quality.** `trackPerf(dt)` samples the last 45 frames. It lowers the `quality` level (0 full, 1 medium, 2 low) when frames run slow and raises it when frames run fast.
5. **Input.** Keyboard state is stored in `keys[e.code]`. Touch joystick and fire/bomb buttons set `firing` and the joystick vector. Mouse steering uses `canvasPointFromEvent`.
6. **Game state and logic.**
   - `resetGame()` initializes state.
   - `update(dt)` (~line 1148) is the main simulation step.
   - Spawning: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`.
   - Weapons: `fireWeapon()` implements `vulcan` and `laser`, each at levels 1–3.
   - Bombs: `useBomb()`.
   - Power-ups (`applyPowerUp`): `life`, `bomb`, and `weapon`. Weapon pickups advance the level, then swap weapon type after level 3.
   - Boss: `updateBoss(dt)`, `killBoss()`.
   - Death and end: `loseLife()`, `endGame()`.
7. **Rendering.** `draw()` renders sky, clouds, planes (`drawPlane`), boss ships (`drawBossShip`), bullets, and particles. `loop(now)` runs on `requestAnimationFrame`, clamps `dt` to 0.05 s, and calls `update` then `draw` only while the game is running and not paused.

**Coordinates.** The logical play area is fixed at `BASE_W = 480` by `BASE_H = 720`. `resize()` scales the canvas CSS size to fit the window. The canvas backing store always stays 480×720.

**Persistence.** Only `localStorage` is used, and only for the four `skyfighter.*` keys above.

## Controls (from the in-game hint)

- Mouse: move to steer, click to fire.
- Keyboard: Arrows or WASD move, Space fires, X or Shift drops a bomb, P or Esc pauses.
- Touch: drag left to fly, tap FIRE, tap BOMB.

## Conventions for editing

- **Keep it a single file** unless the user asks to split it. Don't add a build step, framework, or dependencies without asking.
- **Preserve the IIFE and `'use strict'`.** Helpers stay inside the IIFE and do not leak globals.
- **Use the existing style.** Two-space indentation, single quotes in JS, `const` by default, and section headers in the form `// ---------- Name ----------`.
- **Tune gameplay through the constant tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than magic numbers in logic.
- **Respect the reduced-motion and quality paths.** New particle or screen-shake effects should check `reducedMotion` and the `quality` level, as existing effects do.
- **Keep the mobile layout working.** Changes to the HUD or overlays must still fit the 16px-gutter, phone-width layout and the touch controls.
- **Accessibility.** Announcements go through `announce()`, which writes to the `#srLive` live region. Reuse it for new status messages.

## Verifying changes

There are no automated tests. To verify a change:

1. Open `index.html` in a browser, or serve the folder (for example, `python3 -m http.server`).
2. Play through at least one stage on each difficulty you touched.
3. Check pause (P/Esc), mute, reduced-motion, and game-over/restart.
4. For layout changes, check a narrow phone-sized viewport and the touch controls.

Quick syntax check for the inline script, if Node is available:

```bash
node -e "const s=require('fs').readFileSync('index.html','utf8');const m=s.match(/<script>([\s\S]*?)<\/script>/);new Function(m[1]);console.log('ok')"
```

## Git

- Development happens on the branch named in the session instructions. Don't push to other branches without permission.
- Don't open pull requests unless the user asks for one.
