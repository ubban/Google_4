# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page browser game, **Sky Fighter**: a Raiden-style vertical shoot-em-up.
There is no build step, no package manager, and no test suite. The whole game lives in one file:

- `index.html` (~1,670 lines) — all markup, CSS, and JavaScript, with the game logic in one IIFE.
- `README.md` — a placeholder title only.

Open `index.html` directly in a browser to play. There are no dependencies to install.

## Repository layout

```
index.html   # The entire game: <style> (line ~10), HTML (~362), <script> (~422)
README.md    # Minimal placeholder
```

## Structure of `index.html`

**Markup (lines ~362–420)**
- `#container` holds a single `<canvas id="gameCanvas">` plus HUD overlays: `#hud`, `#bossBarWrap`, `#comboPop`, `#stageBanner`, `#topBar`, `#vignette`, `#flash`.
- `#touchControls` (joystick + FIRE/BOMB buttons) is shown only on touch devices.
- `#overlay` / `#overlayCard` is the start, pause, and game-over screen. It includes the difficulty buttons (`.diffBtn`) and the controls hint text.

**Script (lines ~422–1668)**: one `(function () { 'use strict'; ... })();` IIFE. Sections, in order:

| Section | Approximate lines | Contents |
|---|---|---|
| DOM refs | 424–461 | `getElementById` lookups, storage keys |
| Config | 485–527 | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio | 545–678 | `ensureAudio`, `beep`, `noiseBurst`, `EXPLODE_SFX` (Web Audio, no asset files) |
| UI helpers | 680–750 | hearts/bombs rendering, flashes, combo and stage banners, shake, difficulty switch |
| Canvas sizing | 753–783 | `BASE_W = 480`, `BASE_H = 720`, `resize()` |
| Touch / pointer input | 785–850 | joystick and canvas-point helpers |
| Game state | 875–912 | `resetGame()` |
| Drawing primitives | 913–1021 | `drawPlane`, `drawBossShip` |
| Spawning & effects | 1022–1147 | `spawnEnemy`, `spawnBoss`, power-ups, explosions, scoring, combo chain, bombs, `fireWeapon` |
| Update | 1148–1303 | `update(dt)` — movement, collisions, enemy AI, stage progression |
| Power-ups | 1304–1323 | `applyPowerUp` |
| Boss attacks | 1324–1410 | `fire*` bullet-pattern helpers and the `BOSS_PATTERNS` array |
| Boss / life / end | 1411–1506 | `updateBoss`, `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Rendering | 1507–1639 | `drawClouds`, `draw()` (per-theme sky, sprites, HUD-adjacent effects) |
| Main loop | 1640–1668 | `loop(now)` with `requestAnimationFrame`, start button wiring |

Key facts:
- **Fixed logical resolution** of 480×720. The canvas is scaled to fit the viewport in `resize()`. Game coordinates always use the logical size, not CSS pixels.
- **Time step** is delta-time based and clamped to 50 ms (`Math.min(dt, 0.05)`) so a stalled tab does not teleport objects.
- **Adaptive quality**: `trackPerf()` samples the last 45 frame times and steps `quality` between 0 (full), 1 (medium), and 2 (low) to hold roughly 40–56 FPS.
- **Stages**: after `KILLS_PER_STAGE` (18) kills a boss spawns (`bossWarned`). Defeating it (`killBoss`) enters the next stage. The stage's theme is `THEMES[(stage - 1) % THEMES.length]`. Bosses come from `BOSSES`, and their attacks come from `BOSS_PATTERNS`.
- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, shot chance, starting lives, starting bombs, and boss HP multiplier. Values live in `DIFFICULTIES`.
- **Enemy types**: `grunt` (stage 1+), `interceptor` (stage 1+, weaving), `tank` (stage 2+, 4 HP).

## Controls

| Input | Action |
|---|---|
| Mouse move / drag | Steer |
| Left click (hold) | Fire |
| Arrows / WASD | Move |
| Space | Fire (hold) |
| X or Shift | Bomb |
| P or Esc | Pause / resume |
| Touch: left zone | Virtual joystick |
| Touch: FIRE / BOMB | Fire / bomb buttons |

Keyboard state is stored in `keys[e.code]`. Input handlers are `keydown`/`keyup` on `window`, plus `pointer`/`touch` handlers for mouse and mobile.

## Persistence

Settings are stored in `localStorage` with the `skyfighter.*` prefix:

| Key | Meaning |
|---|---|
| `skyfighter.bestScore` | High score (written on game over) |
| `skyfighter.muted` | `'1'` when sound is muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` or `'0'`; when unset, follows `prefers-reduced-motion` |

Reduced motion toggles the `reduced-motion` class on `<body>`. `screenShake()` dampens shake (about 15% magnitude, capped duration), and `flashHit()` / `flashBomb()` return early.

## Accessibility

- `#srLive` is an `aria-live="polite"` region. Use `announce(text)` for important state changes (stage, game over, pause).
- Keep the `sr-only` class and `role` attributes on controls when editing the markup.
- Honor `reducedMotion` for any new screen shake, flash, or particle effect.

## Working on this codebase

- **Edit in place.** Keep all game code inside the existing IIFE. Do not introduce globals.
- **No dependencies.** Do not add npm packages, bundlers, or external CDN scripts unless the user asks for them. Audio is synthesized with Web Audio, and graphics are drawn on canvas.
- **Match the style**: 2-space indentation, single quotes in JS, `const`/`let` (no `var`), section headers in the form `// ---------- Name ----------`, and arrow functions for callbacks.
- **Add tuning values to the config tables** (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding them in logic.
- **Be careful with the update/draw split.** Game state changes belong in `update(dt)` and its helpers. `draw()` should only read state.
- **Verify by playing.** There are no automated tests. After changes, load `index.html` in a browser and check start, fire, a bomb, pause, a stage transition, a boss fight, and game over. Also check touch layout at phone width.
- **Line numbers drift.** The line references above are approximate. Search for function names instead of trusting the numbers.

## Git

- Development branch for this work: `claude/gifted-tesla-qf0vbv`. Push there, not to `main`.
- Commit messages: short imperative summary, with a body when the reason is not obvious.
