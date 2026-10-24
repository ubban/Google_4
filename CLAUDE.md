# CLAUDE.md

Guidance for working in this repository.

## Project overview

Google_4 contains **Sky Fighter**, a Raiden-style vertical shoot-em-up built as a
single-page HTML5 canvas app. The entire game lives in `index.html`: markup,
CSS, and one inline `<script>` block wrapped in an IIFE.

Files:

- `index.html` — the whole game (~58 KB).
- `README.md` — one-line placeholder.
- `CLAUDE.md` — this file.

There is no build step, package manifest, bundler, linter, or test suite.
To run the game, open `index.html` in a browser. No dependencies are loaded
from a CDN; keep it that way unless the user asks otherwise.

## Layout of `index.html`

- **`<style>`** (approx. lines 10–359): all styling, including the pill HUD,
  glassmorphic overlay card, touch controls, and `.reduced-motion` overrides.
- **`<body>`** (approx. lines 361–421): `#container` holding the `#gameCanvas`,
  HUD (`#hud`, `#bossBarWrap`, `#comboPop`, `#stageBanner`), touch controls
  (`#touchControls`), and the start/pause overlay (`#overlay`).
- **`<script>`** (approx. lines 422–1667): a single IIFE with `'use strict'`.
  Sections, in order:
  1. DOM references and `STORAGE_*` keys.
  2. Adaptive performance (`trackPerf` drops render quality when frames are slow).
  3. Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
  4. Audio (`ensureAudio`, `beep`, `noiseBurst`, `sfx.*`), created lazily on first user gesture.
  5. HUD helpers (hearts, bombs, combo pop, stage banner, screen shake, mute/motion buttons).
  6. Input: keyboard (`keys` map), pointer/canvas, and the touch joystick and buttons.
  7. Game state: `resetGame`, `update(dt)`, `draw()`, `loop(now)`.
  8. Entities and combat: `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `applyPowerUp`.
  9. Boss attacks: `BOSS_PATTERNS` (radial, spread, columns, spiral, cross, homing, double-ring), driven by `updateBoss`/`killBoss`.
  10. Lifecycle: `loseLife`, `endGame`, `togglePause`, and the `requestAnimationFrame` loop.

## Game mechanics to keep in mind

- Logical playfield is `BASE_W = 480` × `BASE_H = 720`; `resize()` scales the canvas.
- Difficulty (`easy` / `normal` / `hard`) changes spawn rate, enemy speed, enemy fire chance,
  starting lives and bombs, and boss HP multiplier.
- Each stage requires `KILLS_PER_STAGE` (18) kills, then a boss spawns. Stages cycle through
  `BOSSES` and `THEMES`.
- Enemy types: `grunt`, `interceptor` (weaving), and `tank` (4 HP, unlocked at stage 2).
- Weapons have three power levels (Vulcan/Laser). Power-ups: weapon level, bomb, extra life.
- A kill-chain multiplier (`addScore`, `breakChain`) scales score.
- `dt` is clamped to 50 ms in `loop`, so long frames (tab switches) do not teleport entities.

## Persistence

`localStorage` keys, all prefixed `skyfighter.`:

| Key | Meaning |
| --- | --- |
| `skyfighter.bestScore` | High score |
| `skyfighter.muted` | `'1'` when muted |
| `skyfighter.difficulty` | `easy` / `normal` / `hard` |
| `skyfighter.reducedMotion` | `'1'` / `'0'`; defaults to the `prefers-reduced-motion` media query when unset |

Every `localStorage` access is currently unguarded. If you add more, wrap reads and
writes in `try/catch` — storage can throw in private windows or when site data is blocked.

## Conventions

- Keep everything in `index.html` unless the user asks for a split. Do not introduce a
  build system, framework, or external dependency without an explicit request.
- Match the existing style: 2-space indent, single quotes in JS, `const` by default,
  short section comments in the form `// ---------- Name ----------`.
- Tuning values live in the constant tables near the top of the script. Prefer changing
  those over hard-coding numbers inside update/draw logic.
- Preserve accessibility hooks: `#srLive` for screen-reader announcements via `announce()`.
  The `reducedMotion` flag dampens `screenShake()` and disables the hit and bomb flashes
  (`flashHit`, `flashBomb`). Any new screen effect should respect it too.
- Input must keep working on keyboard (arrows/WASD move, `Space` fire, `KeyP`/`Escape` pause,
  `KeyX`/`Shift` bomb), mouse (move to steer, click to fire), and touch (joystick, fire and bomb buttons).
- Audio must only start after a user gesture; go through `ensureAudio()`.

## Verifying changes

No automated tests exist. After editing `index.html`:

1. Open the file in a browser (or serve it with any static server) and check the console for errors.
2. Start a game, then exercise: movement, firing, bombs, pause, a power-up, a stage clear
   (boss fight), losing a life, game over, and the best-score update.
3. Toggle difficulty and mute, reload, and confirm the settings persist.
4. For layout changes, check a narrow (phone-width) viewport and the touch controls.

If you add tests or tooling, document the commands here so future sessions can run them.

## Git

- Development branch for this work: `claude/gifted-tesla-w8mb7w`. Push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
