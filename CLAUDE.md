# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game (HTML, CSS, and JavaScript) lives in one file, `index.html` (~1,670 lines). There is no framework, bundler, package manager, or build step.

Other files:
- `README.md` — placeholder (title only).
- `.git/` — the repository; the default development branch for this session is `claude/gifted-tesla-blxww4`.

## Running and testing

- **Run:** open `index.html` directly in a browser, or serve the folder with any static server (e.g. `python3 -m http.server`).
- **Build:** none.
- **Tests / lint:** none configured. There is no `package.json`, test suite, or linter. When changing gameplay, verify manually in a browser (keyboard, touch, and pause/mute/difficulty paths).

## File layout inside `index.html`

- `<style>` (lines 1–359): all CSS. Dark color scheme (`:root { color-scheme: dark; }`); HUD, overlays, touch controls, vignette/flash effects.
- `<body>` markup (lines 361–421): HUD elements (`#score`, `#best`, `#mult`, `#weaponStat`, `#hearts`, `#bombs`, `#bossBarWrap`), overlay/start screen, touch controls (`#moveZone`, `#fireBtn`, `#bombBtn`), and the `#gameCanvas` element.
- `<script>` (lines 422–1667): a single block of game code, not wrapped in modules. Major sections, in order:
  - DOM references and `STORAGE_*` keys
  - Performance tracking (`trackPerf`, `quality` 0–2 for reduced effects on slow devices)
  - Tuning tables: `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`
  - Settings persistence (best score, mute, difficulty, reduced motion)
  - Audio: WebAudio helpers `beep`, `noiseBurst`, and the `sfx` table (no audio files)
  - HUD helpers (hearts, bombs, combo pop, stage banner, screen shake)
  - Input: keyboard (`keys` map), touch joystick, fire/bomb buttons, mouse aim on canvas
  - Game state (`player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, etc.)
  - Spawning and combat: `spawnEnemy`, `spawnBoss`, `fireWeapon`, `useBomb`, `applyPowerUp`
  - Boss attack patterns: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`; `updateBoss`, `killBoss`
  - Lifecycle: `resetGame`, `loseLife`, `endGame`, `togglePause`
  - Rendering: `drawPlane`, `drawBossShip`, `drawClouds`, `draw`
  - Main loop: `update(dt)`, `loop(now)` (requestAnimationFrame)

## Game design facts (keep in sync with the code)

- **Canvas:** logical size 480×720 (`BASE_W`, `BASE_H`), scaled to fit the window up to 1.6×.
- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages:** 6 visual themes (`THEMES`) that cycle via `(stage - 1) % THEMES.length`. A stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss ~1.2 s later.
- **Bosses** (`BOSSES`): six designs, each with a name, colors, and HP multiplier. Boss attack patterns are chosen from the `fire*` helpers.
- **Enemies** (`ENEMY_TYPES`): `grunt` (1 HP), `interceptor` (fast, weaving, 1 HP, from stage 1), `tank` (4 HP, from stage 2).
- **Weapons:** `vulcan` (default) and `laser`; weapon power-ups toggle between them and raise `level`.
- **Smart bombs:** clear the screen; the cap (`maxBombs`) and starting count both come from the difficulty's `bombs` value (2 on normal, 3 on easy).
- **Scoring:** kills add score scaled by a chain combo `multiplier`; the chain breaks via `breakChain`.
- **Lives:** `lives`/`maxLives` with an invulnerability window (`player.invuln`) after being hit.

## Controls

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys or WASD | Left-side joystick |
| Fire | Hold Space | Fire button (hold) |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Escape | Pause button |
| Aim | — | Mouse position over canvas |

## Persistence

All persisted state uses `localStorage` with these keys (defined near line 463):

- `skyfighter.bestScore` — high score
- `skyfighter.muted` — audio mute (`'1'`/`'0'`)
- `skyfighter.difficulty` — `easy` | `normal` | `hard`
- `skyfighter.reducedMotion` — `'1'`/`'0'`; defaults to the OS `prefers-reduced-motion` setting until the user toggles it

Access to `localStorage` is not wrapped in try/catch; if you touch these reads/writes, consider guarding them (private browsing or blocked storage can throw).

## Conventions and guidelines

- **Single file:** keep everything in `index.html` unless the user asks for a split. Don't add a build system, framework, or dependencies without asking.
- **No external requests:** the favicon is an inline SVG data URI; keep it that way. Don't add CDN scripts or fonts without a reason.
- **Preserve the mobile layout:** `touch-action: none`, safe-area insets, and the `resize()` scaling logic all support phone play. Test the touch controls after UI changes.
- **Accessibility:** keep the `aria-live` announcer (`srLive`, `announce()`) in sync with important state changes (stage start, boss warnings, game over). Respect reduced-motion for new visual effects.
- **Performance:** the game loop runs every animation frame. Avoid per-frame allocations in `update`/`draw` where possible, and honor the `quality` level when adding particle or effect counts.
- **Tuning changes:** gameplay numbers live in the tables near the top of the script (`DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`). Prefer editing those over hard-coding values inside update logic.

## Git

- Develop on `claude/gifted-tesla-blxww4`; push with `git push -u origin <branch>`.
- Commit only when asked. Do not open pull requests unless explicitly requested.
