# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a small, dependency-free front-end project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on an HTML5 canvas.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: CSS (`<style>`, lines ~10–359), markup (from line ~361), and JavaScript (`<script>`, lines ~422–1669). |
| `README.md` | Placeholder title only. |

There is no build step, package manager, test suite, or linter. Do not add one unless asked.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Architecture of `index.html`

The script is one IIFE with `'use strict'`. Nothing is exported to `window`; keep it that way.

Major sections, in file order (search for the `// ----------` comment banners):

- **DOM references and constants**: cached element lookups, `STORAGE_*` localStorage keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (one per stage), `ENEMY_TYPES`, `BOSSES`.
- **Adaptive performance**: `trackPerf` samples frame times and steps a `quality` level (0–2) up or down. Drawing code consults it to reduce effects.
- **Audio**: a lazily created `AudioContext` (`ensureAudio`), with `beep`/`noiseBurst` helpers and a named `sfx` table. Audio starts only after a user gesture.
- **Input**: `keys` map keyed by `e.code`; mouse/pointer steering and firing; a virtual joystick and FIRE/BOMB buttons for touch devices.
- **Canvas sizing**: fixed logical resolution `BASE_W × BASE_H` (480×720), scaled to fit the window, capped at 1.6×.
- **Game state**: `resetGame`, `update(dt)`, `draw()`. The main loop is `loop(now)` at the bottom, which clamps `dt` to 50 ms and calls `update` then `draw` only while running and not paused.
- **Weapons and power-ups**: `fireWeapon`, `applyPowerUp`, `useBomb`. Weapon levels 1–3; limited smart bombs.
- **Bosses**: `spawnBoss`, `updateBoss`, `killBoss`. `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireCross`, `fireHomingSeeded`, …).
- **Lifecycle**: `loseLife`, `endGame`, `togglePause`.
- **Rendering**: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`. HUD is DOM-based (`renderHearts`, `renderBombs`, overlays, combo/stage banners), not canvas text.

### Persistence

Settings and high score use `localStorage` under the `skyfighter.*` prefix: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are direct calls, so keep them simple and defaulted.

### Accessibility

- `prefers-reduced-motion` seeds the reduced-motion toggle. When it is on, a `body.reduced-motion` class tones down CSS effects, and `screenShake` caps shake duration at 0.1 s.
- The `#srLive` live region receives text via `announce()`. Use it for important state changes.

## Conventions

- **Single file, vanilla JS.** Do not introduce frameworks, bundlers, or external CDN scripts without an explicit request.
- **Match the local style**: 2-space indent, single quotes, `const`/`let`, arrow functions for callbacks, short section banners of the form `// ---------- Name ----------`.
- **Keep the fixed logical resolution.** Game coordinates are in 480×720 units. Do not change `BASE_W`/`BASE_H` casually, since spawn ranges and HUD positions depend on them.
- **Use `dt` in seconds** for all movement and timers so behaviour stays frame-rate independent.
- **Gate new tunables in `DIFFICULTIES` or the relevant constant table** rather than hard-coding numbers in `update`.
- **Keep the mobile layout working.** The page sets `user-scalable=no` and uses `viewport-fit=cover`. Test touch paths when you change input code.
- **Preserve the `quality` levels** when adding expensive effects. New particle or glow code should respect the current quality.

## Working on the game

- Verify changes by loading the page in a browser and playing a few stages. There are no automated tests.
- Check the browser console for errors after changes, especially around audio (the `AudioContext` is created on first user gesture) and canvas sizing.
- When adding a boss pattern, add it to `BOSS_PATTERNS` and make sure it is reachable from `spawnBoss`/`updateBoss`.
- When adding a power-up, update `maybeSpawnPowerUp`, `applyPowerUp`, and any HUD text that names it.

## Git

- Development branch for this work: `claude/gifted-tesla-p7qkn4`.
- Commit messages are imperative and descriptive, in the style of the existing history (for example, `Add Sky Fighter: ...`).
- Do not open pull requests unless explicitly asked.
