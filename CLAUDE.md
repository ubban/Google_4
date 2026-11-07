# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game is a single self-contained file:

- `index.html` (~1,670 lines): markup, CSS (in `<style>`), and game logic (in one IIFE inside `<script>`).
- `README.md`: a one-line placeholder. No other source, build, or asset files exist.

There is no build step, no package manager, no framework, no external assets, and no test suite. Audio is synthesized at runtime with the Web Audio API.

## Running the game

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There is nothing to install or compile.

## Controls (from the in-game hint)

- Mouse: move to steer, hold click to fire.
- Arrows / WASD: move. Space: fire. X or Shift: smart bomb.
- P or Esc: pause.
- Touch: drag left side to fly (virtual joystick), tap FIRE, tap BOMB.

## Code layout inside `index.html`

The file is organized top-to-bottom roughly as follows. Section headers in the script use `// ---------- Name ----------` comments, so search for those when navigating.

| Area | Where to look |
| --- | --- |
| Markup: HUD, overlay, touch controls, icon buttons | `<body>` (near line 370 onward) |
| CSS (HUD, hearts/bombs, vignette, flash, boss bar, touch UI, `reduced-motion` styles) | `<style>` (lines 10–421) |
| DOM element lookups and constants | Top of the IIFE (`const canvas`, `STORAGE_*`) |
| Adaptive performance (`quality` 0/1/2 from frame-time averages) | `trackPerf` |
| Tuning tables | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx`) | `// ---------- Audio ...` |
| Canvas sizing (`BASE_W = 480`, `BASE_H = 720`, scaled via CSS) | `resize()` |
| Input (keyboard, joystick, mouse steering, touch buttons) | `// ---------- Input ----------` |
| Game state, `resetGame()`, `spawnEnemy`, `spawnBoss`, `useBomb`, `fireWeapon` | `// ---------- Game state ----------` onward |
| Per-frame simulation | `update(dt)` |
| Boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) and `updateBoss` | Around lines 1320–1470 |
| Game over / high score (`endGame`) | `endGame()` |
| Rendering (`drawPlane`, `drawBossShip`, `drawClouds`, `draw`) | Around lines 910–1520 |
| Main loop | `loop(now)` at the end; uses `requestAnimationFrame` and clamps `dt` to 0.05 s |

### Game model

- **Player weapons:** `vulcan` and `laser`, each with power levels 1–3. A weapon power-up raises the level; at level 3 it toggles to the other weapon and resets to level 1.
- **Enemies:** `grunt` (1 HP), `interceptor` (weaving, 1 HP), `tank` (4 HP, appears from stage 2).
- **Stages:** 18 kills per stage (`KILLS_PER_STAGE`), then a boss fight. Each stage uses a theme from `THEMES`, cycling through the list. Bosses come from `BOSSES` in order, with `hpMul` scaling.
- **Difficulty:** `easy`, `normal`, `hard` (`DIFFICULTIES`) control spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.

## Persistence

Per-browser settings live in `localStorage` under these keys (defined near the top of the script):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every `localStorage` access should stay tolerant of failures (private mode, blocked storage). If you add new persisted state, add a `STORAGE_*` constant next to the existing ones, using the `skyfighter.` prefix.

## Conventions to keep

- **Keep it a single file.** Do not split `index.html` into modules or add a bundler, package manager, or CDN dependency unless the user explicitly asks for it.
- **Keep the logical canvas at 480×720.** Scaling is handled by `resize()` via CSS; game coordinates should stay in the 480×720 space.
- **Frame-rate independence:** update logic takes `dt` in seconds. Use it for movement and timers rather than per-frame increments, and remember `dt` is capped at 0.05 s.
- **Respect reduced motion and quality levels.** Screen shake, flashes, and glow effects are gated by `reducedMotion` and by the adaptive `quality` level. New visual effects should honor both.
- **Accessibility hooks:** announcements go through `announce()` into the `#srLive` live region. Use it for meaningful state changes (stage changes, boss defeat, game over).
- **Touch and keyboard parity:** new actions should be reachable both by keyboard and by the touch controls, and the in-game hint text (`#controlsHint`) should be updated if controls change.
- **No external audio files.** Sounds are generated through `beep` / `noiseBurst`; add new sounds in the same style.

## Verifying changes

There is no automated test or lint setup. To verify a change:

1. Open `index.html` in a browser (desktop and, if touch-related, a mobile-sized viewport).
2. Play through at least one stage transition, including a boss fight, and confirm the HUD, score, and banners update.
3. Check the browser console for errors.
4. For UI or layout changes, check both a wide window and a phone-sized viewport, since the HUD and touch controls reflow.

If you add a test or tooling setup in the future, document the command here.

## Git

- Development branch for this work: `claude/gifted-tesla-2ar1iw`.
- Write clear, descriptive commit messages that explain the why when it is not obvious.
