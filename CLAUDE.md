# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up implemented as a single HTML5 canvas page.

## Repository layout

| Path        | Purpose                                                                 |
|-------------|-------------------------------------------------------------------------|
| `index.html`| The entire game: inline `<style>`, markup, and a single inline `<script>` (~1,670 lines). |
| `README.md` | Minimal project readme.                                                 |

There is no build step, package manager, test suite, or linter configured. Do not add one unless asked.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000
```

## Architecture notes for `index.html`

The game is organized inside one IIFE-style script. Key pieces, roughly in file order:

- **Persistence keys** (`skyfighter.*` in `localStorage`): best score, mute, difficulty, reduced motion. All reads/writes are guarded by the existing defaults; keep that pattern when adding settings.
- **`DIFFICULTIES`**: `easy` / `normal` / `hard` tune spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and boss HP multiplier. Change balance here rather than scattering magic numbers.
- **`BOSSES` and `BOSS_PATTERNS`**: boss definitions and bullet patterns (radial, aimed spread, columns, spiral, cross, homing, double ring). Add new patterns as functions and register them in `BOSS_PATTERNS`.
- **`THEMES`**: sky/cloud/sun palettes cycled per stage. `KILLS_PER_STAGE` controls stage progression.
- **Weapons**: Vulcan and Laser, each with 3 power levels (`fireWeapon`, `applyPowerUp`).
- **Bombs**: limited smart bombs (`useBomb`) that clear the screen and damage bosses.
- **Scoring**: `addScore` and `breakChain` implement the kill-chain multiplier.
- **Audio**: Web Audio API, created lazily in `ensureAudio()`. Audio must only start after a user gesture; keep calls to `ensureAudio()` in input handlers.
- **Main loop**: `update(dt)` → `draw()` → `requestAnimationFrame(loop)`. `dt` is frame-time based; keep game logic framerate-independent.
- **Input**:
  - Keyboard: arrows/WASD to move (see `keydown` handler), `Space` to fire, `X` / `Shift` for bomb, `P` / `Esc` to pause.
  - Touch: on-screen joystick (`joystickStart/Move/End`), fire and bomb buttons. Touch handlers use `passive: false` and `preventDefault`.
- **Accessibility**: a reduced-motion toggle disables screen shake and similar effects; `announce()` feeds an ARIA live region. Preserve both when adding visual effects.

## Conventions

- Keep the game as a single self-contained `index.html` unless the user asks for a split. No external network dependencies, CDNs, or frameworks.
- Match the existing style: 2-space indentation, `const`/`let`, small named functions, and comments only where the logic is not obvious.
- Preserve mobile support: `viewport` meta, `touch-action: none`, and the touch control layout must keep working on phone-sized screens.
- Don't break the `localStorage` keys; renaming one resets players' saved data.

## Verifying changes

No automated tests exist. After editing `index.html`:

1. Open it in a browser (or the pre-installed Chromium via Playwright) and confirm the title screen renders.
2. Start a game and check movement, firing, bombs, pause, and game over.
3. Check at least one mobile-sized viewport when touching layout, HUD, or touch controls.
4. Check the browser console for errors.

## Git workflow

- Develop on the branch assigned for the session (currently `claude/gifted-tesla-y67qle`). Do not push to other branches without permission.
- Commit messages should describe the gameplay or code change in the imperative mood, with a short bullet list for multi-part changes.
- Do not open pull requests unless explicitly asked.
