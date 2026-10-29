# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML file on a `<canvas>` element.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: inline CSS (`<style>`), markup, and one IIFE-wrapped `<script>` (~1,670 lines). |
| `README.md` | Placeholder title only. |

There is no build step, package manager, bundler, test suite, or linter. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example `python3 -m http.server`. No install is needed.

## Architecture of `index.html`

- **Styles**: one `<style>` block. Colors are hard-coded hex or `rgba()` values. HUD, overlay card, touch controls, and boss bar are all positioned with absolute or fixed layout inside `#container`.
- **Markup**: `#hud` (score, best, multiplier, weapon, hearts, bombs), `#bossBarWrap`, `#comboPop`, `#stageBanner`, `#topBar` (mute, reduced-motion, pause), `#touchControls`, and `#overlay` (title card, difficulty buttons, start button, controls hint).
- **Script**: a single IIFE. Key sections, roughly in file order:
  - DOM references and `STORAGE_*` keys (`skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`).
  - Tuning tables: `DIFFICULTIES`, `THEMES` (per-stage sky palettes), `ENEMY_TYPES`, `BOSSES`, plus `KILLS_PER_STAGE`.
  - Audio: lazily created `AudioContext`, with `beep()` and `noiseBurst()` helpers. `sfx.*` wraps them.
  - Rendering helpers: `drawPlane`, `drawBossShip`, `drawClouds`, and `draw()`.
  - Gameplay: `spawnEnemy`, `spawnBoss`, `spawnExplosion`, `useBomb`, `fireWeapon`, `update(dt)`, `applyPowerUp`, `updateBoss(dt)`, and the boss-pattern helpers (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`).
  - Input: keyboard listeners, mouse/pointer handling, and touch controls (joystick, FIRE, BOMB).
  - Loop: `loop(now)` drives `update(dt)` and `draw()` through `requestAnimationFrame`. `dt` is clamped to 0.05 s.
- **Adaptive quality**: `trackPerf(dt)` samples frame times and sets `quality` (0 full, 1 medium, 2 low) to reduce effects when frames run slow.

### Game rules to keep in mind

- Difficulty (`easy` / `normal` / `hard`) changes spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier.
- Weapons are `vulcan` and `laser`, each with three levels. A weapon power-up raises the level first. At level 3 it switches weapon and resets to level 1.
- Power-up types are `life`, `bomb`, and `weapon`. `maybeSpawnPowerUp()` rolls a random drop with weights of 6%, 8%, and 20%.
- Stages advance after `KILLS_PER_STAGE` kills. Each stage ends in a boss from `BOSSES`, and the theme cycles through `THEMES`.
- Kill chains feed a score multiplier (`#mult`).
- Best score persists in `localStorage`. Every storage read and write should stay wrapped so the game still works when storage is blocked.

## Conventions

- **Keep it one file.** New game code goes into `index.html` in the existing IIFE, unless the user asks for a split. Do not introduce modules, a build step, or external dependencies without a request.
- **Match the local style.** Use 2-space indentation, single quotes in JS, `const`/`let` (no `var`), and short arrow functions. Tuning values live in the constant tables near the top of the script. Put new balance numbers there rather than inline.
- **Comments are sparse.** Follow the file's pattern: short section-level comments, plus brief notes where a formula is non-obvious.
- **Accessibility and motion.** Honor the reduced-motion toggle (`body.reduced-motion`) for any new screen shake, flashing, or pulsing. New interactive controls need a `title` or `aria-label`, and text announcements go through `announce()` / `#srLive`.
- **Mobile.** Keep layouts working at phone width, with `touch-action: none` on game surfaces. Test new HUD elements at narrow widths.

## Verification

With no test harness, verify changes by loading `index.html` in a browser and playing through the affected path. For logic-only edits, at least confirm the page loads without console errors. Check mobile layout for HUD or overlay changes.

## Git

- Commit to the working branch the session specifies, and keep commits focused on one gameplay or UI change.
