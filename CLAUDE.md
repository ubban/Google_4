# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up with weapon switching, smart bombs, and stage bosses. The entire game lives in one self-contained file, `index.html` (HTML, CSS, and a single inline `<script>` block, about 1,670 lines). `README.md` is a placeholder.

## Repository layout

- `index.html`: the whole game. Start-screen markup and styles are at the top, the game script is in the `<script>` block near line 422.
- `README.md`: placeholder title only.
- `CLAUDE.md`: this file.

There is no `package.json`, bundler, framework, test suite, linter, or CI configuration. Do not invent build or test commands. To verify a change, open `index.html` in a browser (or use the Playwright setup in the cloud environment) and play through the affected path.

## Architecture (inside `index.html`)

The script is plain vanilla JavaScript with no modules. Main areas, in rough order of appearance:

- **Constants and config**: `STORAGE_*` keys, `DIFFICULTIES` (easy / normal / hard: spawn rates, enemy speed and fire chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and per-event SFX. All sound is synthesized with WebAudio, so there are no audio asset files. Audio is created lazily on the first user gesture.
- **UI helpers**: HUD rendering (hearts, bombs), flashes, combo popups, stage banners, screen shake, `announce` for screen readers.
- **Input**: keyboard (`keys` map keyed by `KeyboardEvent.code`), mouse steering and fire, touch joystick (`joystickStart/Move/End`) and fire/bomb buttons.
- **Game state**: `resetGame`, `update(dt)` (per-frame simulation), `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, `useBomb`, `breakChain`, `addScore`.
- **Boss logic**: `BOSS_PATTERNS` (bullet pattern generators such as radial, spiral, cross, homing) driven by `updateBoss`, plus `killBoss`.
- **Lifecycle**: `loseLife`, `endGame`, `togglePause`.
- **Rendering**: `draw()` with helpers `drawPlane`, `drawBossShip`, `drawClouds`.
- **Main loop**: `loop(now)` drives `update` and `draw` each frame.

Persistent state is stored in `localStorage` under `skyfighter.*` keys: best score, mute, difficulty, and reduced-motion preference. Keep these keys stable, because changing them resets player data.

## Controls (keep the start-screen hint in sync)

- Keyboard: arrows or WASD to move, Space to fire, X or Shift for bomb, P or Escape to pause.
- Mouse: move to steer, click to fire.
- Touch: drag left side to fly, tap FIRE, tap BOMB.

If you change a control, update the `#controlsHint` paragraph in `index.html` in the same change.

## Conventions

- Keep the game self-contained in `index.html`. Do not add a build step, package manager, or external script or stylesheet unless the user asks. Google Fonts or CDN dependencies are not currently used.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const` and `let`, and small named functions grouped by the sections above.
- Balance changes go in the `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` tables rather than being hard-coded in logic.
- Preserve mobile support: the page uses `touch-action: none`, safe-area insets, and a viewport meta with `user-scalable=no`. Test layout at phone width after touching HUD or canvas sizing (`resize()`).
- Honor the reduced-motion option when adding screen shake, flashes, or particle effects.
- Keep accessibility hooks such as `announce()` working when you add new player-facing events.

## Working in this repo

- Default branch for development in this session is `claude/gifted-tesla-jbfswf`. Push there and do not push to other branches without permission.
- Do not open a pull request unless the user asks for one.
- Commit messages should be short, imperative, and describe the gameplay or code change.
