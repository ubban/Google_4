# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 is a small static web project. Its only game is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

- `index.html` — the entire game: markup, CSS (inline `<style>`), and JavaScript (one inline `<script>`).
- `README.md` — a one-line placeholder.

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add tooling unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static file server. No install step is needed.

## Code layout inside `index.html`

The script is one IIFE with `'use strict'`. Sections are marked with `// ---------- Name ----------` comments. Keep new code in the matching section.

| Section (approx. line) | Contents |
| --- | --- |
| DOM lookups (~423) | `const x = document.getElementById(...)` for every element the script touches |
| Storage keys (~463) | `STORAGE_*` constants for `localStorage` |
| Adaptive performance (~473) | `trackPerf` lowers/raises a `quality` level based on frame time |
| Tuning tables (~485) | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES` |
| Audio (~545) | `ensureAudio`, `beep`, `noiseBurst`; sound effects are synthesized with WebAudio, no audio files |
| Settings and HUD (~680–760) | hearts/bombs rendering, flashes, banners, `setDifficulty`, mute and motion toggles |
| Layout (~753) | `BASE_W = 480`, `BASE_H = 720`; `resize()` scales the canvas to fit the viewport |
| Input (~766) | keyboard, mouse, and touch handlers; virtual joystick and fire/bomb buttons for touch |
| Game state (~875) | `resetGame()` and the `player`, enemy, bullet, and boss state |
| Drawing (~913–1000, ~1507–1640) | `drawPlane`, `drawBossShip`, `draw()`, `drawClouds` |
| Spawning and combat (~1022–1150) | `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb` |
| Update (~1148) | `update(dt)` runs the per-frame simulation |
| Boss patterns (~1304–1410) | power-ups and `BOSS_PATTERNS` (radial, spread, spiral, cross, homing, double-ring bullet patterns) |
| Lifecycle (~1441–1505) | `killBoss`, `loseLife`, `endGame`, `togglePause` |
| Main loop (~1640) | `requestAnimationFrame` loop; `dt` is clamped to 0.05 s |

Rendering is immediate-mode canvas 2D. Game logic and drawing are separate functions called from the loop: `update(dt)` then `draw()`.

## Conventions

- **Vanilla JS only.** No frameworks, no external libraries, no CDN scripts. Keep everything in `index.html` unless the user asks for a split.
- **Match the surrounding style.** Two-space indent, single quotes in JS, `const` by default, arrow functions for callbacks, and `function` declarations for top-level helpers.
- **Use the tuning tables.** Balance changes (speeds, HP, spawn rates, score values, lives, bombs) belong in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES` rather than inline magic numbers.
- **Time-based motion.** Movement and timers use `dt` in seconds, so they stay consistent across frame rates. Do not tie gameplay to frame counts.
- **Respect the quality level.** Expensive effects should check the `quality` value and `reducedMotion` so they degrade on slow devices and for users who prefer reduced motion.
- **Accessibility.** Announcements go through `announce()` into the `#srLive` live region. Keep the `aria-*` attributes and the `prefers-reduced-motion` handling intact when editing markup.

## Persistence

`localStorage` keys (all prefixed `skyfighter.`):

- `bestScore` — high score, written in `endGame()`.
- `muted` — `'1'` or `'0'`.
- `difficulty` — `easy`, `normal`, or `hard`.
- `reducedMotion` — `'1'` or `'0'`; when unset, the `prefers-reduced-motion` media query decides.

Storage access is not wrapped in try/catch. If you add more storage use, consider that private browsing or blocked storage can throw.

## Controls (for manual testing)

- **Move:** Arrow keys or `A`/`D`; on touch, the left-side joystick zone.
- **Fire:** `Space` (hold), or the fire button on touch/mouse.
- **Bomb:** `X` or `Shift`, or the bomb button.
- **Pause:** `P` or `Escape`.
- **Weapons:** A weapon power-up raises the weapon level up to 3. Past level 3 it switches between `vulcan` and `laser` and resets the level to 1 (`applyPowerUp`).

## Verifying changes

With no tests or linter, verify by playing:

1. Open `index.html` in a desktop browser and a mobile-width viewport (or device emulation).
2. Check that the game starts, fires, takes damage, and ends with a score saved to `localStorage`.
3. Check each difficulty, the mute and motion toggles, and pause/resume.
4. Reach at least one boss to exercise `BOSS_PATTERNS` and `killBoss`.
5. Check the layout at phone width: no horizontal scroll, and the touch controls remain usable.

Mention in your final message if you could not run the game in a browser.

## Git

- Develop on the branch the session names (currently `claude/gifted-tesla-2db06d`). Do not push to `main` without asking.
- Write short, descriptive commit messages in the imperative mood.
- Do not open a pull request unless the user asks for one.
