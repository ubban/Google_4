# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a static, single-file browser game called **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

- The entire game lives in `index.html`: HTML markup, a `<style>` block, and one inline `<script>` that drives an HTML5 `<canvas>`.
- There is no build step, no bundler, no package manager, no framework, and no external dependencies.
- `README.md` is a stub and contains no run instructions. The game is played by opening `index.html` in a browser.
- There is no test suite, linter, or CI configuration in the repository.

## Repository layout

```
index.html   # The whole game: markup, CSS, and JS (~1670 lines)
README.md    # Minimal project title
CLAUDE.md    # This file
```

## Running the game

No install or build is needed. Either:

- Open `index.html` directly in a modern browser, or
- Serve the directory with any static server, for example `python3 -m http.server` from the repo root, then visit `http://localhost:8000`.

## Code structure (inside `index.html`)

The inline script is organized as plain top-level functions and constants, with no modules.

| Area | Key identifiers | Notes |
|---|---|---|
| Persistence | `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED` | `localStorage` keys under the `skyfighter.*` prefix. Read and written directly, with no try/catch. |
| Difficulty | `DIFFICULTIES`, `setDifficulty()` | `easy` / `normal` / `hard`. Sets spawn rate, enemy speed, enemy fire chance, lives, bombs, and boss HP multiplier. |
| Stages | `KILLS_PER_STAGE` (18), `THEMES`, `BOSSES`, `spawnBoss()` | Each stage ends with a boss. Themes and bosses cycle with `(stage - 1) % length`. |
| Enemies | `ENEMY_TYPES`, `spawnEnemy()` | `grunt`, `interceptor` (weaving), and `tank` (4 HP, unlocks at stage 2). |
| Bosses | `BOSS_PATTERNS`, `updateBoss()`, `killBoss()` | Each boss has a list of attack-pattern functions (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireCross`, `fireHomingSeeded`, and others). |
| Player weapons | `fireWeapon()`, `applyPowerUp()` | `vulcan` (spread shot) and `laser` (piercing). Weapon power-ups raise level 1 to 3, then switch weapon type. |
| Power-ups | `maybeSpawnPowerUp()`, `applyPowerUp()` | `life`, `bomb`, and `weapon`. |
| Bombs | `useBomb()` | Clears enemy bullets and triggers screen shake. |
| Damage | `loseLife()` | Grants 1.5 s of invulnerability and breaks the combo chain. |
| Scoring | `addScore()`, `breakChain()` | Combo chain multiplier. |
| Audio | `ensureAudio()`, `beep()`, `noiseBurst()`, `sfx.*` | Procedural Web Audio API sound. No audio files. `ensureAudio()` must run from a user gesture. |
| Input | `keys`, `joystickStart()`, `canvasPointFromEvent()`, touch and mouse listeners | Keyboard, mouse, and touch. See Controls below. |
| Layout | `BASE_W = 480`, `BASE_H = 720`, `resize()` | Canvas scales to fit the viewport while keeping its base aspect ratio. |
| Main loop | `update(dt)`, `draw()`, `loop(now)` | Driven by `requestAnimationFrame`. `dt` is clamped to 0.05 s. `trackPerf()` averages the last 45 frames and adjusts a `quality` level (0 to 2) up or down to keep frame time in range. |
| Accessibility | `announce()`, `srLive`, `reducedMotion`, `updateMotionBtn()` | Screen-reader live region announcements. A reduced-motion toggle that also respects `prefers-reduced-motion`. |

## Controls

| Action | Keyboard | Touch / mouse |
|---|---|---|
| Move | `WASD` or arrow keys | On-screen virtual joystick |
| Fire | `Space` (hold) | `FIRE` button (hold) |
| Smart bomb | `X` or `Shift` | `BOMB` button |
| Pause | `P` or `Esc` | `⏸` button (`#pauseBtn`) |

## Conventions

- **Keep it a single file.** Do not split `index.html` into separate modules or add a build step unless the user asks. The no-dependency, open-and-play design is intentional.
- **Plain JavaScript only.** Match the existing style: `const` and `let`, arrow functions, template or string concatenation, and no transpilation.
- **Game tuning lives in constants.** Prefer editing `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `THEMES`, and `BOSS_PATTERNS` over hard-coding numbers in the loop.
- **Preserve the storage keys.** Renaming any `skyfighter.*` key will silently reset players' saved best score, mute, difficulty, and motion preferences.
- **Preserve accessibility hooks.** Keep the `announce()` calls and the `srLive` region when changing game-state messages. Respect `reducedMotion` for new screen-shake or flash effects.
- **Keep it responsive.** The UI must work at phone width. Touch controls already call `preventDefault()` and set `touch-action: none`; keep those when adding new interactive elements.
- **Audio must start from a user gesture.** Call `ensureAudio()` inside a click, touch, or key handler, never on page load.

## Verification

There are no automated tests. To verify a change:

1. Open `index.html` in a browser, or serve it locally as described above.
2. Play through at least one full stage, including the boss, on the affected difficulty.
3. Check both keyboard and touch input when touching input code.
4. Check the layout at a phone-sized viewport when touching CSS or the HUD.
5. Check the browser console for errors.

## Git

- Default development branch for this work: `claude/gifted-tesla-8mg93s`.
- Commit messages are short imperative summaries, such as `Add Sky Fighter: Raiden-style vertical shoot-em-up game`.
