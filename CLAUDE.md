# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up that runs as a single self-contained HTML5
canvas page.

| File        | Purpose                                                        |
|-------------|----------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS, and one inline `<script>` block |
| `README.md` | Minimal repo title; no setup instructions yet                  |

There is no build step, no package manager, no test suite, and no external
dependencies. The game makes no network requests and loads no CDN assets.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static
server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture of `index.html`

- **Layout:** a 480×720 logical canvas (`BASE_W`, `BASE_H`) scaled to fit the
  viewport. Mobile controls (virtual joystick, FIRE and BOMB buttons) are shown
  on touch devices. Viewport is locked (`user-scalable=no`, `touch-action: none`).
- **Single IIFE:** all game code lives in one `(function () { ... })()` block.
  Helper functions and state are not exported and not modularized.
- **Main loop:** `loop(now)` drives `update(dt)` then `draw()` via
  `requestAnimationFrame`.
- **Adaptive quality:** `trackPerf` samples frame times and steps `quality`
  between full, medium, and low to keep frame rate up on slow devices.
- **Audio:** synthesized with the Web Audio API (`beep`, `noiseBurst`, the `sfx`
  object). No audio files. Audio starts only after a user gesture (`ensureAudio`).
- **Difficulty:** `DIFFICULTIES` table (`easy`, `normal`, `hard`) sets spawn
  rates, enemy speed, enemy fire chance, starting lives and bombs, and boss HP
  multiplier. Stored under `skyfighter.difficulty`.
- **Stages:** `KILLS_PER_STAGE` kills advance the stage. Each stage uses a
  palette from the sky theme list and a boss design from the boss list
  (`VANGUARD`, `BEHEMOTH`, `SPECTER`, `DREADNOUGHT`), each with its own HP
  multiplier and attack patterns (`fireRadial`, `fireAimedSpread`, `fireCross`,
  `fireHomingSeeded`, and others).
- **Weapons and power-ups:** Vulcan and Laser weapons, each with power levels.
  Power-ups include weapon level, bombs, and extra life. Smart bombs clear the
  screen and damage bosses.
- **Scoring:** kill-chain score multiplier (`addScore`, `breakChain`).

### Persistence (localStorage keys)

| Key                          | Meaning                       |
|------------------------------|-------------------------------|
| `skyfighter.bestScore`       | High score                    |
| `skyfighter.muted`           | Mute flag (`'1'` or `'0'`)    |
| `skyfighter.difficulty`     | Selected difficulty name      |
| `skyfighter.reducedMotion`   | Reduced-motion override; defaults to the OS `prefers-reduced-motion` setting |

Always wrap `localStorage` access in try/catch if you add new reads or writes,
since it can throw in private windows or when storage is blocked.

### Controls

- **Keyboard:** arrow keys or WASD to move; Space to fire; `X`, Shift, or the
  bomb button to detonate a smart bomb; `P` or Escape to pause.
- **Mouse:** move over the canvas to steer; hold the mouse button to fire.
- **Touch:** virtual joystick on the left, FIRE and BOMB buttons on the right.

## Conventions

- **Keep it a single file** unless the user asks for a split. The project is
  deliberately dependency-free and opens directly from the filesystem.
- **Do not add a build step, bundler, or package manager** without asking first.
- **Match the existing style** inside the script: two-space indentation, `const`
  and `let`, arrow functions for callbacks, and short helper names
  (`spawnEnemy`, `fireWeapon`, `renderHearts`).
- **Use the logical canvas coordinates** (`BASE_W` × `BASE_H`). Do not hard-code
  pixel sizes from the display.
- **Respect reduced motion:** new visual effects such as screen shake, flashes,
  or large particle bursts should check the `reducedMotion` flag.
- **Keep the mobile path working.** Any new control must have a touch path as
  well as keyboard and mouse.
- **Keep the accessibility hooks.** `announce()` feeds the screen-reader live
  region. Use it for important state changes such as stage starts and game over.

## Verifying changes

There are no automated tests. To check a change:

1. Load `index.html` in a browser and start a game.
2. Exercise the path you changed: movement, firing, bombs, a stage transition,
   a boss fight, pause, and game over.
3. Check at least one desktop viewport and one narrow mobile viewport for HUD
   and control layout.
4. Confirm there are no console errors on load or during play.

If you add a test harness later, document how to run it here.

## Git workflow

- Develop on the branch named for the session. Do not push to other branches
  without explicit permission.
- Keep commit messages short in the imperative mood, with a body when the change
  needs context.
- Do not open a pull request unless the user asks for one.
