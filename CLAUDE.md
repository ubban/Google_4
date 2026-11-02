# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up rendered on an HTML5 `<canvas>`.

| File | Purpose |
| --- | --- |
| `index.html` | The entire game: markup, inline CSS (`<style>`, ~lines 10–420), and one IIFE containing all game logic (`<script>`, ~line 422 onward). |
| `README.md` | Placeholder title only. |

There is no build step, no package manager, no bundler, no test suite, and no linter configured. Do not add any of these unless the user asks.

## Running the game

Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`). Nothing needs to be installed.

## Architecture of `index.html`

- **Single file.** Keep CSS, markup, and JavaScript in `index.html` unless the user explicitly asks for a split. Don't introduce external CDN scripts or fonts.
- **One IIFE.** All game state lives inside `(function () { ... })()`. Avoid leaking globals.
- **Fixed logical resolution.** `BASE_W = 480`, `BASE_H = 720`. The `resize()` function scales the canvas to fit the window (capped at 1.6x). Game coordinates always use the logical size, not the CSS size.
- **Fixed-timestep-ish loop.** `loop(now)` uses `requestAnimationFrame`, clamps `dt` to 0.05 s, then calls `update(dt)` and `draw()` while running and not paused.
- **Adaptive quality.** `trackPerf()` samples frame times over 45 frames and steps `quality` (0 full, 1 medium, 2 low) down or up. Visual costs such as `shadowBlur` and particle counts depend on `quality`. Preserve this when adding effects.
- **Tunable data lives in constant tables** near the top of the script. Prefer editing these over hard-coding numbers in logic:
  - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rates, enemy speed, enemy fire chance, starting lives and bombs, boss HP multiplier.
  - `KILLS_PER_STAGE` (18): kills needed to advance a stage.
  - `THEMES`: per-stage sky palettes, cloud and sun colors, and names. Stage index wraps.
  - `ENEMY_TYPES`: `grunt`, `interceptor` (weaving), `tank` (multi-HP, unlocks at stage 2).
  - `BOSSES`: six boss designs, cycled by `(stage - 1) % BOSSES.length`. Boss HP scales with stage.
- **Persistence** uses `localStorage` with keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every access to `localStorage` should be tolerant of it being unavailable (private mode, blocked storage). The current code does not wrap these calls in `try/catch`; if you touch that code, consider adding it.
- **Audio** is synthesized with the Web Audio API (`ensureAudio()`, `beep()`, `noiseBurst()`, the `sfx` table). Audio is created lazily after a user gesture. Don't add audio files.
- **Accessibility.** There is a live region (`srLive`, via `announce()`) for screen readers, a reduced-motion toggle that respects `prefers-reduced-motion`, and a reduced-motion toggle (`reducedMotion`) that scales down screen shake and skips hit/bomb flashes. New screen-shake or flash effects should go through `screenShake()` / `flashHit()` / `flashBomb()` so they respect it.

### Input

- Keyboard: arrow keys or WASD to move (via `keys[e.code]`), `Space` to fire, `X` / `Shift` to drop a bomb, `P` / `Escape` to pause.
- Mouse and touch: pointer movement steers the ship; touch uses an on-screen joystick (`moveZone`), a fire button, and a bomb button. Touch devices are detected with `isTouchDevice`.
- Input handlers are attached to `window` (keyboard) and to the canvas and touch controls (pointer). Keep new controls consistent with this split.

### Gameplay systems

- **Weapons:** Vulcan and Laser, each with power levels 1–3 that change spread and lanes (`fireWeapon()`).
- **Bombs:** limited per run; `useBomb()` clears the screen and damages bosses.
- **Power-ups:** `life`, `bomb`, and `weapon` (`maybeSpawnPowerUp()`).
- **Combo chain:** `addScore()` builds a kill-chain multiplier; `breakChain()` resets it when the player takes damage.
- **Bosses:** appear at the end of each stage, have a health bar (`bossBarWrap`), and telegraph attacks (`b.telegraph`).

## Conventions

- Match the existing style: two-space indentation, single quotes in JavaScript, `const`/`let` (no `var`), small named helper functions, and comments only where the intent isn't obvious.
- Keep the game dependency-free and runnable by opening the file. Don't add npm, a build step, or frameworks without an explicit request.
- Keep HUD and overlay changes responsive. The layout must work on small phone screens (safe-area insets and `clamp()` sizing are already used).
- Preserve the `quality` checks when adding particles, glows, or shadows.
- Don't remove or rename the `skyfighter.*` storage keys without a migration plan, since they hold players' saved best score and settings.

## Making changes

1. Read the relevant section of `index.html` before editing. The file is large (~1,700 lines), so search with `grep` for the function or constant you need.
2. Make the smallest change that satisfies the request.
3. Verify by loading the page in a browser (or with the Playwright setup available in the environment) and playing briefly: start a run, fire, use a bomb, pause and resume, and check the console for errors.
4. Commit with a clear message describing the gameplay or code change. Push only to the branch the task designates.

## Things to avoid

- Don't add test, lint, or build tooling, or new top-level directories, unless asked.
- Don't commit generated files or editor artifacts.
- Don't change the logical resolution (`BASE_W` / `BASE_H`) or the difficulty tuning as a side effect of another change.
