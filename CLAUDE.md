# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`, with inline `<style>` and an inline `<script>`. There is no framework, bundler, package manager, build step, or test suite.

`README.md` is a placeholder and contains no setup instructions.

## Running it

Open `index.html` directly in a browser. No server or install is needed.

There are no build, lint, or test commands. To verify a change, load the page and play through it (see "Verifying changes" below).

## Layout of `index.html`

| Section | Approx. lines | Contents |
| --- | --- | --- |
| `<head>` | 1–9 | Meta tags (viewport locked to `user-scalable=no`, `theme-color`), `<title>` |
| `<style>` | 10–360 | All CSS: HUD, overlays, touch controls, `.reduced-motion` and `.sr-only` helpers |
| `<body>` markup | 361–421 | Canvas `#gameCanvas`, HUD elements, overlay, touch controls, `#srLive` live region |
| `<script>` | 422–end | One IIFE (`'use strict'`) containing all game logic |

Within the script, the main areas are:

- **Config constants** (~line 485–530): `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES`.
- **Persistence** (~line 463): `localStorage` keys prefixed `skyfighter.` for best score, mute, difficulty, and reduced motion.
- **Audio** (~line 545–680): WebAudio synth. `beep()`, `noiseBurst()`, and the `sfx` object. Audio is created lazily by `ensureAudio()`, which must run from a user gesture.
- **Input** (~line 769–865): keyboard (`keys[e.code]`), touch joystick plus fire/bomb buttons, and mouse on the canvas.
- **Game state and flow** (~line 871–1150): `resetGame()`, `update(dt)`, spawning, `useBomb()`, `fireWeapon()`.
- **Bosses** (~line 1304–1456): `applyPowerUp`, the `BOSS_PATTERNS` bullet patterns, `updateBoss`, `killBoss`.
- **Lifecycle** (~line 1456–1505): `loseLife`, `endGame`, `togglePause`.
- **Rendering** (~line 913–1020, 1507–1640): `drawPlane`, `drawBossShip`, `draw()`, and the parallax `drawClouds`.
- **Main loop** (~line 1640): `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05s, and calls `trackPerf` for adaptive quality.

## Conventions and gotchas

- **Keep it one file.** New code goes in `index.html` unless the user asks for a split. Don't add a build step, bundler, or dependencies without asking.
- **Coordinates:** the logical play field is `BASE_W = 480` × `BASE_H = 720`. `resize()` scales the canvas to fit the viewport, so game logic should use these base units, not CSS pixels.
- **Time-based updates:** all movement and timers use `dt` in seconds. Don't use per-frame increments.
- **Adaptive quality:** `quality` (0 full, 1 medium, 2 low) is driven by `trackPerf`. Rendering that is expensive should respect it.
- **Reduced motion:** honor `reducedMotion` (user toggle, defaulting to `prefers-reduced-motion`) when adding screen shake, flashes, or other motion effects.
- **Accessibility:** announce important state changes through `announce()` (the `#srLive` region). Keep HUD and overlay buttons labeled.
- **Audio gating:** call `ensureAudio()` before playing sound, and only from a user interaction, because browsers block autoplay audio.
- **Mute** is persisted and must stay in sync with `muteBtn` via `updateMuteBtn()`.
- **Best score** is written to `localStorage` in `endGame`. Keep that behavior if you change the end-of-game flow.
- **Touch vs. desktop:** the page supports both. Changes to input handling should be checked on both paths (touch joystick and fire/bomb buttons, plus keyboard and mouse).
- **Phone layout:** the viewport is locked to a single scale, so verify the overlay and HUD still fit on small screens.
- **Don't break the IIFE:** everything is scoped inside `(function () { ... })()`. Don't expose globals unless needed.

## Verifying changes

Because there is no test suite:

1. Open `index.html` in a browser (or serve the directory with any static server).
2. Check the start overlay, then play a run through at least one stage transition and one boss fight.
3. Exercise each input path you touched: keyboard, mouse, and touch controls (use browser device emulation).
4. Confirm best score, mute, difficulty, and reduced-motion settings persist across a reload.
5. Watch the browser console for errors.

For browser automation, Playwright with the pre-installed Chromium works. Avoid running `playwright install`.

## Git

- Development branch for this session: `claude/gifted-tesla-dkjwgp`.
- Commit messages should be short and describe the gameplay or code change.
