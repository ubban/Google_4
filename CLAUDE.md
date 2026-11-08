# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser.

- `index.html` — the entire game: markup, CSS, and JavaScript in one self-contained file (~1,670 lines).
- `README.md` — a minimal placeholder.

There is no build step, no package manager, no bundler, and no test suite. Open `index.html` directly in a browser (or serve the folder with any static server) to play.

## Repository layout

```
index.html   # Full game (inline <style> and a single IIFE <script>)
README.md    # Placeholder
```

Inside `index.html`:

- `<style>` (~lines 10–420): layout, HUD, overlays, touch controls, reduced-motion overrides.
- Markup (~lines 360–420): HUD (`#hud`), canvas (`#gameCanvas`), start/pause/game-over overlay, touch controls, controls hint.
- `<script>` (~line 422 onward): one IIFE containing all game logic.

## Architecture of the game script

Everything lives in a single `(function () { ... })();` closure. Key sections, roughly in order:

1. **DOM references** — `getElementById` lookups for every HUD/overlay element.
2. **Constants** —
   - `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, enemy fire chance, lives, starting bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18) — kills needed before a stage boss warning.
   - `THEMES` — six sky palettes, cycled by stage.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP). `minStage` gates when each appears.
   - `BOSSES` — six boss designs, selected by `(stage - 1) % BOSSES.length`.
   - `BASE_W = 480`, `BASE_H = 720` — logical canvas size; `resize()` scales it to the window (capped at 1.6×).
3. **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep`/`noiseBurst` primitives and an `sfx` map. Audio is created only after a user gesture.
4. **Input** — keyboard (`keys[e.code]`), mouse steering and click-to-fire on the canvas, and touch (virtual joystick in `#moveZone`, `#fireBtn`, `#bombBtn`).
5. **Game state** — module-level `let`s (`score`, `lives`, `bombCount`, `stage`, `stageKills`, `player`, `enemies`, `bullets`, `powerUps`, `boss`, etc.). `resetGame()` initializes them from the current difficulty.
6. **Update / draw** — `update(dt)` advances the simulation; `draw()` renders. Both are driven by a single `requestAnimationFrame` `loop(now)`, which clamps `dt` to 0.05 s and only updates/draws while `running && !paused`.
7. **Power-ups** — `maybeSpawnPowerUp` drops `life` (6%), `bomb` (8%), or `weapon` (20%) pickups on enemy death. Weapon pickups raise `player.level` up to 3; past level 3 they toggle between `vulcan` and `laser` and reset the level to 1.
8. **Bombs** — `useBomb()` consumes a bomb, clears enemy bullets, and damages everything on screen.

### Conventions when editing the game

- Keep everything inside the existing IIFE. Do not introduce globals.
- Add new tunable values to the existing constant objects (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`) rather than hard-coding numbers in logic.
- Persistent state goes through the `STORAGE_*` keys and must be wrapped so a blocked `localStorage` does not break the game. Note that the current reads and writes are unguarded, so this is a known gap to fix when touching that code.
- Respect `reducedMotionPreferred` / `body.reduced-motion` for any new screen shake, flashing, or animation.
- Keep the HUD and overlays working at phone width; the layout uses `clamp()` and `vmin`/`vw` units and must not introduce horizontal page scroll (`html, body` are `overflow: hidden`).
- Announce important state changes through `announce()` (the `#srLive` live region) so screen-reader users get them.
- Keep the file self-contained. Do not add external scripts, stylesheets, or fonts unless there is a clear need.

## Running and verifying changes

- **Run:** open `index.html` in a browser, or serve the directory, e.g. `python3 -m http.server 8000` and visit `http://localhost:8000/`.
- **Test:** there is no automated test suite. Verify changes by playing the game in a browser, checking all three difficulties, a stage transition (18 kills), a boss fight, bomb use, and the mobile/touch layout if the change touches input or layout.
- **Lint/format:** none configured. Keep the existing code style: 2-space indentation, single quotes in JavaScript, and `const`/`let` declarations.

## Git workflow

- Develop on the branch specified by the current task. Do not push to other branches without explicit permission.
- Write clear, descriptive commit messages that explain the gameplay or code change.
- Do not open a pull request unless explicitly asked.

## Keeping this file current

Update this CLAUDE.md when the structure changes in a way that matters to future work — for example, when the game is split into multiple files, a build step or test framework is added, or the constants and mechanics documented above change meaningfully. Keep line-number references approximate, or remove them if they go stale.
