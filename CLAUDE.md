# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a single-file HTML5 canvas game.

| Path        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html` | The entire game: markup, CSS (`<style>`), and JavaScript (`<script>`). ~1,670 lines. |
| `README.md`  | Placeholder. Currently just the repo name and a stray timestamp.        |

There is no `package.json`, bundler, framework, test suite, linter, or CI config. Do not invent build or test commands. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository state

- Development branch for this work: `claude/gifted-tesla-oyd1k2`.
- History: `Initial commit`, then `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.

## Architecture of `index.html`

Everything lives in one IIFE (`(function () { 'use strict'; ... })();`) inside a single `<script>` tag. Nothing is exported or attached to `window`. Keep it that way unless the user asks for a split into modules.

Major sections, in file order:

1. **DOM references**: `const xyzEl = document.getElementById(...)`. HUD, overlay, touch controls, and boss bar elements are all grabbed up front. If you add a new UI element, add its lookup here.
2. **Storage keys**: `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`. All persistence uses `localStorage` under the `skyfighter.*` namespace. Reuse this pattern for new settings.
3. **Adaptive performance**: `quality` (0 full, 1 medium, 2 low) is driven by `trackPerf(dt)` over a 45-frame rolling window. Use `quality` to scale particle counts, clouds, and similar effects rather than hard-coding them.
4. **Tuning tables** (edit these first when balancing):
   - `DIFFICULTIES`: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.
   - `KILLS_PER_STAGE` (currently 18): kills needed to advance a stage.
   - `THEMES`: background palettes cycled per stage.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP). `minStage` gates when each type appears.
   - `BOSSES`: six bosses, cycled by stage, each with `hpMul` and colors.
   - `BOSS_PATTERNS`: bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
5. **Audio**: Web Audio helpers `ensureAudio`, `beep`, `noiseBurst`, and a named `sfx` set. Audio is created lazily on the first user gesture (`ensureAudio()` is called from input handlers). Respect the `muted` flag.
6. **Input**: keyboard (`keys[e.code]`), mouse on the canvas, and a touch joystick (`moveZone`) plus `fireBtn` / `bombBtn`. `firing` and `mouseActive` are the shared input flags.
7. **Game state and loop**: `resetGame()`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`. `update` is frame-rate independent via `dt`. Pause is `togglePause()`.
8. **Lifecycle**: `loseLife()`, `endGame()`, `killBoss()`, and `applyPowerUp(type)`.
9. **Start**: the `startBtn` click handler calls `ensureAudio()` then `resetGame()`. The game loop is kicked off by a single `requestAnimationFrame(loop)` at the bottom.

### Conventions to preserve

- **Canvas coordinates** use a fixed logical size, `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the viewport. Write game logic in base units, not CSS pixels.
- **Accessibility**: announce state changes through `announce(text)`, which writes to the `srLive` live region. Honor `reducedMotion` (toggled via `motionBtn` and the `reduced-motion` body class) for screen shake, flashes, and similar effects.
- **Scoring**: use `addScore(base)` so the kill-chain multiplier (`breakChain()` resets it) stays consistent.
- **Bosses**: once `stageKills` reaches `KILLS_PER_STAGE`, the boss is scheduled (`bossWarned` guards against repeats) and spawns after a 1.2 s delay via `spawnBoss()`. Boss HP bar is `bossBarWrap` / `bossBarFill`.
- **Bombs**: `useBomb()` clears enemy bullets and enemies, and deals 24 damage to the boss. Bomb count comes from difficulty.
- **Power-ups**: spawned via `maybeSpawnPowerUp`, applied via `applyPowerUp`. Two weapons (`vulcan`, `laser`) with upgrade levels shown in the HUD (`weaponLvl`).
- **Mobile first**: touch controls appear for touch devices. Any new control needs a touch path, and `preventDefault` with `{ passive: false }` is used on touch handlers to stop scroll.
- **No dependencies**: do not add external libraries or CDN scripts without an explicit request.

## Working in this repo

- **Keep changes minimal and local.** The file is one unit. Edit the specific function or table involved rather than restructuring the IIFE.
- **Verify by running the game**, not just by reading it. There are no automated tests, so for gameplay changes open `index.html` in a browser and check the affected path (start, fire, bomb, boss, pause, game over, mute, difficulty switch). If a headless browser is needed, Playwright with the preinstalled Chromium works.
- **Check the console** for JavaScript errors after any edit. A single syntax error breaks the whole game because everything shares one IIFE.
- **Test on a narrow viewport** (phone width) for layout or touch changes, since the HUD and touch controls are responsive.
- **Preserve localStorage keys.** Renaming a `skyfighter.*` key silently resets player data (best score, mute, difficulty, motion preference).
- **Commit style**: short imperative subject. Prior commit subjects are descriptive, e.g. `Add Sky Fighter: ...`.

## Things not to do

- Do not add a build step, package manager, or framework unless the user asks.
- Do not split `index.html` into multiple files without asking. Single-file delivery is the current design.
- Do not commit or push unless asked. Do not open pull requests unless explicitly requested.
- Do not add dependencies from unpinned CDN URLs.

## Keeping this file current

Update this CLAUDE.md when any of the following changes: the file layout, a new top-level section in `index.html`, a new build or test command, a new storage key, or a new convention. Keep it factual. Describe what the code does now, not what it was planned to do.
