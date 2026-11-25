# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a single-file browser game, **Sky Fighter**: a Raiden-style vertical shoot-'em-up. Everything (markup, CSS, and JavaScript) lives in `index.html`. There is no build step, no package manager, no framework, and no test suite.

Files:
- `index.html` (~1,670 lines): the entire game. Inline `<style>` (starts ~line 10) and one inline `<script>` (starts ~line 422).
- `README.md`: a one-line title only.

## Running and testing

- Open `index.html` directly in a browser, or serve the folder with any static server (for example `python3 -m http.server`).
- There are no automated tests, linters, or build commands configured. Verify changes by playing the game in a browser and checking the console for errors.
- For visual or gameplay checks, use the `run` skill or the pre-installed Playwright/Chromium (see the environment notes; do not reinstall Playwright).

## Architecture of `index.html`

The script is one IIFE-style block that wires up DOM references, then defines game state and systems.

- **DOM references** (~lines 426–461): cached elements such as `#gameCanvas`, `#hud`, `#overlay`, `#touchControls`, `#bossBarWrap`, `#stageBanner`.
- **Storage keys** (~lines 463–466): `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`. Changing these keys resets players' saved values, so treat them as a public contract.
- **Performance scaling**: `trackPerf` samples frame times and sets `quality` (0 full, 1 medium, 2 low), which reduces effects on slow devices.
- **Tuning tables** (edit these to rebalance; avoid hard-coding values elsewhere):
  - `DIFFICULTIES` (~line 485): `easy`, `normal`, `hard`, with spawn rates, enemy speed and fire chance, lives, bombs, and boss HP multiplier.
  - `KILLS_PER_STAGE` (~line 491): kills needed to advance a stage (currently 18).
  - `THEMES` (~line 493): six sky palettes, cycled per stage.
  - `ENEMY_TYPES` (~line 502): `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
  - `BOSSES` (~line 508): six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with colors and HP multipliers.
- **Audio**: a Web Audio API context created lazily by `ensureAudio()`. Sound effects are built from `beep()` and `noiseBurst()` and grouped in the `sfx` object. Respect the `muted` flag.
- **Weapons**: two types, `vulcan` and `laser`, stored in `player.weapon`, with a level 1–3. Weapon power-ups raise the level; at level 3 the next power-up switches the weapon type and resets the level to 1 (~line 1315). Fire delay is 0.15 s for vulcan and 0.1 s for laser.
- **Input**:
  - Keyboard (~line 769): arrows/WASD to move; Space to fire; `X` or Shift to bomb; `P` or Esc to pause.
  - Mouse (~line 847): move to steer, click to fire.
  - Touch: left-side joystick (`moveZone`), with `fireBtn` and `bombBtn` on the right.
- **Accessibility**: `reducedMotion` (honors `prefers-reduced-motion` and a user toggle) disables screen shake and flashes. Announcements go through `announce()` into the `#srLive` live region.

## Conventions

- Keep the game in one file unless the user asks for a split. Don't add a bundler, framework, or dependencies without asking.
- Use plain ES2015+ JavaScript and CSS compatible with current evergreen browsers, including mobile Safari and Chrome for Android.
- Keep the layout phone-friendly: the page uses `touch-action: none` and `user-select: none`, and fills the viewport. Don't introduce page scroll.
- New tunable numbers belong in the tuning tables above, not inline in update/draw code.
- Preserve the `aria-live` announcements and `prefers-reduced-motion` behavior when adding effects.
- Don't change `localStorage` keys without a migration plan.

## Git

- Development branch: `claude/gifted-tesla-uhn9z8`. Push there with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks.
- Commit messages: short imperative subject; include the attribution lines required by the current session instructions.
