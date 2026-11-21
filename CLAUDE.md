# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a `<canvas>` in a single HTML file.

- `index.html` (~1,670 lines): the entire game. Inline `<style>`, markup, and one inline `<script>` IIFE. No external assets, libraries, or network calls.
- `README.md`: placeholder. Its only content is a stray timestamp line, not a real description. Do not treat it as documentation.
- No `package.json`, build step, bundler, linter, or test suite exists. Nothing is installed or compiled.

## Running and verifying

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. For gameplay or UI changes, verify in a real browser (desktop and a phone-sized viewport). The Playwright install is preinstalled; see the environment notes for the browser path.

## Code layout inside `index.html`

- **DOM refs** (`document.getElementById`) are grouped near the top of the script.
- **Storage keys**: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Read and write them through `localStorage` only; keep them wrapped so a blocked or missing storage does not break the game.
- **Difficulty table** `DIFFICULTIES` (`easy` / `normal` / `hard`): controls spawn rate, enemy speed, enemy fire chance, starting lives, starting bombs, and boss HP multiplier. Tune balance here, not in the spawn or combat code.
- **Content tables**: `THEMES`, `ENEMY_TYPES` (grunt, interceptor, tank), `BOSSES`. `KILLS_PER_STAGE` sets stage progression.
- **Audio**: synthesized with the Web Audio API (`beep`, `noiseBurst`, and a named SFX map). Audio starts only after a user gesture via `ensureAudio()`. Keep it that way; browsers block audio otherwise.
- **Game state** is module-level `let` variables (`player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, and so on). `resetGame()` reinitializes them; add any new state there too.
- **Rendering**: canvas drawing helpers (`drawPlane`, `drawBossShip`, and others). The logical play area is `BASE_W` x `BASE_H` (480 x 720), scaled by `resize()`.
- **HUD and overlays** are HTML elements updated from script (hearts, bombs, weapon label, boss bar, stage banner, combo pop).

## Controls (keep the hint text in sync)

- Desktop: mouse to steer and click to fire; arrows or WASD to move; Space to fire; X or Shift for bomb; P or Esc to pause.
- Mobile: drag on the left zone to fly; FIRE and BOMB buttons on screen.
- The on-screen hint is in `#controlsHint`. Update it whenever a control changes.

## Conventions

- Keep everything in `index.html` unless the user asks to split it. Do not add a build system or dependencies without asking.
- Use plain ES2017+ JavaScript and CSS that runs in current evergreen browsers. No transpilation.
- Keep touch and mouse input both working. Any new control needs a mobile path.
- Respect the reduced-motion setting: gate new screen shake, flashes, or animations behind `reducedMotion`.
- Keep the phone layout working: `viewport-fit=cover`, safe-area insets, and no page scroll (`overflow: hidden` is intentional).
- Do not add external CDN scripts, fonts, or analytics without an explicit request.

## Git

- Default development branch for this session: `claude/gifted-tesla-ptatah`. Do not push to other branches without permission.
- Commit only when asked. Do not open a pull request unless the user asks for one.
