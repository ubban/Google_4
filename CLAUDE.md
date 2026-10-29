# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up rendered on a single `<canvas>`. The whole game is one self-contained file.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, CSS (in `<style>`), and JavaScript (in one IIFE `<script>`). |
| `README.md` | Minimal project readme. |

There is no build step, no package manager, no bundler, no test suite, and no CI configuration. To run the game, open `index.html` in a browser (or serve the directory with any static file server).

## Editing guidelines

- **Keep it a single file.** Add new CSS to the `<style>` block and new JS inside the existing IIFE. Do not introduce external dependencies, CDN scripts, or a build step unless the user asks for it.
- **Stay in the IIFE.** All game state is local to `(function () { 'use strict'; ... })()`. Don't leak globals; only `localStorage` keys (below) persist across sessions.
- **Match the existing style.** Two-space indentation, single quotes in JS, `const`/`let` (no `var`), arrow functions for callbacks, and short section banners in the form `// ---------- Section name ----------`.
- **Frame-rate independence.** `update(dt)` takes seconds. `loop()` clamps `dt` to `0.05`, so long pauses in the tab don't cause huge jumps. Speeds and timers should be expressed per second and multiplied by `dt`.
- **Rendering is immediate-mode.** `draw()` redraws the full canvas each frame. Don't cache per-frame objects in the DOM.
- **No automated tests exist.** Verify changes by loading the game in a browser and playing through the affected path (menu → start → gameplay → game over). For visual or audio changes, check both desktop and a narrow/touch viewport.

## Architecture of `index.html`

Major sections, in file order (search for the `// ----------` banners):

1. **Setup and DOM refs**: canvas, HUD elements, overlay, touch controls, and `STORAGE_*` keys.
2. **Adaptive performance** (`trackPerf`): a rolling 45-frame average adjusts a `quality` level (0 full, 1 medium, 2 low). Quality gates shadow blur, glow, and particle counts. Lower it rather than removing effects when you add new expensive visuals.
3. **Config tables**:
   - `DIFFICULTIES`: `easy` / `normal` / `hard`, controlling spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
   - `THEMES`: background palettes, selected by `(stage - 1) % THEMES.length`.
   - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (4 HP, minimum stage 2).
   - `BOSSES`: six bosses, selected by `(stage - 1) % BOSSES.length`. Each has a matching entry in `BOSS_PATTERNS`.
   - `KILLS_PER_STAGE = 18`: kills needed to trigger a boss warning and fight.
4. **Audio**: `ensureAudio()` lazily creates the `AudioContext`. `sfx.*` helpers (`beep`, `noiseBurst`, named sounds) synthesize everything. There are no audio files. Audio is created on the first user gesture, so keep `ensureAudio()` on input handlers that play sounds.
5. **Canvas sizing** (`resize`): the logical resolution is fixed at 480×720 and scaled to fit the window, capped at 1.6×.
6. **Input**: keyboard, mouse, and touch handlers (see the controls table below).
7. **Game state and reset** (`resetGame`): one `player` object plus arrays for `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, and a `boss`. Lives and bombs come from the difficulty.
8. **Gameplay**:
   - `update(dt)` advances the simulation: movement, firing, spawning, collisions, chain/multiplier, and stage progression.
   - `applyPowerUp` handles `life`, `bomb`, and `weapon` pickups. Weapons upgrade in levels 1–3, then toggle between `vulcan` and `laser`.
   - `useBomb` clears enemy bullets, destroys every enemy on screen (+5 score each), and deals 24 damage to the boss.
9. **Boss patterns**: `BOSS_PATTERNS` is an array of bullet-pattern functions (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, and others). `updateBoss` selects and runs them.
10. **Flow**: `loseLife`, `endGame`, `togglePause`, `killBoss`.
11. **Rendering**: `draw()` renders background (themed sky with clouds), entities, particles, HUD effects, and screen shake. `drawPlane` and `drawBossShip` build the ship sprites from paths.
12. **Main loop**: `loop(now)` runs `trackPerf`, then `update` and `draw` if the game is running and not paused, then re-queues via `requestAnimationFrame`.

## Controls

| Input | Action |
| --- | --- |
| Mouse move | Steer the ship |
| Mouse click | Fire |
| Arrow keys / WASD | Move |
| Space | Fire |
| X or Shift | Bomb |
| P or Esc | Pause |
| Touch drag (left side) | Virtual joystick |
| FIRE / BOMB buttons | Fire / bomb on touch devices |

Keep the on-screen hint (`#controlsHint` in the overlay) in sync if you change any binding.

## Persistence

The game stores these keys in `localStorage`. Don't rename them without a migration, since players' saved data depends on them:

- `skyfighter.bestScore`: the best score.
- `skyfighter.muted`: sound mute flag (`'1'` = muted).
- `skyfighter.difficulty`: last selected difficulty.
- `skyfighter.reducedMotion`: reduced-motion toggle. If unset, it falls back to the `prefers-reduced-motion` media query.

Reads and writes to `localStorage` can throw in private browsing or when storage is blocked. If you add new storage, wrap it in `try`/`catch`, or follow the existing pattern and accept that the game may not persist settings.

## Accessibility

- Screen-reader announcements go through `announce(text)`, which writes to the `#srLive` live region. Use it for important state changes (stage start, game over, new best).
- Reduced motion (`reduced-motion` body class) disables screen shake, flash, and pulse effects. New motion effects should respect `reducedMotion`.

## Working in this repo

- Default branch is `main`. Feature work for this session goes on `claude/gifted-tesla-6jktfd`.
- Commit messages should be short, imperative, and describe the gameplay or code change (see the existing `Add Sky Fighter: ...` commit for style).
- Keep changes focused. Because everything lives in one file, unrelated edits create noisy diffs. Avoid reformatting untouched code.
