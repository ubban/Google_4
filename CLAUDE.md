# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single self-contained file:

- `index.html` — all markup, CSS (`<style>` block), and game logic (one `<script>` block, an IIFE starting around line 423).
- `README.md` — a stub containing only the repo name.

There is no build step, no package manager, no bundler, and no test suite. To run the game, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout and conventions

- Keep the game in `index.html` unless the user asks for a split. Do not introduce a build system, framework, or npm dependencies without an explicit request.
- Use vanilla JavaScript and the 2D canvas API, matching the existing style (`const`/`let`, function declarations, 2-space indentation).
- Game logic is grouped by section inside the IIFE; keep new code near related code (e.g. enemy definitions near `ENEMY_TYPES`, boss patterns near `BOSSES` and the `fire*` helpers).
- Base canvas size is `BASE_W = 480`, `BASE_H = 720`; `resize()` scales it to the viewport (max scale 1.6).

## Key game structure (for orientation)

Constants and tuning live near the top of the script:

- `DIFFICULTIES` — `easy` / `normal` / `hard` settings: spawn rates, enemy speed and shot chance, starting lives and bombs, and `bossHpMul`. Difficulty is persisted in `localStorage`.
- `KILLS_PER_STAGE` (18) — kills needed before a boss warning and boss spawn.
- `THEMES` — six background palettes, cycled by `(stage - 1) % THEMES.length`.
- `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, unlocked at stage 2).
- `BOSSES` — six bosses, cycled by stage. Each has a distinct attack pattern built from the `fire*` helpers (radial, aimed spread, columns, spiral, cross, homing, double ring).

Core runtime functions:

- `resetGame()` — initializes player and state for a new run.
- `update(dt)` / `draw()` / `loop(now)` — the main frame cycle.
- `spawnEnemy()`, `spawnBoss()`, `updateBoss(dt)`, `killBoss()` — enemy and boss lifecycle. `killBoss()` advances `stage`.
- `fireWeapon()` — player weapons (`vulcan` and `laser`, switched by weapon power-ups via `applyPowerUp`).
- `useBomb()` — smart bomb: clears enemy bullets, destroys all on-screen enemies, and deals 24 damage to the boss.
- `loseLife()` / `endGame()` — damage and game-over flow. Best score is saved to `localStorage`.

Persisted `localStorage` keys (use these exact names; they are user-visible state):

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

Wrap any new `localStorage` access in try/catch, since storage can be unavailable (private windows, blocked site data).

## Controls (keep in sync with the in-game hint)

- Desktop: mouse to steer and click to fire; arrows/WASD to move; Space to fire; X or Shift for bomb; P or Esc to pause.
- Touch: drag left side to fly (virtual joystick), tap FIRE, tap BOMB.

If you change a control, update the `#controlsHint` text and the touch controls markup together.

## Accessibility and motion

- The game respects `prefers-reduced-motion` and offers a manual toggle (`motionBtn`). Reduced motion disables or softens screen shake, flashes, and stage-banner animation. New visual effects should check `reducedMotion`.
- Announcements for screen readers go through `announce()` (the `srLive` region). Use it for meaningful state changes such as stage transitions and boss defeat.
- Keep `touch-action`, `user-select`, and safe-area inset rules in the CSS intact so the layout works on phones.

## Making changes

- Read the relevant section of `index.html` before editing; the file is long (~1,670 lines), so use targeted searches.
- Verify changes by loading the page in a browser and playing through at least one boss fight, or by using the pre-installed Playwright/Chromium if automated checks are needed. There are no unit tests to run.
- Do not add tracking, analytics, or external network requests to the game.
- Keep changes focused: do not reformat unrelated code or rename existing identifiers.

## Git

- Development branch for this session: `claude/gifted-tesla-ib4a60`.
- Write clear, descriptive commit messages.
- Do not open pull requests unless explicitly asked.
