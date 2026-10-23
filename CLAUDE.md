# CLAUDE.md

Guidance for working in this repository.

## What this is

`Google_4` is a single-page HTML5 canvas game, **Sky Fighter** — a Raiden-style vertical shoot-'em-up. The whole game lives in one file:

- `index.html` (~1,670 lines) — inline `<style>` (lines ~10–359), HTML markup (~361–421), and one inline `<script>` (~422–1667).
- `README.md` — placeholder only.

There is no build step, no package manager, no bundler, no test suite, no linter config, and no external assets. Audio is synthesized with WebAudio. Everything must keep working by opening `index.html` directly in a browser.

## Running it

- Open `index.html` in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verifying a change means playing it in the browser. There are no automated tests.

## Code layout (inside `index.html`)

**Markup** — `#container` holds the `<canvas id="gameCanvas">`, a `#hud` (score, best, multiplier, weapon, hearts, bombs), boss bar, combo/stage banners, `#topBar` icon buttons (mute, reduced motion, pause), touch controls (`#touchControls`: joystick zone, FIRE, BOMB), and `#overlay` (title / game-over card with difficulty buttons and `#startBtn`).

**Script** — a single IIFE-style block. Sections, in rough order:

1. **DOM refs and storage keys.** `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` (all `skyfighter.*` in `localStorage`).
2. **Perf / quality.** `trackPerf()` samples frame times and sets `quality` (0 full, 1 medium, 2 low) to reduce effects on slow devices.
3. **Tuning tables** — the main place to change balance:
   - `DIFFICULTIES` (easy/normal/hard): spawn timing, enemy speed, enemy shoot chance, starting `lives`, starting `bombs`, `bossHpMul`.
   - `KILLS_PER_STAGE` (18) — kills needed before a boss is warned and spawned.
   - `THEMES` — six stage palettes, cycled by `(stage - 1) % THEMES.length`.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, minStage 2).
   - `BOSSES` — six bosses (VANGUARD → TITAN), cycled per stage, each with colors and `hpMul`.
4. **Audio** — `ensureAudio()` lazily creates the `AudioContext` on the first user gesture; `beep()`, `noiseBurst()`, and the `sfx` object build all sounds.
5. **UI helpers** — `renderHearts`, `renderBombs`, `flashHit`, `flashBomb`, `showComboPop`, `showStageBanner`, `screenShake`, `announce()` (polite ARIA live region).
6. **Input** — `keys` map (keyboard via `e.code`), mouse steering/firing on the canvas, touch joystick (`joystickStart/Move/End`), and FIRE/BOMB buttons.
7. **Game state** — `resetGame()` initializes `player`, arrays (`bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`), `boss`, score, lives, bombs, stage, chain/multiplier.
8. **Drawing** — `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
9. **Spawning and combat** — `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `fireWeapon`, `useBomb`, `addScore` / `breakChain` (kill-chain multiplier), and the boss attack patterns (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`) driven by `updateBoss`.
10. **Update / lifecycle** — `update(dt)` (per-frame logic and collisions), `applyPowerUp`, `killBoss`, `loseLife`, `endGame`, `togglePause`.
11. **Main loop** — `loop(now)` calls `update` then `draw` each frame via `requestAnimationFrame`.

## Game rules worth knowing

- **Weapons:** `vulcan` and `laser`, each with power `level` 1–3. A `weapon` pickup raises the level; at level 3 it swaps to the other weapon and resets to level 1.
- **Power-up drops** (`maybeSpawnPowerUp`): 6% life, 8% bomb, 20% weapon.
- **Bombs** (`useBomb`) clear all enemy bullets, destroy every enemy (+5 score each), and deal 24 damage to the boss. Starting count comes from difficulty; pickups add one up to `maxBombs`.
- **Kill chain:** each kill increments `chain`; `multiplier = 1 + min(9, floor(chain / 5))`. The chain breaks after 2.2 s without a kill or when the player is hit.
- **Stages:** after `KILLS_PER_STAGE` kills, a boss warning fires and the boss spawns ~1.2 s later. Defeating it awards 500 points and advances `stage`.
- **Damage:** `loseLife()` grants 1.5 s of invulnerability (`player.invuln`) and ends the run at 0 lives.
- **Persistence:** best score is saved to `localStorage` on game over. Mute, difficulty, and reduced-motion choices persist too. Storage access is not wrapped in try/catch today.

## Controls

- Keyboard: arrows / WASD to move, Space to fire, X or Shift for a bomb, P or Esc to pause.
- Mouse: move to steer, click to fire.
- Touch: drag left zone to fly, tap FIRE, tap BOMB.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split it. The single-file, zero-dependency design is intentional.
- Match the existing style: 2-space indent, single quotes in JS, `const`/`let`, short section comments with `// ---------- Name ----------` banners, and `const` tuning tables near the top of the script.
- Don't add external scripts, fonts, or image assets without a reason. Sounds stay synthesized.
- Respect the reduced-motion path: any new screen shake or flash should go through `screenShake` / `flashHit` / `flashBomb` so the `reducedMotion` check applies.
- Keep touch support working. Any new control needs a touch equivalent.
- Keep the canvas at the logical size `BASE_W` × `BASE_H` (480 × 720); `resize()` scales it to the viewport.
- Respect `prefers-reduced-motion` and keep accessibility hooks (`announce`, `#srLive`, `aria-*`) when adding UI.

## Verifying changes

Since there is no test harness:

1. Open `index.html` in a desktop browser and play through at least one stage, a boss fight, and a game over.
2. Check all three difficulties if balance values changed.
3. Check the mobile layout (narrow viewport or device emulation) and touch controls if input or HUD changed.
4. Check the console for errors.
5. Confirm that mute, pause, reduced-motion, and best score still persist across reloads.
