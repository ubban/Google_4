# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repo is

`Google_4` is a tiny static project containing one game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on a `<canvas>`.

- `index.html` — the entire game: markup, CSS, and one inline `<script>` (~1,650 lines). Nothing else is needed to run it.
- `README.md` — a one-line title. No other docs exist yet.

There is no build step, no package manager, no dependencies, no bundler, and no test suite. Do not add any of these unless the user asks.

## Running it

Open `index.html` in a browser, or serve the directory with any static server (for example `python3 -m http.server`). No install step is required.

## Architecture of `index.html`

Everything lives inside one IIFE-style `<script>` block. Key pieces, roughly in file order:

- **DOM references** (~line 426): all `getElementById` lookups are grouped at the top of the script.
- **Persistence**: `localStorage` keys are prefixed `skyfighter.` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Every read/write should tolerate storage being unavailable.
- **Tuning tables**:
  - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed and fire chance, starting `lives`, starting `bombs`, `bossHpMul`.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: six sky palettes, one per stage, cycling.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaves), `tank` (4 HP, appears from stage 2).
  - `BOSSES`: six bosses, each with its own colours and `hpMul`.
- **Audio**: synthesized with the Web Audio API (`beep`, `noiseBurst`, the `sfx` table). Audio context is created lazily by `ensureAudio()` on the first user gesture; keep it that way, since browsers block autoplay.
- **Input**: keyboard (arrows/WASD to move, Space or hold to fire, X/Shift to bomb, P/Esc to pause), mouse (drag/aim plus fire button), and touch (virtual joystick in `#moveZone`, `#fireBtn`, `#bombBtn`).
- **Rendering**: fixed logical resolution `BASE_W × BASE_H` (480 × 720), scaled to fit the window by `resize()` (capped at 1.6×). Draw code is in `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`.
- **Adaptive quality**: `trackPerf()` samples frame times and lowers `quality` (0 full, 1 medium, 2 low) when frames are slow.
- **Simulation**: `update(dt)` runs each frame; `loop(now)` drives it through `requestAnimationFrame`. Bosses use pattern functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, `fireHomingSeeded`, etc.) dispatched from `updateBoss(dt)`.
- **Game flow**: `resetGame()`, `loseLife()`, `killBoss()`, `endGame()`, `togglePause()`, `applyPowerUp()`, `useBomb()`.
- **Accessibility**: an `aria-live` region (`srLive`, via `announce()`), a reduced-motion mode (honours `prefers-reduced-motion` and a user toggle), and a mute toggle.

## Conventions

- **Single file.** Keep the game in `index.html` unless the user asks for a split. Don't introduce a framework or external CDN dependency; the file currently loads nothing external.
- **Keep tuning in the tables.** Balance changes (speeds, spawn rates, HP, scores) belong in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, and similar constants, not scattered as magic numbers through the logic.
- **Preserve mobile behaviour.** The page uses `touch-action: none`, safe-area insets, and a phone-sized viewport. Test layout changes at narrow widths.
- **Respect reduced motion.** New animations or screen-shake effects should check the reduced-motion flag.
- **Keep audio user-gated.** Don't start audio contexts or playback before a user gesture.
- **Storage access must be defensive.** Wrap `localStorage` calls so the game still runs when storage is blocked.

## Verifying changes

There are no automated tests or linters configured. To verify a change:

1. Open `index.html` in a browser (or a headless Chromium) and confirm the page loads with no console errors.
2. Start a game, then exercise the path you changed: movement, firing, a bomb, a stage transition, a boss fight, game over, and the restart.
3. For layout changes, check both a desktop viewport and a phone-sized viewport.
4. Run a quick syntax check on the inline script if you have Node available, for example by extracting the `<script>` body and running `node --check`.

If you cannot exercise the change in a browser, say so explicitly rather than claiming it works.

## Git

- Development branch for this work: `claude/gifted-tesla-lqrjd0`. Do not push to other branches without explicit permission.
- Do not open pull requests unless the user asks for one.
- Keep commits focused, with a short imperative subject line.
