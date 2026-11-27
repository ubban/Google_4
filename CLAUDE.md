# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, single-file web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

| File | Purpose |
|------|---------|
| `index.html` | The entire game: markup, CSS, and JavaScript in one file (~1,670 lines). |
| `README.md` | Placeholder (title only). Keep it accurate if you expand it. |

There is no build step, no package manager, no bundler, no framework, and no test suite. Do not introduce any of these without asking first.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

- **`<style>` (around lines 10–360):** all CSS. Colors and layout for the HUD, boss bar, overlay card, and touch controls. The reduced-motion preference is read in JS (`matchMedia('(prefers-reduced-motion: reduce)')`), not in CSS.
- **`<body>` (around lines 362–421):** DOM scaffolding. Key IDs: `gameCanvas`, `hud`, `bossBarWrap`, `comboPop`, `stageBanner`, `touchControls` (`moveZone`, `joystickBase`, `actionZone`), `overlay`/`overlayCard`, `diffRow`, `statsRow`, and the `srLive` live region used for screen-reader announcements.
- **`<script>` (around line 422 onward):** all game logic, in roughly this order:
  1. Constants: `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BASE_W`/`BASE_H` (480×720 logical resolution).
  2. Persistence and settings: best score, mute, difficulty, reduced motion.
  3. Audio: a Web Audio synth (`ensureAudio`, `beep`, `noiseBurst`, `panFor`).
  4. HUD and effects helpers: hearts, bombs, flashes, combo pop, stage banner, screen shake.
  5. Canvas sizing and input: `resize`, keyboard handlers, joystick, mouse and touch handlers.
  6. Game state: `resetGame`, the main `update(dt)` loop, `drawPlane`, `drawBossShip`, spawning, collisions (`rectsOverlap`), and explosions.
  7. Weapons and bombs: `fireWeapon`, `useBomb`, `applyPowerUp`.
  8. Bosses: `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireCross`, `fireHomingSeeded`), `updateBoss`, `killBoss`.
  9. Life and game-over flow: `loseLife` and onward.

Search for a function name with `grep -n "function <name>" index.html` rather than relying on line numbers, which will drift as the file changes.

## Game design facts (keep these consistent when editing)

- **Difficulties** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier.
- **Stages:** `KILLS_PER_STAGE` (18) kills advance the stage. Each stage uses a theme from `THEMES` (six themes, cycled). Tank enemies appear from stage 2 (`minStage`).
- **Bosses:** `BOSSES` has six entries with per-boss `hpMul`. Each boss cycles through `BOSS_PATTERNS`.
- **Weapons:** two weapon types, `vulcan` and `laser`, toggled with the weapon switch logic near line 1315. Power levels are applied via `applyPowerUp`.
- **Score:** kill-chain combo multiplier (`addScore`, `breakChain`). Best score persists to `localStorage`.

## Controls

| Action | Keyboard | Touch |
|--------|----------|-------|
| Move | `WASD` or arrow keys | Left-side joystick |
| Fire | `Space` (hold) | Fire button (hold) |
| Smart bomb | `X` or `Shift` | Bomb button |
| Pause | `P` or `Esc` | — |
| Mouse | Move pointer over canvas to steer; mouse-down fires | — |

## Persistence keys

All persisted values use `localStorage` under these keys. Changing a key name silently resets that setting for existing players, so only rename with a migration.

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Note: the existing `localStorage` calls are not wrapped in try/catch. Any new storage access should be wrapped so the game still runs if storage is blocked (private windows, disabled site data).

## Conventions

- **Keep it a single file.** Put new CSS in `<style>`, new JS in the existing `<script>`, and do not add external dependencies or CDN links unless the user asks for them.
- **Be careful in the per-frame path.** `update(dt)` and the draw functions run every frame. Avoid adding per-frame allocations (new arrays, objects, or closures) in those paths.
- **Use the existing helpers.** Reuse `announce()` for screen-reader messages, `showStageBanner()`/`showComboPop()` for on-screen text, and `screenShake()`/`flashHit()` for impact feedback, rather than duplicating them.
- **Respect reduced motion.** New visual effects that shake, flash, or pulse should check the `reducedMotion` flag (set from storage or the OS preference).
- **Mobile first.** The layout must work at phone width with no horizontal scroll. Touch controls rely on `touch-action: none`; do not remove that.
- **Naming:** `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constants, and `STORAGE_*` for persistence keys.
- **Style:** 2-space indent, single quotes in JS, semicolons, matching the surrounding code.

## Verifying changes

There is no automated test suite. Before declaring a change done:

1. Open `index.html` in a browser (or use the pre-installed Playwright Chromium in this environment) and confirm there are no console errors on load.
2. Play through at least one stage start, a weapon switch, a bomb, a pause/resume, and a game-over.
3. For layout or touch changes, check a narrow (phone-sized) viewport as well as a desktop one.
4. For audio changes, confirm the mute toggle still works and that nothing throws when audio is unavailable.

If you cannot run the game, say so in your summary and limit claims to what you verified by reading the code.

## Git and commits

- Default branch work happens on the branch set up for the session. Do not push to other branches without explicit permission.
- Write clear, imperative commit messages that describe the gameplay or code change.
- Do not open a pull request unless the user asks for one.
