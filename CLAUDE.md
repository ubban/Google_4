# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only app is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser as a single-file HTML5 canvas game.

- `index.html` — the entire game: inline CSS (`<style>`), markup, and one inline IIFE script (`<script>`). No external assets, libraries, or network calls.
- `README.md` — placeholder title only.

There is no build step, package manager, bundler, linter, or test suite. Do not add one unless asked.

## Running locally

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Architecture of `index.html`

Everything below lives inside one IIFE in the inline `<script>`.

- **DOM references** — cached at the top (`canvas`, `#hud` elements, overlay, touch controls, boss bar).
- **Persistence** — `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Reads and writes are not wrapped in try/catch; keep that in mind if you touch them.
- **Adaptive performance** — `trackPerf` samples frame times and steps a `quality` level (0 full, 1 medium, 2 low) to reduce effects when frame time is high.
- **Tunables** — `DIFFICULTIES` (spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky/cloud/sun palettes), `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES` (six named bosses, cycled by stage), and `BOSS_PATTERNS` (bullet patterns). Prefer changing these tables over hard-coding numbers in logic.
- **Audio** — Web Audio synthesis (`beep`, `noiseBurst`, `ensureAudio`) with no sample files. Audio is created lazily on the first user gesture.
- **Input** — keyboard (`keys` map on `keydown`/`keyup`), mouse/pointer on the canvas, and touch (virtual joystick, fire and bomb buttons).
- **Game state** — `resetGame()` initializes state; `update(dt)` advances simulation; `draw()` renders; `loop(now)` drives both via `requestAnimationFrame`. `dt` is clamped to 50 ms.
- **Weapons and bombs** — `fireWeapon()` (Vulcan/Laser with 3 power levels), `useBomb()` (clears the screen and damages bosses), `applyPowerUp()`.
- **Bosses** — `spawnBoss()`, `updateBoss()`, `killBoss()`. Once `stageKills` reaches `KILLS_PER_STAGE`, the boss warning fires; the boss for a stage is `BOSSES[(stage - 1) % BOSSES.length]`.
- **Scoring** — `addScore()` applies a kill-chain multiplier; `breakChain()` resets it.
- **Pause and end** — `togglePause()`, `loseLife()`, `endGame()`.

## Controls (for reference when testing)

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | Arrow keys or `WASD` | Virtual joystick |
| Fire | `Space` (hold) | Fire button |
| Bomb | `X` or `Shift` | Bomb button |
| Pause | `P` or `Esc` | — |
| Mute | Mute button | — |

## Conventions

- Keep the game self-contained in `index.html` unless the user asks to split it. If you split it, update the paths in this file.
- Keep the code in the existing style: `'use strict'` IIFE, `const` by default, two-space indentation, single quotes in JS.
- Keep the game's DOM IDs and `skyfighter.*` storage keys stable. Changing a key silently resets players' saved best score, mute, difficulty, and motion settings.
- Preserve mobile behavior: `touch-action: none`, safe-area insets, and the responsive `resize()` that scales the canvas to `BASE_W × BASE_H` (480 × 720).
- Respect `prefers-reduced-motion`. Screen shake, flashes, and pulses should be suppressed or reduced when `reducedMotion` is on.
- Keep accessibility hooks: the `srLive` live region is used via `announce()` for important state changes.
- Do not add external CDN scripts, fonts, or images without a reason; the game is intended to be a single offline-capable file.

## Verification

Because there is no test suite, verify changes by:

1. Opening `index.html` in a browser and checking the browser console for errors.
2. Playing through at least one stage, a boss fight, a bomb use, a pause/resume, and a game over.
3. Checking the desktop layout and a narrow (phone-width) viewport.
4. Confirming that the best score, mute, difficulty, and reduced-motion settings persist across a reload.

## Git

- Default development branch for this session: `claude/gifted-tesla-cgo9c1`.
- Use clear, imperative commit subjects that describe the gameplay or code change.
- Do not open pull requests unless the user asks.
