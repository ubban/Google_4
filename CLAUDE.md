# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a
Raiden-style vertical shoot-'em-up written as a single-page HTML5 canvas app.

- `index.html` — the entire game: markup, CSS (`<style>`, ~lines 10–360), and JavaScript
  (`<script>`, ~lines 422–1667). No external dependencies, no bundler.
- `README.md` — placeholder; contains no real project docs.

There is no `package.json`, build step, test suite, or linter. Open `index.html` in a
browser (or serve the directory with any static server) to run it.

## Repository layout and conventions

- Keep the game in one self-contained `index.html` unless the user asks for a split.
  Do not add a build system or npm dependencies for a change that fits in the file.
- All game code lives in one IIFE inside the `<script>` tag. Helper functions and
  constants are local to it; nothing is exposed on `window`.
- Match the existing style: 2-space indentation, `const`/`let` (no `var`), single quotes
  in JS, arrow functions for callbacks, and short section-level comments only where the
  intent is not obvious from the code.
- Constants are UPPER_SNAKE_CASE objects/arrays (`DIFFICULTIES`, `ENEMY_TYPES`,
  `BOSSES`, `THEMES`). Storage keys are prefixed `skyfighter.` (`STORAGE_*` constants).
- DOM element handles are grabbed once at the top of the script (`const xEl = document.getElementById(...)`).
  Add new handles there rather than querying inside hot paths.

## Game architecture (index.html)

The main loop is `requestAnimationFrame` → `loop(now)`, which clamps `dt` to 0.05s,
calls `trackPerf(dt)`, then `update(dt)` and `draw()` when the game is running and not
paused.

Key systems, roughly in code order:

| Area | Where to look | Notes |
| --- | --- | --- |
| Difficulty | `DIFFICULTIES` | `easy` / `normal` / `hard` control spawn rate, enemy speed, shoot chance, starting lives and bombs, and boss HP multiplier. Selected via `setDifficulty`. |
| Stages | `KILLS_PER_STAGE`, `THEMES`, `showStageBanner` | A stage ends after 18 kills, then a boss spawns. Each stage cycles to the next theme (6 themes). |
| Enemies | `ENEMY_TYPES`, `spawnEnemy` | `grunt`, `interceptor` (weaving, fast), `tank` (4 HP, unlocks stage 2). Patterns are `aimed` or `spread`. |
| Bosses | `BOSSES`, `spawnBoss`, `drawBossShip` | Six designs cycled by stage. HP scales with stage and difficulty. Telegraphs attacks; health bar in `#bossBarWrap`. |
| Weapons | `fireWeapon`, `player.level` | Vulcan and Laser, with power levels 1–3 (spread and lane count grow with level). |
| Bombs | `useBomb`, `maxBombs`, `bombCount` | Cleans the screen and damages bosses (`dmg = 24`). Triggered by `X` / `Shift` or the touch bomb button. |
| Power-ups | `maybeSpawnPowerUp`, power-up pickup block (~line 1305) | Types: `life` (6%), `bomb` (8%), `weapon` (20%). Chance rolls are cumulative. |
| Scoring | `addScore`, `breakChain`, `multiplier`, `chain` | A kill chain raises the multiplier; `chainTimer` breaks the chain when it expires. |
| Rendering | `draw`, `drawPlane`, `drawClouds`, `spawnExplosion` | Canvas 2D. Particle and explosion counts scale with `quality` (0 full, 1 medium, 2 low). |
| Performance | `trackPerf`, `quality` | Averages the last 45 frame times and steps `quality` down when frames run slow, back up when they recover. Keep new effects behind this. |
| Audio | `ensureAudio`, `beep`, `noiseBurst`, `sfx` | Web Audio, created lazily on first user gesture. `beep` and `noiseBurst` return early when `muted`. |
| Input | `keys`, `joystick*`, `mouseTarget`, touch handlers | Keyboard: arrows/WASD move, Space fires, `P`/`Esc` pause, `X`/`Shift` bomb. Mouse/touch aim the ship; mobile has a joystick and fire/bomb buttons. |
| Accessibility | `reducedMotion`, `announce`, `#srLive` | Honors `prefers-reduced-motion` and can be toggled. Use `announce()` for screen-reader status text. |
| Persistence | `STORAGE_*`, `localStorage` | Best score, mute, difficulty, and reduced-motion preference. Access is wrapped in the existing code. Keep it that way: storage can be unavailable in some browsers. |
| Layout | `resize`, `BASE_W` / `BASE_H` (480×720), `scale` | Fixed logical resolution scaled to fit the window (max scale 1.6). |

## Working on this codebase

- **Verify in a browser.** There are no automated tests. For any gameplay or UI change,
  open the page and play through the affected path (start, shoot, pick up a power-up,
  bomb, reach a boss, pause, game over). If you cannot run a browser, say so explicitly
  rather than claiming the change works.
- **Preserve the performance guardrails.** New per-frame work should respect `quality`
  and avoid allocating in hot loops where practical. Particle-heavy effects should scale
  with the quality level.
- **Keep the mobile path working.** Any new control must have a touch equivalent. Check
  that the layout still fits a phone-sized viewport with no horizontal scroll.
- **Keep the HUD and overlay consistent.** Stat elements are pills in `#hud`; overlays use
  the glassmorphic `#overlayCard`. Reuse these classes instead of adding new styling
  patterns.
- **Do not change balance numbers casually.** Difficulty, enemy HP and speed, boss HP
  multipliers, and power-up drop rates are tuned together. Call out the intended effect
  in the commit message when you adjust them.

## Git

- Default development happens on the branch the session was given (e.g.
  `claude/gifted-tesla-wi0tsq`). Do not push to other branches without permission.
- Write commit messages that describe the gameplay or code change, with a short
  body for non-obvious decisions.
- Do not open a pull request unless the user asks for one.
