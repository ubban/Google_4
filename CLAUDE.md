# CLAUDE.md

Guidance for Claude Code working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-page HTML5 canvas game. The whole game lives in `index.html`: markup, CSS,
and one inline `<script>` IIFE. There is no build step, no bundler, no package
manager, and no test suite.

Files:

- `index.html` — the entire game (about 1,670 lines)
- `README.md` — title only
- `CLAUDE.md` — this file

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static
server (for example `python3 -m http.server`). No install step is required.

Nothing is compiled or linted in this repo, so verify changes by loading the page
and playing. Check the browser console for errors after any change.

## Controls

| Action         | Keyboard                  | Touch                         |
| -------------- | ------------------------- | ----------------------------- |
| Move           | Arrow keys / WASD         | Virtual joystick (left zone)  |
| Fire           | Space (hold)              | Fire button                   |
| Smart bomb     | X or Shift                | Bomb button                   |
| Pause / resume | P or Escape               | Pause button                  |
| Mouse aim      | Mouse move (desktop)      | —                             |

Touch controls are shown only when `ontouchstart` or `navigator.maxTouchPoints`
indicates a touch device (`isTouchDevice`).

## Architecture of `index.html`

Everything runs inside one IIFE. Key pieces, in source order:

1. **DOM references** — `document.getElementById` lookups at the top of the
   script. Add new HUD or overlay elements here.
2. **Constants** — `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`,
   `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, and `BOSSES`.
3. **Audio** — a lazily created `AudioContext` (`ensureAudio`) plus synthesized
   sound helpers (`beep`, `noiseBurst`) and a named `sfx` table. No audio files.
4. **HUD and feedback** — `renderHearts`, `renderBombs`, `flashHit`,
   `showComboPop`, `showStageBanner`, `screenShake`, and an `announce` live region
   for screen readers.
5. **Layout scaling** — the logical play field is `BASE_W` × `BASE_H`
   (480 × 720). `resize()` scales the canvas to fit the window, capped at 1.6×.
6. **Input** — `keys` map keyed by `e.code`, touch joystick handling, and mouse
   targeting via `mouseTarget`.
7. **Game state** — `resetGame()` initializes `player`, `enemies`, `bullets`,
   `enemyBullets`, `powerUps`, bombs, lives, stage, and score.
8. **Entities** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, and drawing
   helpers `drawPlane` and `drawBossShip`.
9. **Simulation** — `update(dt)` advances everything by a clamped delta time.
   `fireWeapon`, `useBomb`, `addScore`, and `breakChain` (kill-chain multiplier)
   live here.
10. **Rendering** — `draw()` paints the scene. It is skipped while paused.
11. **Loop** — `loop(now)` runs on `requestAnimationFrame`. `dt` is capped at
    0.05 s so tab switches do not teleport entities.

### Game systems to know about

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal`, `hard`. Each sets spawn
  cadence, enemy speed, enemy fire chance, starting lives and bombs, and a boss
  HP multiplier. Selected via `setDifficulty` and persisted.
- **Stages**: each stage ends after `KILLS_PER_STAGE` kills and spawns a boss.
  Stage number selects a boss design (`BOSSES[(stage - 1) % BOSSES.length]`) and
  a theme (`THEMES`). Boss HP scales with stage and difficulty.
- **Enemy types**: `grunt`, `interceptor` (fast, weaving), and `tank` (high HP).
  Each has a `minStage` gate that controls when it can spawn.
- **Weapons**: `vulcan` and `laser`, each with power levels 1–3 that change the
  spread (`fireWeapon`). Fire delay is 0.1 s for laser and 0.15 s otherwise.
  Weapon power-ups raise the level up to 3; at level 3 the next pickup switches
  to the other weapon and resets the level to 1.
- **Power-ups**: weapon, bomb, and extra life, dropped by `maybeSpawnPowerUp`.
- **Adaptive quality**: `trackPerf` watches a 45-frame rolling average and steps
  `quality` between 0 (full), 1 (medium), and 2 (low). Quality controls shadow
  blur, particle counts, and glow. Changing the thresholds affects visuals on slow
  devices, so test on one.
- **Reduced motion**: honors `prefers-reduced-motion` on first load and can be
  toggled with the motion button. It scales screen shake down (to 15% magnitude,
  capped at 0.1 s) and disables hit and bomb flashes.

## Persistence

All persistence uses `localStorage` with these keys. Keep the names stable, since
changing a key silently resets a player's saved state.

| Key                        | Value                         | Purpose                  |
| -------------------------- | ----------------------------- | ------------------------ |
| `skyfighter.bestScore`     | number                        | High score               |
| `skyfighter.muted`         | `'1'` or `'0'`                | Audio mute               |
| `skyfighter.difficulty`    | `easy` / `normal` / `hard`    | Selected difficulty      |
| `skyfighter.reducedMotion` | `'1'` or `'0'`                | Reduced-motion override  |

Every `localStorage` access is currently unguarded. If you add new reads or writes,
consider wrapping them in `try/catch`, because storage can throw in private
browsing or when site data is blocked.

## Conventions

- **Keep it a single file.** Do not split the game into modules or add a build
  step unless the user asks. Everything the page needs should stay in
  `index.html`.
- **Match the existing style.** The code uses 2-space indentation, single quotes
  in JavaScript, `const` by default, and short section banners such as
  `// ---------- Input ----------`. Helper functions are plain `function`
  declarations inside the IIFE.
- **Use the constants tables.** Add new enemies, bosses, themes, and difficulty
  settings to the existing tables instead of hard-coding values in logic.
- **Keep the simulation frame-rate independent.** Scale movement and timers by
  `dt`, not by frame count.
- **Respect the quality levels.** Any new effect that is expensive should check
  `quality` and reduce or skip itself at levels 1 and 2.
- **Respect reduced motion.** New shakes, flashes, or pulsing effects should
  check the reduced-motion flag.
- **Accessibility.** Announce important state changes through `announce()` so the
  `srLive` region reaches screen readers.

## Gotchas

- `Space` calls `preventDefault` in the keydown handler to stop page scrolling.
  Keep that behavior if you add keys.
- `useBomb` returns early unless the game is running, unpaused, not over, and has
  bombs left. Bomb-related UI should respect the same guard.
- `togglePause` is a no-op during game over. Pause is also how "Resume" works
  from the start overlay.
- Audio starts only after a user gesture. `ensureAudio()` is called from key
  handlers and the start button, so any new sound must go through `sfx` or call
  `ensureAudio()` first.
- The best score is written to `localStorage` only when a new best is set at game over.

## Verifying changes

There are no automated tests or linters. For any change:

1. Load `index.html` in a browser and check the console for errors.
2. Play through at least one full stage to a boss, and confirm HUD updates.
3. Check difficulty switching, pause and resume, and bomb use.
4. If you touch rendering, check at least two quality levels.
5. If you touch touch controls, test with device emulation.

## Git

- Default branch: `main`.
- Commit messages are imperative and descriptive: a short subject line, then a
  bulleted body explaining the change.
