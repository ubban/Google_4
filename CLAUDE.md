# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

The whole game lives in a single file, `index.html` (~1,670 lines): HTML markup, inline `<style>`, and one inline `<script>` block wrapped in an IIFE. There is no build step, no package manager, no bundler, and no external assets. Audio is synthesized with the Web Audio API.

Other files:
- `README.md`: one-line project title only.
- `.git/`: the repository. The current branch for development is `claude/gifted-tesla-iqak8i`.

## Running the game

No build is needed. Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/index.html
```

There is currently no test suite, linter, or CI configuration. Verify changes by running the game in a browser and playing through the affected behaviour.

## Code layout (`index.html`)

Rough map of the inline script (line numbers are approximate and will drift):

| Area | What it contains |
|---|---|
| Storage keys (~463) | `skyfighter.*` keys for best score, mute, difficulty, reduced motion |
| `DIFFICULTIES` (~485) | easy / normal / hard tuning: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier |
| `KILLS_PER_STAGE`, `THEMES` (~491–500) | 18 kills per stage; six background themes cycled by stage |
| `ENEMY_TYPES` (~502) | `grunt`, `interceptor` (weaving), `tank` (4 HP, stage 2+) |
| `BOSSES` (~508) | six stage bosses (VANGUARD → TITAN), each with a colour scheme and HP multiplier |
| Canvas sizing (~753) | fixed logical resolution `BASE_W = 480`, `BASE_H = 720`, scaled to fit the window (max 1.6×) |
| Input (~769–861) | keyboard, touch joystick and buttons, mouse |
| `resetGame`, `spawnEnemy`, `spawnBoss` | state reset and entity spawning |
| `update(dt)`, `updateBoss(dt)` | per-frame simulation; `dt` is clamped to 50 ms |
| `draw()`, `drawPlane`, `drawBossShip` | canvas rendering |
| `loop(now)` (~1640) | `requestAnimationFrame` loop; runs `update` + `draw` only when running and not paused |

### Controls

- Move: arrow keys or `WASD`, or the on-screen joystick on touch devices.
- Fire: `Space`, or hold the fire button (mouse or touch).
- Bomb: `X` or `Shift`, or the bomb button.
- Pause: `P` or `Escape`.
- Weapons: picking up a `weapon` powerup raises the weapon level (1→3); past level 3 it toggles between `vulcan` and `laser` and resets the level to 1.

### Persistent state

Only `localStorage` is used, under the `skyfighter.*` keys listed above. Reads and writes are direct calls, not wrapped in `try/catch`. Keep that in mind if you touch them: storage can throw in private or restricted browsing contexts.

### Adaptive performance

`trackPerf(dt)` (~476) monitors frame timing. Preserve it when changing the render or update loop.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a restructure. Do not add a bundler, framework, or npm dependencies without asking.
- Keep new code inside the existing IIFE and follow its style (2-space indent, `const`/`let`, arrow functions for handlers).
- Tunable gameplay values belong in the existing config objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than as magic numbers scattered through `update`/`draw`.
- Preserve mobile behaviour: the layout uses `viewport-fit=cover`, safe-area insets, `touch-action: none`, and a touch control overlay. Check touch and keyboard paths when changing input.
- Preserve the `prefers-reduced-motion` / `reducedMotion` option when adding screen shake, flashes, or other motion effects.
- Keep accessibility hooks intact (the `srLive` live region and `sr-only` class).

## Working in this repo

- Develop on the designated branch for the session; push with `git push -u origin <branch>`. Do not push to other branches without explicit permission.
- Do not open pull requests unless the user asks for one.
- Commit messages in this repo are short, imperative summaries, for example `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.
- Before finishing a change, open the game in a browser (or describe clearly what could not be verified) and confirm that the HUD, pause/restart flow, and game-over screen still work.
