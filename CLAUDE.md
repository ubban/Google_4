# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is one self-contained file, `index.html` (HTML, CSS, and JavaScript inline, rendered on a 2D `<canvas>`). `README.md` is a placeholder.

- No build step, bundler, package manager, or dependencies. There is no `package.json`.
- No automated test suite or linter is configured.
- Audio is synthesized with the WebAudio API. There are no external asset files, so do not add image or sound files unless the user asks for them.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- To check changes, load the page and play it. Playwright with the pre-installed Chromium works for scripted checks.

## Code layout (`index.html`)

- **Lines ~10–359**: `<style>`. Pill-style HUD, glassmorphic overlay card, touch controls, and the reduced-motion variants.
- **Lines ~360–421**: Markup. Key element IDs include `gameCanvas`, `hud`, `bossBarWrap`, `topBar` (mute, motion, pause), `touchControls`, and `overlay`.
- **Lines ~422–1667**: One IIFE in `<script>`, with `'use strict'`. Sections, in order:
  1. DOM references and storage keys.
  2. Adaptive performance (`trackPerf` lowers quality when frames are slow).
  3. Tuning tables: `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
  4. Audio: `beep`, `noiseBurst`, and the `sfx` map.
  5. HUD helpers and screen effects (`renderHearts`, `renderBombs`, `screenShake`, `flashHit`).
  6. Input: keyboard, mouse, and touch joystick.
  7. Game state and loop: `resetGame`, `update(dt)`, `draw()`, `loop(now)`.
  8. Gameplay: spawning, power-ups, weapons (`fireWeapon`, `useBomb`), boss patterns (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireDoubleRing`, and others), `loseLife`, `endGame`, `togglePause`.

Everything lives in one IIFE, so functions and state are not reachable from the console. When a feature needs debugging access, add it deliberately rather than leaking globals.

## Game rules and constants to keep in sync

When you change any of these, check the matching UI text and the other places that use them:

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal`, `hard` set spawn rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Stages**: `KILLS_PER_STAGE = 18` kills trigger a boss. `THEMES` cycles the sky palette and the stage name. `BOSSES` lists the boss roster with name, colors, and `hpMul`.
- **Enemies**: `grunt`, `interceptor` (weaving, fast), and `tank` (4 HP, unlocked at stage 2). Each has a score value.
- **Weapons**: Two weapon types, `vulcan` and `laser`. Each has three power levels. The weapon power-up raises the level, and after level 3 it switches weapon and resets to level 1 (`applyPowerUp`).
- **Power-ups** (spawned in `maybeSpawnPowerUp`): `life`, `bomb`, `weapon`.
- **Score**: `addScore` applies a kill-chain multiplier (`1 + min(9, floor(chain / 5))`). The chain resets when its 2.2 s timer runs out and when the player is hit (`breakChain`).
- **Player damage**: `loseLife` gives 1.5 s of invulnerability and triggers screen shake and a hit flash.
- **Best score** is saved to `localStorage` only in `endGame`.

## Persistence

`localStorage` keys (defined near the top of the script):

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | Best score (number) |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`. Defaults to the OS `prefers-reduced-motion` setting until the user toggles it |

Keep these keys stable. Renaming one resets saved player data. Wrap any new storage access in try/catch, since storage can be blocked in private windows.

## Controls (keep in sync with `#controlsHint` in the markup)

- **Mouse**: move to steer, click to fire.
- **Keyboard**: Arrows or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause.
- **Touch**: drag on the left zone to fly (joystick), tap FIRE, tap BOMB.

## Conventions

- Keep the game as a single file unless the user asks for a restructure.
- Match the existing style: 2-space indentation, `const`/`let`, arrow functions for callbacks, and short section comments with `// ---------- Name ----------`.
- The canvas uses a fixed logical size (`BASE_W` × `BASE_H`) scaled by `resize()`. Use logical coordinates in game code, not CSS pixels.
- Respect reduced motion: effects such as shake, flash, and pulse should check `reducedMotion` and skip or soften.
- Keep the game playable on phone-sized screens and with the touch controls.
- Use `announce()` for screen-reader updates on important state changes. Existing examples are life loss, boss defeat, game over, and the reduced-motion toggle.

## Git

- Develop on the branch the session specifies (currently `claude/gifted-tesla-kuvnor`). Do not push to other branches without permission.
- Commit messages describe the gameplay or UI change. The existing history uses short titles followed by a bulleted body.
