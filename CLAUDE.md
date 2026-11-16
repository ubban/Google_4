# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a small, dependency-free web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs as a single HTML5 canvas page.

- `index.html` — the whole game: markup, CSS (inline `<style>`), and JavaScript (one inline `<script>` wrapped in an IIFE with `'use strict'`).
- `README.md` — one-line placeholder title.

There is no build step, package manager, test suite, or bundler. Nothing needs to be installed.

## Running and testing

- Run it by opening `index.html` in a browser. For a local server: `python3 -m http.server` from the repo root, then visit `http://localhost:8000/`.
- There are no automated tests. Verify changes by playing the game in a browser, and check both desktop (keyboard) and narrow/touch layouts.
- Keep the game working as a plain static file. Do not introduce a build step, frameworks, or npm dependencies unless the user asks for them.

## Code layout inside `index.html`

The script is organized in commented sections. Keep new code in the matching section:

1. **DOM references and constants** — element lookups, `STORAGE_*` keys, `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
2. **Adaptive performance** — `trackPerf` drops visual quality (`quality` 0/1/2) when frame times are slow.
3. **Audio** — Web Audio helpers (`ensureAudio`, `beep`, `noiseBurst`). Audio is created lazily on first user gesture.
4. **Input** — keyboard (`keys` map keyed by `e.code`), touch joystick, FIRE and BOMB buttons, and pointer handling.
5. **Game state and loop** — `resetGame`, `update(dt)`, `draw()`, and `loop(now)` driven by `requestAnimationFrame`.
6. **Weapons, bombs, bosses** — `fireWeapon`, `useBomb`, `applyPowerUp`, and the `BOSS_PATTERNS` bullet-pattern functions.

Helper functions are plain function declarations inside the IIFE. Do not add globals; expose nothing on `window` unless there is a clear reason.

## Game rules and tuning

When changing gameplay, these are the values to check first:

- **Difficulty** (`DIFFICULTIES`): `easy`, `normal` (default), `hard`. Each sets spawn rate, enemy speed, enemy shoot chance, starting `lives`, starting `bombs`, and `bossHpMul`.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills. Each stage ends in a boss fight. `THEMES` cycles the sky palette per stage.
- **Enemies**: `grunt`, `interceptor` (fast, weaving), and `tank` (4 HP, appears from stage 2). Each has a score value.
- **Bosses**: `BOSSES` entries carry colors and `hpMul`; patterns come from `BOSS_PATTERNS`.
- **Weapons**: Vulcan and Laser, each with 3 power levels. Weapon level is raised by power-ups.
- **Power-ups**: `life`, `bomb`, and `weapon`. Colors are set in the draw loop.
- **Scoring**: a kill-chain multiplier (`addScore`, `breakChain`).

## Controls

| Action | Keyboard | Mouse | Touch |
|---|---|---|---|
| Move | Arrow keys or WASD | Move pointer to steer | Left-side joystick (drag) |
| Fire | Hold Space | Hold click | Hold FIRE button |
| Bomb | X or Shift | — | BOMB button |
| Pause | P or Escape | — | Pause button (top bar) |

Fire is held down: `update()` reads `keys['Space'] || firing`, where `firing` is set by the mouse and FIRE button. Preserve these bindings. Space's default scroll behavior is suppressed on purpose (`keydown` handler).

## Persistence

The game stores settings in `localStorage` using these keys. Keep the names stable, since changing them resets players' saved data:

- `skyfighter.bestScore`
- `skyfighter.muted`
- `skyfighter.difficulty`
- `skyfighter.reducedMotion`

All `localStorage` access should tolerate it being unavailable (private mode, blocked storage). The current code reads it directly, so wrap new reads and writes in `try/catch` if you add them.

## Accessibility and UX conventions

- **Reduced motion**: screen shake, hit flash, and the low-health pulse respect `reducedMotion`. It defaults to the `prefers-reduced-motion` media query and can be toggled from the top bar. Any new motion effect should honor this flag.
- **Screen reader announcements**: use the `announce(text)` helper, which writes to the `#srLive` live region.
- **Mobile**: `touch-action: none` and `user-select: none` are set on the page to prevent scrolling and selection during play. The layout must work at phone width with no horizontal scroll. Use `env(safe-area-inset-*)` for notched devices, as the existing HUD does.
- **Resizing**: the canvas is a fixed logical size (`BASE_W` = 480, `BASE_H` = 720) scaled by `resize()`. Draw in logical coordinates; do not hard-code screen pixels.

## Style and conventions

- Plain ES2015+ JavaScript, no transpilation. Use `const` by default and `let` when reassigned.
- Two-space indentation in the existing file. Match the surrounding style.
- Use the existing section comments (`// ---------- Name ----------`) when adding a new section.
- Keep the file self-contained: CSS stays in `<style>`, JS stays in the one `<script>` block, and only inline `data:` URIs are used for assets such as the favicon.
- Prefer small, named functions over long inline blocks, matching the current code.

## Git workflow

- The default working branch for this repository's cloud sessions is `claude/gifted-tesla-t8yjpv`. Commit and push there. Do not push to other branches without permission.
- Commit messages should be short imperative summaries, with an optional body for the why.
- Do not create pull requests unless the user explicitly asks for one.
- Do not commit generated or scratch files. The repo currently tracks only `README.md`, `index.html`, and this file.

## Notes for future changes

- If the project grows beyond one page, add a build or dev-server section here and move the game out of `index.html` only when there is a clear need.
- Update this file when controls, storage keys, difficulty tuning, or the section layout of `index.html` change.
