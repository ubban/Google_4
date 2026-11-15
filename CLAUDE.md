# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`. The whole game lives in one file:

- `index.html` (~1,670 lines): inline `<style>` (lines ~10–359), markup (~362–421), and one inline `<script>` (~422–1667) wrapped in an IIFE.
- `README.md`: a placeholder with only a timestamp. It has no setup or usage docs.

There is no build system, package manager, bundler, framework, test suite, or linter. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. For gameplay or UI changes, check the desktop keyboard controls and the touch controls (the touch overlay appears on touch-capable devices).
- Keep changes inside the existing single file unless there is a clear reason to split it out. If you do split files, update this document.

## Architecture of `index.html`

The script is one IIFE with these sections, roughly in file order:

1. **Persistence and settings**: `localStorage` keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read and write must tolerate storage being unavailable.
2. **Tuning tables**:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, shoot chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage, cycled), `ENEMY_TYPES` (`grunt`, `interceptor`, `tank`), and `BOSSES` (six boss definitions, with `hpMul`).
   - `BOSS_PATTERNS` (~line 1372): bullet patterns such as radial, aimed spread, columns, spiral, cross, homing, and double ring.
3. **Audio**: a lazily created `AudioContext` (`ensureAudio`). Sound effects are synthesized with `beep` and `noiseBurst`. Nothing is loaded from disk.
4. **UI helpers**: HUD rendering (`renderHearts`, `renderBombs`), flashes, combo pop, stage banner, screen shake, and `announce()` for screen readers via the `#srLive` region.
5. **Layout and input**: `resize()` fits a `BASE_W × BASE_H` (480 × 720) logical canvas to the window. Input comes from keyboard, mouse, a virtual joystick (`moveZone`), and touch buttons (FIRE, BOMB).
6. **Game state**: module-level `let` variables (`score`, `lives`, `stage`, `player`, `enemies`, and so on). `resetGame()` reinitializes them.
7. **Update and draw**: `update(dt)` advances the simulation. `draw()` renders it. `loop(now)` runs on `requestAnimationFrame`, caps `dt` at 0.05 s, and skips updates while paused or not running. `trackPerf` lowers the render `quality` level (0 full, 1 medium, 2 low) when frames are slow.

### Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or WASD | Virtual joystick |
| Fire | Space (hold) | FIRE button |
| Bomb | X or Shift | BOMB button |
| Pause | P or Esc | Pause button in top bar |
| Mute | Mute button | Mute button |

### Game systems

- **Weapons**: `vulcan` and `laser`. Weapon power-ups raise `player.level` from 1 to 3. At level 3, a further power-up toggles the weapon type and resets the level.
- **Power-ups**: `life`, `bomb`, and `weapon` (see `applyPowerUp`).
- **Combo chain**: `chain` and `multiplier` increase with kills and reset on `breakChain()`.
- **Stages and bosses**: every `KILLS_PER_STAGE` kills, a boss warning appears and a boss spawns. Bosses use `BOSS_PATTERNS` and are defeated through `killBoss()`.
- **Game over**: `loseLife()` and `endGame()`. `bestScore` is saved when it is beaten.

## Conventions

- Keep the file dependency-free: vanilla JS, Canvas 2D, and inline CSS. Do not add CDN scripts or libraries without a strong reason.
- Put tuning values in the existing constant tables rather than in magic numbers inside functions.
- Respect the reduced-motion setting: any new screen shake, flash, or pulse should go through the existing helpers, which check `reducedMotion`.
- Preserve mobile support. The layout must work at phone width with the touch controls. The viewport meta disables user zoom, so keep the layout responsive.
- Keep accessibility hooks working: new status messages should go through `announce()`, and interactive elements need labels or `title` attributes.
- Match the surrounding style: 2-space indentation, single quotes in the script, and terse comments that explain why rather than what.

## Git

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-9p4hjv`). Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Commit messages should be short and imperative. Recent history uses messages such as `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.

## Known gaps

- `README.md` is a placeholder. Expand it with a description and run instructions if the project grows.
- There is no automated test coverage. Gameplay changes can only be checked manually.
