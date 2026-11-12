# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

- `index.html` — the entire game: markup, CSS, and JavaScript in one self-contained file (~58 KB).
- `README.md` — placeholder title only.

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code structure of `index.html`

- The game logic sits in a single IIFE (`(function () { ... })()`) inside the final `<script>` block.
- **Configuration constants** are declared near the top of the script: `DIFFICULTIES` (easy/normal/hard tuning: spawn rate, enemy speed, shot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, and `BOSSES`.
- **Persistence** uses `localStorage` under keys prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Read and write these through the existing variables so the defaults stay consistent.
- **Game loop:** `update(dt)` advances the simulation each frame; the draw functions (`drawPlane`, `drawBossShip`, and related helpers) render it. Shared helpers include `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `maybeSpawnPowerUp`, `applyPowerUp`, `fireWeapon`, and `useBomb`.
- **Boss attack patterns** are separate functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`.
- **Audio** is synthesized with the Web Audio API (`beep`, `noiseBurst`, `ensureAudio`). No audio files are used. Audio must start only after a user gesture, so keep calling `ensureAudio()` from input handlers.
- **Input** covers keyboard (arrows/WASD move, Space fire, X or Shift bomb, P/Esc pause), mouse (move to steer, click to fire), and touch (left-side joystick, FIRE and BOMB buttons).
- **Layout:** the game renders at a fixed logical size (`BASE_W = 480`, `BASE_H = 720`) and is scaled by `resize()` to fit the viewport. Keep gameplay coordinates in these base units.

## Conventions

- Keep the game as a single self-contained HTML file unless the user asks for a split. Don't add external dependencies or CDN scripts without a reason.
- Match the existing style: `const` and `let`, two-space indentation, and small named helper functions.
- Preserve accessibility and mobile support when changing input or HUD code. The reduced-motion setting (`reducedMotion` in JS, `body.reduced-motion` in CSS) weakens screen shake (`screenShake`) and turns off the CSS animations for banners, flashes, vignette, and combo pops; respect it in any new effects. The `announce()` helper feeds screen readers.
- Keep the mobile layout working: `touch-action: none` and `user-select: none` are intentional on the game surface.

## Verifying changes

Since there is no test suite, verify by running the game:

1. Load `index.html` in a browser (or Playwright/Chromium headless) and check that the console has no errors.
2. Exercise the change directly: movement, firing, bombs, weapon power-ups, stage transitions, boss fights, and game over/restart.
3. For visual changes, check desktop and phone-sized viewports.

If you cannot run the game in the environment, say so explicitly and describe what you verified by reading the code.

## Git

- Development branch for automated work: `claude/gifted-tesla-amx7t2`. Do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
