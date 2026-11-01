# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a single-file, Raiden-style vertical shoot-'em-up
built as an HTML5 canvas game. There is no build step, no package manager, and no
test suite.

- `index.html` (~1,670 lines): the entire game — markup, CSS, and JavaScript, all inline.
- `README.md`: a one-line title only.

To play, open `index.html` in a browser. Nothing needs to be installed or compiled.

## Repository layout

```
.
├── index.html   # The whole game: <style> (lines ~10–362), markup (~363–421), <script> (~422–end)
└── README.md    # Title only
```

Inside `index.html`, the `<script>` is one IIFE `(function () { ... })();` with these
logical sections, roughly in source order:

1. **DOM refs and constants** — element lookups, `STORAGE_*` keys, `DIFFICULTIES`,
   `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
2. **Persistence and settings** — best score, mute, difficulty, and reduced-motion
   are read from and written to `localStorage`.
3. **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep`,
   `noiseBurst`, and a named `sfx` table. Audio is created only after a user gesture.
4. **HUD and feedback** — `renderHearts`, `renderBombs`, combo popups, stage banner,
   screen shake, hit and bomb flashes, and an `aria-live` region (`announce`).
5. **Input** — keyboard (`keys` map), touch joystick and fire/bomb buttons, and
   mouse/pointer targeting (`mouseTarget`).
6. **Entities and spawning** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`,
   `spawnExplosion`.
7. **Simulation** — `update(dt)` advances player, bullets, enemies, bosses, power-ups,
   collisions, and stage progression. `fireWeapon` and `useBomb` live here.
8. **Rendering** — `draw()` and helpers (`drawPlane`, `drawBossShip`, `drawClouds`).
   Rendering is driven by `requestAnimationFrame` via `loop(now)`.
9. **Startup** — the start button handler and the initial `requestAnimationFrame(loop)`.

## Key design points

- **Fixed logical resolution.** `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the
  canvas to fit the window, capped at 1.6×. Game coordinates always use the base size.
- **Delta-time loop.** `dt` is clamped to 0.05 s. Update logic must be frame-rate
  independent (multiply by `dt`).
- **Adaptive quality.** `trackPerf` moves `quality` between 0 (full), 1 (medium), and
  2 (low) based on average frame time. Shadow blur, glow, and particle counts depend
  on it. Keep new expensive effects gated by `quality`.
- **Reduced motion.** Respect the `reducedMotion` flag (user setting, defaulting to the
  `prefers-reduced-motion` media query) for screen shake and similar effects.
- **Difficulty** is data-driven via `DIFFICULTIES` (spawn rates, enemy speed, shot
  chance, starting lives and bombs, boss HP multiplier). Tune balance there rather than
  in the logic.
- **Stages** advance every `KILLS_PER_STAGE` kills. Each stage cycles through a
  `THEMES` entry, and bosses come from `BOSSES[(stage - 1) % BOSSES.length]`.
  Enemy variety unlocks by `minStage`.
- **Weapons:** `vulcan` and `laser`, each with power levels 1–3 (`player.level`).

## Controls (for reference when testing)

| Action        | Keyboard                        | Touch / mouse                              |
|---------------|---------------------------------|--------------------------------------------|
| Move          | Arrow keys / WASD               | Joystick (touch); pointer over canvas (mouse) |
| Fire          | Space (hold)                    | Fire button (hold)                         |
| Bomb          | X or Shift                      | Bomb button                                |
| Pause         | P or Escape                     | Pause button in HUD                        |
| Mute          | —                               | Mute button in HUD                         |

Keep this table in sync with the input handlers in the `<script>` if controls change
(search for `keydown`, `touchstart`, `mousedown`, and the `*Btn` listeners).

## Persistence keys

| Key                       | Purpose                       |
|---------------------------|-------------------------------|
| `skyfighter.bestScore`    | Highest score                 |
| `skyfighter.muted`        | Mute flag (`'1'` / `'0'`)     |
| `skyfighter.difficulty`   | Selected difficulty name      |
| `skyfighter.reducedMotion`| Reduced-motion override       |

Changing a key's name or format will reset existing players' saved data. Treat these
as a public contract.

## Working in this repo

- **Single file.** Keep the game in `index.html` unless the user asks for a split.
  Don't add a build system, bundler, or framework without being asked.
- **No dependencies.** Use only browser-native APIs. Avoid adding CDN scripts.
- **No tests or linter exist.** Verify changes by opening the game in a browser (or a
  headless Chromium) and checking the affected flow: start, play, pause, bomb, boss,
  game over, and restart. Also check narrow/mobile viewports, since the layout and
  touch controls are responsive.
- **Quick syntax check.** After editing, you can extract the `<script>` body and run
  `node --check` on it to catch syntax errors cheaply.
- **Keep it readable.** Match the existing style: 2-space indentation, single quotes in
  JS, `const`/`let`, and small named helper functions.
- **Commits.** Use clear, descriptive messages that state the gameplay or UI change.
  Develop on the feature branch assigned for the session; do not push to other branches.
