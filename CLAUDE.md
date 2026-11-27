# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-page HTML5 canvas game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up. The whole game lives in one file, `index.html` (~1,670 lines: inline `<style>`, markup, and one inline `<script>` IIFE). `README.md` is a stub.

There is no build step, no package manager, no bundler, no framework, and no test suite. Nothing is installed or compiled.

## Running and testing

- Run it: open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No network access is required; the page has no external scripts, fonts, or images. The favicon is an inline SVG data URI.
- Verify changes by loading the page in a browser and playing. There are no automated tests or linters configured, so do not claim tests pass.
- For JavaScript syntax, you can extract the inline script and run `node --check` on it, but this is optional.

## File layout (`index.html`)

| Region | Approx. lines | Contents |
|---|---|---|
| `<head>` / `<style>` | 1–359 | Meta tags, inline favicon, all CSS (HUD, overlay card, touch controls, boss bar, animations, reduced-motion styles) |
| Markup | 360–421 | `#container` with `#gameCanvas`, `#hud`, `#bossBarWrap`, `#topBar`, `#touchControls`, `#overlay` |
| Script | 422–1667 | One `(function () { 'use strict'; ... })()` wrapping all game logic |

Inside the script, the code is grouped by `// ----------` section comments: adaptive performance, audio, input, game state, rendering (`drawPlane`, `drawBossShip`, `draw`), spawning, weapons, bosses, and the main `loop(now)` driven by `requestAnimationFrame`.

## Key constants and tuning

All tuning values are near the top of the script (around lines 485–516):

- `DIFFICULTIES`: `easy` / `normal` / `hard`. Each sets `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, and `bossHpMul`.
- `KILLS_PER_STAGE = 18`. Reaching this many kills in a stage triggers the boss after a 1.2 s delay.
- `THEMES`: six sky palettes, one per stage (cycled).
- `ENEMY_TYPES`: `grunt`, `interceptor` (fast, weaving), and `tank` (4 HP, appears from stage 2).
- `BOSSES`: six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with `hpMul` multipliers. Attack patterns are in `BOSS_PATTERNS` (around line 1372).
- Base resolution: `BASE_W = 480`, `BASE_H = 720` (around line 753). The canvas scales to fit the viewport.

## Game mechanics (for reference when changing behavior)

- **Weapons:** Vulcan (spread shot) and Laser (piercing lanes). Each has 3 power levels. The weapon-power-up cycle is level 1→2→3, then it switches weapon type and resets to level 1 (`applyPowerUp`, around line 1304).
- **Bombs:** limited stock, used with `X`/`Shift`/the BOMB button. A bomb clears all enemy bullets and all enemies (+5 score each) and deals 24 damage to the boss (`useBomb`).
- **Lives:** hearts shown in the HUD. A hit calls `loseLife()`, which costs one life, grants 1.5 s of invulnerability (`player.invuln`), and breaks the kill chain. Losing the last life ends the game.
- **Scoring:** kill-chain multiplier (`chain` → `multiplier = 1 + min(9, floor(chain / 5))`). The chain breaks after 2.2 s with no kills (`breakChain`).
- **Power-ups:** `life`, `bomb`, and `weapon`.
- **Game over:** `endGame()` updates the best score in `localStorage` if beaten.

## Persistence

`localStorage` is used with these keys, all prefixed `skyfighter.`:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Every read and write must tolerate storage being unavailable (private mode, blocked storage). Keep this in mind if you add new keys; wrapping them in `try`/`catch` is the safe pattern.

## Audio

Audio is synthesized with the Web Audio API. `ensureAudio()` lazily creates the `AudioContext` on the first user gesture, which browsers require. Sound effects are in the `sfx` object (around line 624) and built from `beep()` and `noiseBurst()`. Do not add audio files; keep the project asset-free.

## Input

- **Keyboard:** arrows/WASD to move, Space to fire, `X` or Shift to bomb, `P` or Esc to pause.
- **Mouse:** move to steer, hold to fire, click the canvas.
- **Touch:** left-side virtual joystick, FIRE and BOMB buttons. Shown only when a touch device is detected.

## Accessibility and performance

- `announce()` writes to a polite `aria-live` region (`#srLive`) for screen-reader updates.
- A reduced-motion toggle (`#motionBtn`) disables screen shake, flashes, and the vignette pulse. It also defaults from `prefers-reduced-motion`.
- `trackPerf()` adapts a `quality` level (0–2) from frame times. Keep new effects cheap, or gate them on `quality`.

## Conventions when editing

- Keep everything in `index.html`. Do not split out modules, add a build step, or add dependencies unless the user asks for it.
- Keep the code inside the IIFE and `'use strict'`. Do not add globals.
- Use `const` and `let`, matching the existing style. Use 2-space indentation.
- Keep changes small and local. Gameplay values belong in the constants block, not scattered through the logic.
- Preserve mobile behavior: the layout must work at phone width with `touch-action: none` on the canvas and controls.
- Do not add external CDN links, web fonts, or tracking. The game is intentionally self-contained.

## Git

- Development branch for this work: `claude/gifted-tesla-kjq7vk`. Push there only.
- Do not open pull requests unless the user asks.
