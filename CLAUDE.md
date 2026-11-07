# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

This repo is a single-page browser game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up.

- `index.html` — the entire game: inline CSS, markup, and one inline `<script>` (~1,670 lines). No build step, no bundler, no external assets (audio is synthesized with WebAudio; graphics are drawn on a `<canvas>`).
- `README.md` — minimal placeholder.

There is no package.json, test suite, linter, or CI config. Verify changes by opening `index.html` in a browser (or serving the directory with any static server) and playing.

## Running locally

```bash
# any static server works; the file has no dependencies
python3 -m http.server 8000   # then open http://localhost:8000/index.html
```

Opening the file directly via `file://` also works.

## Code layout (inside `index.html`)

Approximate line anchors — they will drift as the file changes:

| Area | What's there |
|---|---|
| `<style>` (top) | Layout, HUD, touch-control and overlay styling. Uses `env(safe-area-inset-*)` and `clamp()` for phone sizing. |
| Storage keys (~L463) | `skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion` in `localStorage`. |
| Adaptive quality (~L473) | `trackPerf()` steps `quality` 0–2 based on rolling frame time. |
| `DIFFICULTIES` (~L485) | Per-difficulty spawn rates, enemy speed, shot chance, starting lives/bombs, boss HP multiplier. |
| `THEMES` (~L493) | Six sky palettes; the theme cycles with `(stage - 1) % THEMES.length`. |
| `ENEMY_TYPES` / `BOSSES` (~L502–520) | Enemy stats (grunt, interceptor, tank) and boss designs (six, cycled per stage). |
| Audio (~L542–680) | `ensureAudio()`, `beep()`, `noiseBurst()`, and the `sfx` table. |
| Canvas sizing (~L753) | Logical size is `BASE_W=480 × BASE_H=720`, scaled to fit the viewport in `resize()`. |
| Input (~L767–862) | Keyboard, on-screen joystick, FIRE/BOMB touch buttons, and mouse/pointer handlers. |
| Game state (~L863–912) | `resetGame()` and the globals `player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, `stage`, `kills`. |
| Spawning / weapons (~L1022–1147) | `spawnEnemy()`, `spawnBoss()`, `fireWeapon()`, `useBomb()`. |
| `update(dt)` (~L1148) | Per-frame simulation: movement, firing, collisions, stage progression. |
| Power-ups (~L1304) | `applyPowerUp()`: `life`, `bomb`, and `weapon` (raises level 1→3, then toggles vulcan↔laser). |
| Boss patterns (~L1324–1410) | Bullet-pattern helpers and `BOSS_PATTERNS`; `updateBoss()` and `killBoss()`. |
| Flow (~L1456–1505) | `loseLife()`, `endGame()`, `togglePause()`. |
| Rendering (~L1507–1639) | `draw()`, plus plane/boss drawing helpers. |
| `loop(now)` (~L1640) | `requestAnimationFrame` main loop. |

### Game rules worth knowing

- A stage lasts `KILLS_PER_STAGE` (18) kills, then a boss spawns. Defeating the boss advances `stage`.
- Enemy types unlock by `minStage` (tanks from stage 2).
- Power-ups drop from destroyed enemies: `life`, `bomb`, or `weapon`.
- Bombs clear all enemy bullets and enemies on screen and deal fixed damage to the boss.
- Game state is reset by `resetGame()`. Anything new added to game state must be initialized there too.

## Conventions

- **Keep it a single file.** Styles, markup, and script stay inline in `index.html` unless the user asks for a split. Don't introduce a build step or external CDN dependencies without a reason.
- **No external assets.** Sounds come from WebAudio, art from canvas primitives. Don't add image or audio files unless requested.
- **Persist settings through the existing `STORAGE_*` keys** and wrap new `localStorage` access in the same style as the existing code.
- **Respect the reduced-motion toggle.** Screen shake, flashes, and pulses should honor `reducedMotion` (shake is damped, flashes are skipped; the `reduced-motion` body class is also set).
- **Keep touch and keyboard parity.** Any new control must work with both the keyboard and the touch UI (`#touchControls`, shown via the `enabled` class on touch devices).
- **Accessibility:** announce important state changes through the existing `announce()` live region.
- **Accept the existing indentation** (two spaces inside `<script>`/`<style>`) and single-quoted JS strings.

## Making changes

1. Read the surrounding section before editing; the file is large and most logic is in one `<script>` block.
2. Test in a browser after each gameplay change: start a game, fire, use a bomb, collect power-ups, reach a boss, and check game over and the best-score save. Check at least one phone-sized viewport if you touched layout or touch controls.
3. Commit with a clear message describing the gameplay or UI effect.

## Git

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-m9puj1`). Do not push to other branches without permission.
- Do not open a pull request unless explicitly asked.
