# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

The whole game lives in one file, `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>`). `README.md` is a one-line stub.

There is no build system, package manager, bundler, test suite, or linter. Nothing needs installing.

## Running it

- Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`) and visit it.
- Verify changes by loading the page and playing; there are no automated tests to run.

## Repository layout

```
index.html   # The entire game: CSS, HTML overlay/HUD, and game JS
README.md    # Title only
```

## Architecture of `index.html`

- **Canvas and scaling**: fixed logical size `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the window (max scale 1.6).
- **Game loop**: `requestAnimationFrame(loop)` drives `update(dt)` then `draw()`. `trackPerf(dt)` lowers a `quality` level (0 full / 1 medium / 2 low) when frame times are poor.
- **Difficulty**: `DIFFICULTIES` (easy / normal / hard) sets spawn rates, enemy speed, shot chance, lives, starting bombs, and boss HP multiplier.
- **Progression**: `KILLS_PER_STAGE = 18`. After that many kills, a boss is spawned (`spawnBoss`) and the stage advances. `THEMES` cycles the sky palette per stage; `BOSSES` supplies the boss roster (VANGUARD → TITAN).
- **Enemies**: `ENEMY_TYPES` (grunt, interceptor, tank). `spawnEnemy()` gates types by `minStage`.
- **Weapons**: `player.weapon` is `vulcan` or `laser`, with `player.level` 1–3. Weapon pickups raise the level; past level 3 they toggle the weapon type. `fireWeapon()` implements both.
- **Bombs**: `useBomb()`, triggered by `X` or Shift (or the touch bomb button). Pickups add to `bombCount` up to the difficulty cap.
- **Boss patterns**: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, driven by `updateBoss(dt)`.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`. Screen shake and vignette/flash effects are gated by `reducedMotion`.
- **Audio**: Web Audio API, created lazily in `ensureAudio()` (browsers need a user gesture first). Sound effects are generated procedurally by `beep()` and `noiseBurst()`; there are no audio files. `sfx` holds the named effects.
- **Persistence**: `localStorage` keys, all prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are direct; keep them that way unless adding a helper.
- **Accessibility**: `announce()` writes to the `srLive` live region. Reduced motion defaults to the `prefers-reduced-motion` media query until the user overrides it.

## Controls

| Action        | Keyboard                          | Touch / mouse                                  |
|---------------|-----------------------------------|------------------------------------------------|
| Move          | Arrow keys / WASD                 | Drag left side (joystick) / mouse pointer      |
| Fire          | Space (held)                      | FIRE button (touch) / mouse click on canvas    |
| Bomb          | X or Shift                        | BOMB button                                    |
| Pause         | P or Escape                       | Pause button                                   |

Mute, reduced-motion toggle, and difficulty are set from the overlay/HUD buttons.

## Conventions and guidelines

- **Keep it a single file** unless the user asks for a split. Don't introduce a build step, framework, or dependencies without a clear reason.
- **Preserve the existing style**: 2-space indentation, semicolons, `const`/`let`, plain DOM APIs, no jQuery.
- **Tune gameplay through the constant tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hardcoding numbers into logic.
- **Respect the reduced-motion and quality paths** when adding visual effects: new shake, flash, or particle effects should check `reducedMotion` and `quality`.
- **Touch support is first-class**: any new input must work on both keyboard and touch. Keep `touch-action: none` and the `preventDefault` calls on game controls.
- **Mobile layout**: the page must work at phone width with no horizontal scroll; the HUD uses `clamp()` font sizes and safe-area insets.
- **Don't break the saved-data keys**: renaming a `skyfighter.*` localStorage key resets players' best scores and settings.

## Working agreements

- Verify gameplay changes in a real browser, not just by reading the code: start a game, exercise the changed path, and check the console for errors.
- Keep commits focused. Commit and push only when asked, and to the branch the session specifies.
- Don't add a PR unless explicitly requested.
