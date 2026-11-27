# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo contains **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. The entire game is a single self-contained file, `index.html`, with inline CSS and JavaScript and no external assets (audio is synthesized with WebAudio).

Other files:
- `README.md` — a placeholder title only.

There is no package manager, build step, bundler, test suite, or linter. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Code layout (`index.html`)

The file is organized as one `<style>` block, the DOM markup, and one `<script>` block. Major sections of the script, in order:

- **Storage keys and persisted settings** (`STORAGE_*`): best score, mute, difficulty, and reduced motion are stored in `localStorage` under the `skyfighter.*` keys.
- **Difficulty table** (`DIFFICULTIES`): spawn rates, enemy speed, enemy fire chance, lives, bombs, and boss HP multiplier for `easy`, `normal`, and `hard`.
- **Constants**: `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, and `BASE_W`/`BASE_H` (480×720 logical play area).
- **Adaptive performance** (`trackPerf`): adjusts rendering cost based on frame timing.
- **Audio** (`ensureAudio`, `beep`, `noiseBurst`, `sfx`): WebAudio synthesis only. Audio is created lazily on first user gesture.
- **Canvas sizing** (`resize`): scales the canvas to fit the viewport.
- **Input**: keyboard (`keydown`/`keyup` on `window`), mouse steering, and a touch joystick (`joystickStart`/`joystickMove`/`joystickEnd`). Keyboard and joystick input take priority over mouse steering.
- **Game state** (`resetGame`, `update`, `draw`, `loop`): the main loop is driven by `requestAnimationFrame`. `update(dt)` handles movement, collisions, spawning, and scoring; `draw()` renders everything.
- **Weapons and bosses**: `fireWeapon`, the `fire*` pattern helpers, and `BOSS_PATTERNS` (one attack set per boss, selected by stage). `updateBoss` and `killBoss` manage boss phases and stage progression.
- **Power-ups and bombs**: `maybeSpawnPowerUp`, `applyPowerUp`, `useBomb`.
- **Lifecycle**: `loseLife`, `endGame`, `togglePause`.

### Controls (for reference when testing)

- Move: arrow keys or WASD; mouse hover steers; touch drag on the left side flies the plane.
- Fire: Space, mouse button (hold), or the on-screen FIRE button on touch devices.
- Bomb: `X` or Shift.
- Pause: `P` or Escape.
- Mute, difficulty, and reduced motion are toggled from the in-game UI.

## Conventions

- **Keep it a single file** unless the user asks to split it. Do not add a build system, package manager, or framework without an explicit request.
- **No external dependencies or network requests.** The game is designed to work offline with no assets. Do not add CDN scripts, fonts, or images without asking.
- **Match the existing style**: two-space indentation, `const`/`let`, plain ES2017+ JavaScript, and small named functions grouped under `// ---------- Section ----------` banners.
- **Mobile matters.** Keep the layout working at phone widths and preserve the touch controls and safe-area insets (`env(safe-area-inset-*)`).
- **Respect reduced motion.** Any new screen shake, flashing, or large animation should honor the `reducedMotion` setting.
- **Persist settings through the existing `skyfighter.*` keys** and wrap `localStorage` access so the game still runs when storage is unavailable.
- **Keep the frame loop cheap.** Avoid allocating objects or touching the DOM inside `update`/`draw` per frame where a reusable value works.

## Verifying changes

Since there is no test suite, verify by running the game:

1. Open `index.html` in a browser (or serve it locally) and check the console for errors.
2. Exercise the path you changed: movement, firing, a bomb, a power-up, a boss fight, game over, and pause.
3. For layout changes, check both a desktop viewport and a phone-sized viewport, and confirm there is no horizontal scroll.
4. For audio changes, confirm sound plays after the first interaction and that mute still works.

Where a headless browser is available, a Playwright script can drive these checks, but it is not required.

## Git workflow

- Default development branch for this session: `claude/gifted-tesla-9y4est`. Commit and push there; do not push to other branches without explicit permission.
- Do not open a pull request unless the user asks for one.
- Use clear, imperative commit messages that describe the gameplay or code change.
