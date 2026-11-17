# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs as a single-page HTML5 canvas game. The entire game (markup, CSS, and JavaScript) lives in `index.html` (~1,670 lines). There is no build step, no package manager, and no test suite.

`README.md` contains only a title and a stray timestamp; it is not a source of project documentation.

## Repository layout

| Path | Purpose |
| --- | --- |
| `index.html` | The whole game: `<style>` block (~lines 10–361), HUD/overlay/touch-control markup (~lines 362–421), and one IIFE `<script>` (starts ~line 422). |
| `README.md` | Placeholder. |

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install step is needed.

## Code structure (inside `index.html`)

The script is one `(function () { 'use strict'; ... })()` closure. Rough order:

- **DOM refs**: `document.getElementById(...)` lookups at the top.
- **Persistence keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion` (localStorage).
- **Tuning tables**: `DIFFICULTIES`, `THEMES` (per-stage palettes), `ENEMY_TYPES` (grunt/interceptor/tank, with `minStage`), `BOSSES` (six bosses, `hpMul` scales health), `KILLS_PER_STAGE` (18), `BASE_W × BASE_H` = 480 × 720 logical resolution.
- **Audio**: WebAudio synthesized `beep()` / `noiseBurst()` effects; no audio files. `ensureAudio()` must run from a user gesture.
- **HUD and FX**: hearts/bombs rendering, combo pops, stage banner, screen shake, hit/bomb flash, vignette.
- **Input**: keyboard (`keys[e.code]`), a virtual joystick + fire/bomb buttons for touch, and mouse on the canvas.
- **Game state**: `resetGame()`, `update(dt)`, the draw functions (`drawPlane`, `drawBossShip`), and spawners (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`).
- **Weapons and bosses**: `fireWeapon()` (Vulcan/Laser, 3 levels each), `useBomb()`, and boss attack patterns in `BOSS_PATTERNS` (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireDoubleRing`, etc.), driven by `updateBoss(dt)`.
- **Lifecycle**: `loseLife()`, `killBoss()`, `endGame()` (writes best score to localStorage).

## Conventions

- **Keep it a single file.** Do not split out modules, a bundler, or a framework unless the user asks. New code goes into the existing `<script>` IIFE and `<style>` block.
- **Vanilla JS only.** No external libraries or CDN links are currently used; avoid adding any without a reason.
- **Use the existing constants.** Add enemy, boss, or difficulty variants to the tuning tables rather than hardcoding numbers in update/draw logic.
- **Logical coordinates.** Game logic works in the 480 × 720 base space; the canvas is scaled by `resize()`. Don't hardcode screen pixels.
- **Time-based updates.** `update(dt)` takes a delta time; keep new movement and timers frame-rate independent.
- **Respect reduced motion.** Screen shake and flashes are gated by the `reducedMotion` setting; new visual effects that move the screen should honor it too.
- **Mobile first.** The page sets `touch-action: none` and `user-select: none`, and has touch controls. Test any input change for both keyboard and touch.
- **Accessibility hooks.** Announcements go through `announce()` into the `srLive` live region. Use it for important state changes (stage start, game over, etc.).

## Making changes

- There are no tests or linters. Verify changes by loading `index.html` in a browser and playing through the affected path (start, fire, bomb, a stage transition or boss, game over, mute, difficulty switch).
- Check the browser console for errors after edits.
- Keep the file valid HTML; a stray `</script>` or unbalanced brace will blank the whole game.
- When adjusting balance, change values in the tuning tables first.

## Git

- Development branch: `claude/gifted-tesla-b8i00j`. Commit and push there.
- Commit messages use a short imperative subject line, with a body for non-trivial changes.
- Do not open pull requests unless explicitly asked.
