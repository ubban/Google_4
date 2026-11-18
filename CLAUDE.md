# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only application is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser on an HTML5 `<canvas>`.

- `index.html` — the entire game: markup, inline CSS (`<style>`, ~lines 10–359), and inline JavaScript (`<script>`, ~lines 422–1667), all wrapped in one IIFE with `'use strict'`.
- `README.md` — a stub with only the title. Keep it short; put detailed docs here instead.

There is no build step, no bundler, no `package.json`, no test suite, and no linter configuration. Do not invent npm/build/test commands. To check a change, open `index.html` in a browser (or serve the directory with any static server) and play it.

## Repository layout

```
.
├── index.html   # Sky Fighter (single-file game)
├── README.md    # Title stub
└── CLAUDE.md    # This file
```

## Game architecture (`index.html`)

The script follows a simple fixed-structure game loop:

- **Canvas setup.** The game draws at a logical resolution of `BASE_W = 480` × `BASE_H = 720` and `resize()` scales it to the viewport.
- **Main loop.** `loop(now)` computes `dt` (clamped to 0.05 s), calls `trackPerf(dt)`, and, when `running && !paused`, runs `update(dt)` then `draw()`. It is driven by `requestAnimationFrame`.
- **State.** Game state lives in module-level `let` variables inside the IIFE, reset by `resetGame()`.
- **Update / draw split.** `update(dt)` advances entities, collisions, spawning, and scoring. `draw()` is render-only. Keep logic out of `draw*` helpers and rendering out of `update*`.
- **Entity config tables** near the top of the script are the main tuning surface:
  - `DIFFICULTIES` — per-difficulty `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.
  - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (multi-hit). Each has size, HP, speed multiplier, colors, score, and the `minStage` it first appears in.
  - `BOSSES` — six bosses cycled by stage, each with colors and an `hpMul`.
  - `THEMES` — six sky palettes, one per stage.
  - `BOSS_PATTERNS` — bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring) used by `updateBoss`.
  - `KILLS_PER_STAGE` — kills needed to advance a stage.
- **Audio.** Procedural only, via Web Audio (`ensureAudio`, `beep`, `noiseBurst`, and the `sfx` object). There are no audio asset files. `ensureAudio()` must be called from a user gesture before sound will play.
- **Persistence.** `localStorage` keys, all prefixed `skyfighter.`:
  - `skyfighter.bestScore`
  - `skyfighter.muted` (`'1'`/`'0'`)
  - `skyfighter.difficulty` (`easy` | `normal` | `hard`)
  - `skyfighter.reducedMotion` (`'1'`/`'0'`; falls back to `prefers-reduced-motion` when unset)

  Every `localStorage` access is currently unguarded. If you change this, wrap reads and writes in `try/catch`, since storage can throw in private windows or when site data is blocked.
- **Accessibility.** `announce(text)` writes to the `#srLive` `aria-live` region. Use it for important state changes (stage start, boss, game over), not for every frame.
- **Reduced motion.** When enabled, the `reduced-motion` body class turns off the CSS flashes, pulses, and entrance animations, and `screenShake()` damps shake magnitude and duration. New visual effects that move or flash should respect this flag.

## Controls (keep in sync with `#controlsHint`)

- **Desktop:** mouse to steer, click to fire; arrows/WASD to move; Space to fire; X or Shift for bomb; P or Esc to pause.
- **Touch:** drag on the left zone for a virtual joystick; FIRE and BOMB buttons on the right.

If you add or change a control, update both the handlers and the `#controlsHint` text in the overlay.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks to split it. Do not add a build step or framework without asking.
- **No external dependencies.** The current page loads nothing from the network; the only external reference is the inline SVG favicon. Keep it that way unless there is a clear reason.
- **Style.** Match the existing code: 2-space indentation, single quotes in JS, `const` by default, arrow functions for callbacks, and `camelCase` for functions and variables. Config tables use `UPPER_SNAKE_CASE`.
- **Naming.** Entity and boss names are uppercase display strings (`'VANGUARD'`, `'SPECTER'`). Storage keys use the `skyfighter.` prefix.
- **Mobile first.** Verify layout changes at phone width. The page uses `viewport-fit=cover` and `user-scalable=no`, and touch controls must stay usable.
- **Comments.** Keep them short and only where the reason is not obvious.

## Making changes

1. Read the relevant section of `index.html` before editing. The file is large, so use targeted reads rather than the whole file.
2. Keep changes scoped. Tuning numbers belong in the config tables, not scattered through the logic.
3. Test in a real browser: start a game, check at least one stage transition and a boss fight if you touched those paths, check pause/resume, and confirm the mute and difficulty settings persist after a reload.
4. Check both desktop and phone-sized viewports for any UI change.

## Git

- Development branch for automated work: `claude/gifted-tesla-fy4vh2`.
- Write commit messages in the imperative mood with a short summary line, then a body explaining the why when it is not obvious.
