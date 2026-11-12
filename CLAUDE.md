# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The whole game lives in a single self-contained file, `index.html` (inline `<style>` and `<script>`, HTML5 canvas, no external assets or libraries).

Files:

- `index.html` — the entire game (markup, CSS, and JavaScript).
- `README.md` — minimal project readme.

There is no build step, package manager, test suite, or linter configured.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

## Code structure (inside `index.html`)

The script is a single IIFE-style block. Major sections, in rough order:

- **DOM references** — `getElementById` lookups for HUD, overlays, and touch controls.
- **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` in `localStorage`.
- **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low).
- **Config tables** — `DIFFICULTIES` (spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six boss designs, cycled by stage).
- **Audio** — WebAudio-based `beep`/`noiseBurst` helpers and a `sfx` table; audio is lazily initialized via `ensureAudio()` on user gesture.
- **Input** — keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse (steer and fire on canvas), and touch (virtual joystick, FIRE and BOMB buttons).
- **Game state and loop** — `resetGame`, `update(dt)`, render functions (`drawPlane`, `drawBossShip`), and a `requestAnimationFrame` `loop`.
- **Combat** — `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`, boss attack patterns in `BOSS_PATTERNS`, `updateBoss`, `killBoss` (advances stage), `loseLife`, `endGame`, `togglePause`.
- **Accessibility** — a live region (`srLive`) fed through `announce()`, a reduced-motion toggle, and `prefers-reduced-motion` detection.

Gameplay controls (from the in-game hint): mouse to steer and click to fire; arrows/WASD to move; Space to fire; X or Shift for bomb; P or Esc to pause. Touch: drag left to fly, tap FIRE, tap BOMB.

## Conventions

- Keep the game as a single file unless the user asks for a restructure. Adding a bundler, framework, or package manager is a significant change and needs the user's sign-off.
- Preserve the existing style: 2-space indentation, `const`/`let`, plain ES2015+ JavaScript, no transpilation.
- Tune gameplay through the config tables (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, `KILLS_PER_STAGE`) rather than hard-coding magic numbers into logic.
- Any new user-facing state that should persist should use the `skyfighter.*` key prefix.
- Any new visible status change should also be announced through `announce()` for screen-reader users.
- Respect `reducedMotion` when adding screen shake, flashes, or other motion effects.
- Keep the layout working on mobile: the page uses `touch-action: none`, safe-area insets, and a fixed full-viewport container. Check touch controls when changing HUD or overlay markup.

## Verifying changes

There are no automated tests. To verify a change:

1. Open `index.html` in a browser (or serve it locally) and check the console for errors.
2. Play through at least one stage, including a boss fight, a bomb, a life loss, and game over.
3. Check each difficulty level if the change touches balance.
4. Check the layout at a phone-sized viewport and with touch controls, and confirm there is no horizontal page scroll.
5. Toggle mute, pause, and reduced motion to confirm they still work.

## Git

- Default development branch for this work: `claude/gifted-tesla-opo0pc`.
- Commit with clear messages describing the gameplay or code change.
- Do not open pull requests unless the user asks for one.
