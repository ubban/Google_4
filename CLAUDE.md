# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser on a `<canvas>`.

| File | Purpose |
| --- | --- |
| `index.html` | The whole game: markup, CSS (`<style>`, roughly lines 10–361), and JavaScript (`<script>`, starting around line 422). |
| `README.md` | Minimal project readme. |

There is no build step, package manager, bundler, test suite, or linter. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

Verify changes by loading the page and playing: start a run, fire, use a bomb, pause, and check the game-over screen. There are no automated tests to run.

## Code layout inside `index.html`

- **Single IIFE.** All game code is wrapped in `(function () { 'use strict'; ... })();`. Nothing is exposed on `window`; keep it that way unless there is a reason.
- **DOM references** are grabbed once at the top of the script by `getElementById` / `querySelector`. Add new elements there, not inline.
- **Constants** are `UPPER_SNAKE_CASE` objects/arrays near the top: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`, `KILLS_PER_STAGE`, `BASE_W`/`BASE_H` (480×720 logical play field).
- **Persistence** uses `localStorage` with keys prefixed `skyfighter.` (`STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`). Reads and writes are not wrapped in `try/catch` today; if you add storage access, guard it, because private windows and blocked storage can throw.
- **Game loop.** `loop(now)` clamps `dt` to 50 ms, calls `trackPerf(dt)`, then `update(dt)` and `draw()` while `running && !paused`. Logic belongs in `update*`, rendering in `draw*`/`draw*` helpers.
- **Adaptive quality.** `quality` (0 full, 1 medium, 2 low) is driven by a rolling 45-frame average in `trackPerf`. Respect it when adding expensive effects.
- **Audio** is synthesized with the Web Audio API (`beep`, `noiseBurst`, the `sfx` object). `ensureAudio()` must run from a user gesture before any sound plays. Honor the `muted` flag.
- **Accessibility.** `announce()` writes to the `srLive` ARIA live region. `reducedMotion` (user setting, defaulting to `prefers-reduced-motion`) should gate screen shake, flashes, and similar effects.
- **Input.** Keyboard (`keys[e.code]`), mouse on the canvas, and touch controls (`moveZone` joystick, `fireBtn`, `bombBtn`) are all supported. Changes to controls must keep all three working.

## Game mechanics reference

- **Difficulty** (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- **Enemies:** `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2).
- **Stages:** `KILLS_PER_STAGE` kills advance the stage. Stages cycle through `THEMES` (background palette) and `BOSSES`, each with a unique attack pattern in `BOSS_PATTERNS`.
- **Weapons:** `vulcan` and `laser`, toggled by a weapon power-up. `player.level` upgrades the weapon. Fire delay is 0.15 s for vulcan and 0.1 s for laser.
- **Scoring:** points come from `addScore`, with a combo multiplier that `breakChain()` resets.
- **Bombs** are cleared with `useBomb()`. Lives are lost via `loseLife()`, and `endGame()` shows the overlay and saves the best score.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Do not introduce a framework or external dependencies. The page must work offline from a plain file open.
- Match the existing style: 2-space indentation, single quotes in JS, `const` by default, small named helper functions.
- Keep the game playable on phones: preserve the `viewport` meta, the touch controls, and the scaling in `resize()`.
- Do not add new `localStorage` keys without a `skyfighter.` prefix and a matching `STORAGE_*` constant.
- When changing balance numbers, change the constants rather than hard-coding values in the logic.

## Git workflow

- Develop on the branch assigned to the session (for example `claude/gifted-tesla-u0d3cm`). Do not push to `main` directly.
- Commit only when asked, and write clear, imperative commit messages.
- Do not create a pull request unless the user explicitly asks for one.

## Things to avoid

- Do not commit generated files, local server logs, or `node_modules`. None should exist here; add a `.gitignore` only if tooling is introduced.
- Do not rewrite `README.md` beyond what is asked. Its current contents are a placeholder.
