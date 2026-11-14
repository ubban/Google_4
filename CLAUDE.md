# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style
vertical shoot-'em-up. The entire game lives in one file, `index.html`
(inline `<style>`, markup, and a single `<script>` block, roughly 1,700 lines).

Repository contents:

- `index.html` — the complete game (HTML, CSS, JavaScript, canvas rendering, audio, input).
- `README.md` — minimal title-only README.
- `CLAUDE.md` — this file.

There is no build step, package manager, test suite, linter, or CI configuration.

## Running the game

No tooling is required. Open the file directly in a browser:

```
xdg-open index.html        # Linux
open index.html            # macOS
```

For mobile or cross-device testing, serve the directory over a local HTTP server
(for example `python3 -m http.server 8000`) and load it from another device on the
same network. Touch controls only appear on touch-capable devices.

## Code architecture (`index.html`)

The script is a single IIFE-style block of plain JavaScript that runs a
`requestAnimationFrame` loop (`loop(now)`) driving `update(dt)` and `draw()`.

Key sections, in file order:

- **Constants and config** (~lines 463–516): `STORAGE_*` localStorage keys,
  `DIFFICULTIES` (easy / normal / hard tuning: spawn rate, enemy speed, enemy fire
  chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`,
  `ENEMY_TYPES`, and `BOSSES`.
- **Persistence**: best score, mute, difficulty, and reduced-motion preference are
  stored in `localStorage` under the `skyfighter.*` keys. Reads are not wrapped in
  try/catch, so they can throw in browsers with storage blocked.
- **Audio** (~lines 543–680): a lazily created `AudioContext` (`ensureAudio`) and
  synthesized sound effects (`beep`, `noiseBurst`, and the `sfx` table). No audio
  files are loaded.
- **Canvas sizing** (~line 753): fixed logical resolution `BASE_W = 480`,
  `BASE_H = 720`, scaled to the viewport by `resize()`.
- **Input** (~lines 769–862): keyboard, touch joystick (`joystickStart/Move/End`),
  on-screen fire and bomb buttons, and mouse aiming on the canvas.
- **Game state** (~lines 865–911): module-level `let` variables (score, lives,
  bombs, stage, chain/multiplier, timers). `resetGame()` restores the initial state.
- **Entities and systems**: player and enemy drawing (`drawPlane`, `drawBossShip`),
  spawning (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`), collisions
  (`rectsOverlap`), scoring and chains (`addScore`, `breakChain`), and bombs
  (`useBomb`).
- **Bosses**: `BOSS_PATTERNS` is a list of bullet-pattern generators
  (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, …) used by
  `updateBoss`. `killBoss` advances the stage.
- **Rendering**: `draw()` renders everything to the 2D canvas. `quality`
  (0 full, 1 medium, 2 low) is adjusted at runtime by `trackPerf(dt)` to keep the
  frame rate up.
- **Game flow**: `startBtn` starts a run, `loseLife`, `endGame`, and `togglePause`
  handle state transitions. `announce()` writes to an ARIA live region for
  screen readers.

### Controls

| Action        | Keyboard                    | Touch / mouse                    |
|---------------|-----------------------------|----------------------------------|
| Move          | Arrow keys or W/A/S/D       | Virtual joystick / mouse on canvas |
| Fire          | Space (hold)                | Fire button (hold) / mouse down  |
| Smart bomb    | X or Shift                  | Bomb button                      |
| Pause         | P or Escape                 | Pause button                     |
| Mute          | Mute button                 | Mute button                      |

## Conventions

- Keep the game self-contained in `index.html`. Do not add a build system,
  package manager, or external runtime dependencies unless the user asks for it.
- Only load external resources the page already relies on. The current page loads
  no scripts or stylesheets from the network; its only external reference is an
  inline SVG namespace string.
- Use the existing naming style: `camelCase` for functions and variables,
  `UPPER_SNAKE_CASE` for constants, and `STORAGE_*` for localStorage keys.
- Tuning values (spawn rates, speeds, HP, bomb counts) belong in the config
  objects near the top of the script, not hard-coded in update logic.
- Preserve mobile support: keep the touch controls working, and keep the
  `{ passive: false }` option on touch listeners that call `preventDefault()`.
- Respect the reduced-motion setting when adding screen shake, flashes, or other
  motion effects (see `reducedMotion` and `screenShake`).
- Keep UI announcements flowing through `announce()` so screen-reader users get
  the same information.

## Making changes

- Read the relevant section of `index.html` before editing; the file is large and
  section comments (`// ---------- … ----------`) mark its boundaries.
- Because there are no automated tests, verify changes by loading the game in a
  browser and playing through: start, fire, bomb, take damage, boss fight, pause,
  game over, and restart. Check both desktop keyboard and touch layouts if the
  change touches input or the HUD.
- Check the browser console for errors after each change.
- Keep commits focused. The git history so far is two commits (`Initial commit`
  and the Sky Fighter addition); follow a short, imperative subject-line style.

## Git and branches

- Develop on the branch assigned for the session (currently
  `claude/gifted-tesla-qmfib7`). Do not push to other branches without permission.
- Do not open a pull request unless the user explicitly asks for one.
