# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Sky Fighter is a single-page, Raiden-style vertical shoot-'em-up built on an HTML5 canvas. The whole game lives in `index.html`: markup, CSS, and one inline `<script>` IIFE. There is no build step, no package manager, no bundler, and no test suite. `README.md` is a placeholder.

To run it, open `index.html` in a browser (or serve the directory with any static server).

## Repository layout

- `index.html` — the entire game (about 1,670 lines)
  - Lines ~10–359: inline `<style>` (HUD pills, overlay card, touch controls, reduced-motion rules)
  - Lines ~360–421: DOM skeleton (`#hud`, `#stageBanner`, `#overlay`, `#touchControls`, icon buttons)
  - Lines ~422–end: single `<script>` wrapped in an IIFE; all state is local to it
- `README.md` — placeholder title only

## Architecture of the script

Everything runs inside one IIFE. Key sections, in file order:

1. **DOM lookups** — every element is grabbed once at the top by id.
2. **Config constants** — `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES` (grunt / interceptor / tank with `minStage` gating), `BOSSES` (six boss designs, cycled by stage).
3. **Persisted settings** — best score, mute, difficulty, and reduced motion are read from and written to `localStorage`.
4. **Audio** — a lazily created `AudioContext` (`ensureAudio`), with `beep`, `noiseBurst`, and a `sfx` table. Audio is created on the first user gesture.
5. **Feedback helpers** — hearts/bombs HUD rendering, hit flash, combo pop, stage banner, screen shake.
6. **Layout** — `resize()` scales a fixed 480×720 logical canvas to fit the window (max scale 1.6).
7. **Input** — keyboard (`keys` map keyed by `e.code`), mouse-follow on the canvas, and touch joystick plus fire/bomb buttons.
8. **Game state and mechanics** — `resetGame`, `update(dt)`, `fireWeapon`, `useBomb`, `spawnEnemy`, `spawnBoss`, `maybeSpawnPowerUp`, `addScore` / `breakChain` (kill-chain multiplier).
9. **Rendering** — `draw()` with helpers `drawPlane`, `drawBossShip`, `drawClouds`, explosions and particles.
10. **Loop** — `loop(now)` driven by `requestAnimationFrame`. `dt` is clamped to 0.05 s per frame. A rolling frame-time tracker (`trackPerf`) adjusts the `quality` level (0 full, 1 medium, 2 low), which controls glow and particle counts.

### Gameplay constants to know

- Controls: Arrow keys or WASD to move, Space to fire, X or Shift for a bomb, P or Escape to pause. Touch uses the on-screen joystick plus fire and bomb buttons.
- Difficulty (`easy` / `normal` / `hard`) sets spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and a boss HP multiplier.
- Weapons: Vulcan and Laser, each with power levels 1–3 that change spread and lanes.
- Stages: an 18-kill quota per stage, then a boss fight with a health bar. Boss HP scales with stage (`70 + stage * 35`, times the difficulty and boss multipliers).

## Conventions

- Keep the game in a single file unless the user asks for a split. Do not introduce a build tool, framework, or npm dependency without a clear request.
- Match the existing style: 2-space indent, single quotes in JS, `const`/`let`, arrow functions for callbacks, and short section banners in the form `// ---------- Name ----------`.
- Preserve the mobile and accessibility behavior. Respect `prefers-reduced-motion` and the in-game reduced-motion toggle when adding screen shake, flashes, or pulses. Keep the `#srLive` announcements for important state changes.
- Use `Number(localStorage.getItem(...)) || default`-style reads, and keep new storage keys under the `skyfighter.` prefix.
- Coordinates are in logical 480×720 space; do not hard-code CSS pixel sizes for in-game objects.
- Time-based motion should go through `dt`, not frame counts. The difficulty ramp (`frame / 300`) is a deliberate exception.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser and check the browser console for errors.
2. Exercise the affected path: start a game, play through a stage transition if the change touches stages or bosses, pause and resume, use a bomb, and check game-over and best-score persistence.
3. For HUD or layout changes, check both a desktop-sized window and a narrow phone-sized viewport, including touch controls.
4. If the change touches input, test both keyboard and touch paths.

If you add tests or tooling, document the commands here.

## Git

- Default branch: `main`. Feature work for this session happens on the branch named in the session instructions.
- Commit messages should be short imperative summaries, with a body when the change needs context. The existing history uses descriptive multi-paragraph messages for larger features.
