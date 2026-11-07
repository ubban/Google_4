# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this project is

**Sky Fighter** — a Raiden-style vertical shoot-'em-up that runs in the browser on a 2D `<canvas>`. The whole game is a single self-contained file:

- `index.html` — markup, inline `<style>`, and one inline `<script>` containing all game logic (~1,670 lines).
- `README.md` — minimal placeholder; no real project docs yet.

There is no build step, no bundler, no `package.json`, no test suite, and no linter configured. Nothing is fetched from a network; all art is drawn with canvas primitives and all audio is synthesized with the Web Audio API.

## Running it

Open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). No install step is needed.

## Code layout inside `index.html`

The script is one IIFE-style block. Key sections, roughly in file order:

| Area | What lives there |
|---|---|
| DOM lookups (~L426–461) | Cached element references (`canvas`, `overlay`, HUD elements, touch controls, boss bar). |
| Persistence constants (~L463–466) | `localStorage` keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. |
| Perf scaling (~L474–483) | `quality` tiers (0 full / 1 medium / 2 low) driven by `trackPerf` frame-time averages. |
| Tuning tables (~L485–516) | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`. |
| Audio (~L543–660) | Lazy `ensureAudio()`, `beep`/`noiseBurst` primitives, and the `sfx` table. |
| HUD helpers (~L680–750) | Hearts/bombs rendering, combo pop, stage banner, screen shake, difficulty switch. |
| Input (~L768–861) | Keyboard (`keys` map), virtual joystick + fire/bomb buttons for touch, mouse aiming on canvas. |
| Game state (~L864–910) | `player`, `bullets`, `enemies`, `enemyBullets`, `powerUps`, `boss`, plus `resetGame()`. |
| Drawing (~L913–1070) | `drawPlane`, `drawBossShip`, spawners (`spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`). |
| Combat & scoring (~L1073–1150) | Explosions, `addScore`/chain/multiplier, `useBomb`, `fireWeapon`. |
| Main update (~L1148–1300) | `update(dt)` — movement, collisions, stage progression. |
| Power-ups (~L1304–1322) | `applyPowerUp` for `life`, `bomb`, `weapon`. |
| Boss patterns (~L1324–1410) | Bullet-pattern helpers (`fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, …) and `updateBoss`. |
| Lifecycle (~L1441–1505) | `killBoss`, `loseLife`, `endGame`, `togglePause`. |
| Rendering (~L1507–1640) | `drawClouds`, `draw()`. |
| Main loop (~L1640–1670) | `loop(now)` via `requestAnimationFrame`; start button wiring. |

When changing gameplay, most knobs are in the tuning tables at the top of the script; prefer editing those over hard-coding numbers in logic.

## Game rules (for reference when editing)

- **Difficulty** (`DIFFICULTIES`): `easy` / `normal` / `hard` control spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier.
- **Stages**: each stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss fight. Bosses cycle through `BOSSES` by `(stage - 1) % BOSSES.length`, and HP scales with stage (`70 + stage * 35`) times the boss and difficulty multipliers. Backgrounds cycle through `THEMES`.
- **Enemies**: `grunt` (stage 1+), `interceptor` (stage 1+, weaves), `tank` (stage 2+, 4 HP).
- **Weapons**: `vulcan` and `laser`, each with levels 1–3. A weapon power-up raises the level; past level 3 it toggles to the other weapon and resets to level 1.
- **Power-ups**: `life` (up to max lives), `bomb` (up to max bombs), `weapon`.
- **Scoring**: kills add score scaled by a chain multiplier; the chain breaks on a timer or on getting hit.

## Controls

- Keyboard: arrow keys / WASD to move, Space to fire, `X` / Shift for smart bomb, `P` / Esc to pause.
- Touch: virtual joystick on the left, fire and bomb buttons on the right.
- Mouse: move the pointer over the canvas to steer; hold the button to fire.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a restructure. Do not introduce a build system or external dependencies without asking.
- **No external network calls.** Keep assets procedural (canvas drawing, Web Audio). The only persistence is `localStorage`.
- **Accessibility.** Preserve the reduced-motion toggle (`skyfighter.reducedMotion`, `prefers-reduced-motion` respected), the `srLive` announcer used by `announce()`, and the mute toggle.
- **Mobile-friendly.** Layout scales through `resize()` with `BASE_W = 480`, `BASE_H = 720`. Keep touch controls working and `touch-action: none` on the game surface.
- **Performance.** The main loop uses `dt`-based updates and adaptive `quality` tiers. Avoid per-frame allocations in hot paths (bullet/enemy/particle updates and drawing) and keep new effects gated by `quality` where practical.
- **Styling.** Dark color scheme (`color-scheme: dark`); the HUD uses translucent pills with `text-shadow`. Follow the existing palette in the CSS when adding UI.

## Verifying changes

There are no automated tests. After editing:

1. Open `index.html` in a browser (or serve it locally) and confirm there are no console errors.
2. Start a run, and exercise the touched area: firing and weapon switching, bombs, power-ups, boss fight, pause/resume, and game over.
3. Check difficulty switching, mute, and reduced-motion toggles still persist across reloads.
4. For layout changes, check a narrow (phone-width) viewport and a desktop viewport.

## Git

- Develop on the branch assigned to the session (currently `claude/gifted-tesla-9ucwlj`) and push with `git push -u origin <branch>`.
- Write clear, descriptive commit messages focused on the gameplay or code change.
- Do not open pull requests unless explicitly asked.
