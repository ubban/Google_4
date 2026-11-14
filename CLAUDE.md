# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in the browser. It is a single, self-contained file:

- `index.html` — all markup, CSS (`<style>`), and JavaScript (one IIFE in a `<script>` block starting around line 422). Audio is synthesized with WebAudio; there are no external assets, images, or libraries.
- `README.md` — minimal placeholder.

There is no build step, package manager, test suite, or linter configured. Nothing needs to be installed.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```sh
python3 -m http.server 8000
```

## Code layout inside `index.html`

The game logic lives in the IIFE. Key areas, roughly in file order:

- **Constants**: `STORAGE_*` keys for `localStorage`, `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`.
- **Persisted settings**: best score, mute, difficulty, and reduced motion are stored in `localStorage` under `skyfighter.*` keys. Reduced motion defaults to `prefers-reduced-motion` when no saved value exists.
- **Audio**: `ensureAudio`, `beep`, `noiseBurst`, and the `EXPLODE_SFX` map. Audio is created lazily on the first user gesture.
- **Layout and input**: `resize` (fixed `BASE_W = 480`, `BASE_H = 720` logical canvas, scaled to fit), keyboard handler, touch joystick (`joystick*`), and pointer-to-canvas mapping (`canvasPointFromEvent`).
- **Game state**: `resetGame`, `update(dt)` (main simulation step), `applyPowerUp`, `useBomb`, `fireWeapon`.
- **Enemies and bosses**: `spawnEnemy`, `spawnBoss`, `BOSS_PATTERNS` (bullet-pattern functions such as `fireRadial`, `fireSpiralBurst`, `fireHomingSeeded`), `updateBoss`, `killBoss`.
- **Rendering**: `drawPlane`, `drawBossShip`, plus HUD helpers (`renderHearts`, `renderBombs`, `showStageBanner`, `showComboPop`, `screenShake`).
- **Loop and lifecycle**: `loseLife`, `endGame`, and the `requestAnimationFrame` loop at the bottom of the IIFE.

Keep new code inside the existing IIFE and follow its style: `const`/`let`, two-space indentation, and section comments such as `// ---------- Audio ----------`.

## Controls (for reference when testing)

- Arrow keys / WASD: move (read from `keys[e.code]` in `update`)
- Space: fire (hold). Mouse/touch also fire via the fire button or by pressing on the canvas.
- X or Shift: smart bomb
- P or Escape: pause
- Touch: on-screen joystick and fire zone, shown only on touch devices

## Conventions for changes

- Keep the game as a single self-contained HTML file unless the user asks for a split. Do not add external CDN scripts or fonts without being asked.
- Preserve the `localStorage` key names (`skyfighter.*`); changing them silently resets players' saved data.
- Tune gameplay through the `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and `THEMES` tables rather than hard-coding numbers in the loop.
- Keep the layout working at phone width. The page uses `touch-action: none` and `overflow: hidden`, so avoid changes that add page scroll.
- Respect the reduced-motion setting when adding screen shake, flashes, or other motion effects.

## Verification

No automated tests exist. For gameplay or rendering changes, load the page in a browser and play through at least one stage and one boss fight on the affected difficulty. For UI changes, check both desktop and a narrow phone-width viewport.

## Git

- Development branch for this work: `claude/gifted-tesla-algtm2`.
- Commit messages should describe the change in the imperative mood.
