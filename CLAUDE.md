# CLAUDE.md

Guidance for working on this repository.

## Project overview

Google_4 is a single-page HTML5 canvas game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. The entire game (markup, CSS, and JavaScript) lives in one file, `index.html`. `README.md` is a placeholder and has no project documentation.

There is no build step, package manager, bundler, test suite, or linter. Nothing needs to be installed.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static file server (for example `python3 -m http.server`).
- There are no automated tests. Verify changes by playing the game in a browser. For touch behaviour, use a touch device or the browser's device emulation.

## File layout (`index.html`)

- **Lines ~1–360, `<style>`**: all CSS. The HUD uses pill-style elements, the title and pause screens use a glassmorphic overlay card, and there are keyframe animations for banners, hit flashes, and the low-health vignette. Reduced-motion rules are under `body.reduced-motion`.
- **Lines ~360–420, `<body>`**: HUD elements, overlay, touch controls, and the `#srLive` aria-live region used for screen-reader announcements.
- **Lines ~422–1668, `<script>`**: one IIFE wrapping the game. Sections are marked with `// ---------- Name ----------` comments:
  - *Setup and DOM refs*: cached element lookups and `STORAGE_*` keys.
  - *Adaptive performance*: `trackPerf` drops quality (`quality` 0–2) when frame time is too high.
  - *Config tables*: `DIFFICULTIES`, `THEMES` (one per stage, cycled), `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`.
  - *Audio*: a lazily created `AudioContext` (`ensureAudio`) and a `sfx` object built from `beep` and `noiseBurst`.
  - *Canvas sizing*: `BASE_W × BASE_H` is 480×720, scaled to fit the window (max 1.6×).
  - *Input*: keyboard listeners, mouse/pointer steering, and a touch joystick plus FIRE and BOMB buttons.
  - *Game state and logic*: `resetGame`, `spawnEnemy`, `spawnBoss`, `update(dt)`, power-ups (`applyPowerUp`), bombs, the kill-chain multiplier (`addScore`, `breakChain`), and lives (`loseLife`, `endGame`).
  - *Boss patterns*: `BOSS_PATTERNS` holds one list of bullet-pattern functions (radial, aimed spread, columns, spiral, cross, homing, double ring) per boss design. Each boss cycles through its own list as the fight goes on. The boss for a stage is `BOSSES[(stage - 1) % BOSSES.length]`.
  - *Rendering*: `drawPlane`, `drawBossShip`, `drawClouds`, `draw()`.
  - *Main loop*: `loop(now)` clamps `dt` to 50 ms, runs `update` and `draw` only while running and not paused, then calls `requestAnimationFrame`.

## Game rules to keep in mind

- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- Stages advance every `KILLS_PER_STAGE` (18) kills. Each stage ends in a boss from `BOSSES`, and the stage index selects a theme.
- Enemy types are gated by `minStage` (tanks from stage 2).
- Controls: mouse or touch to steer, Space or click to fire, X / Shift for a bomb, P / Esc to pause. Touch devices enable the on-screen controls.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

| Key | Holds |
|---|---|
| `skyfighter.bestScore` | high score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | last difficulty name |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; if unset, follows `prefers-reduced-motion` |

Changing a key's name or format will reset existing players' saved state, so treat these as a public interface.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. Do not add a build system or dependencies without a request to do so.
- Match the existing style: 2-space indentation, single quotes in JavaScript, `const` by default, arrow functions for callbacks, and short section banner comments.
- Gameplay tuning lives in the config tables at the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`). Prefer editing those over adding magic numbers in logic.
- Audio and motion effects must respect the mute toggle and reduced-motion setting.
- Keep accessibility hooks working: announcements go through `announce()` into `#srLive`, and buttons need labels.
- Keep the layout working on phones, with the canvas scaled and touch controls shown on touch devices.

## Git

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-6fiaa9`). Do not push to `main` directly.
- Write descriptive commit messages in the imperative mood, as in the existing history.
