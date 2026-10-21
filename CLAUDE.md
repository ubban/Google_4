# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up that runs in the browser. The whole game lives in one file, `index.html`: markup, CSS, and a single inline `<script>` wrapped in an IIFE. `README.md` is only a title line.

There is no build step, no package manager, no bundler, no test suite, and no linter. Nothing is imported from a CDN. Do not add any of these without being asked.

## Running the game

Open `index.html` directly in a browser, or serve the folder with any static server. No install step is needed.

## Repository layout

```
index.html   # The entire game: <style> (lines ~10-359), markup (~361-421), <script> (~422-end)
README.md    # Title only
```

Inside the script, the code is grouped by `// ---------- Section ----------` comments. Keep new code in the matching section and follow the same comment style.

| Section (approx. location) | Contents |
| --- | --- |
| Adaptive performance | `quality` levels 0–2, driven by `trackPerf(dt)` frame-time averages |
| Constants | `DIFFICULTIES`, `KILLS_PER_STAGE`, `THEMES` (sky palettes per stage), `ENEMY_TYPES`, `BOSSES` |
| Persistence | `localStorage` keys `skyfighter.bestScore`, `.muted`, `.difficulty`, `.reducedMotion` |
| Audio | Lazy `AudioContext` via `ensureAudio()`, synthesized `beep()` / `noiseBurst()`, named effects in `sfx` |
| HUD | `renderHearts`, `renderBombs`, `showComboPop`, `showStageBanner`, `flashHit`, `flashBomb`, `screenShake` |
| Canvas sizing | Fixed logical size `BASE_W × BASE_H` = 480 × 720, scaled to fit the window (max 1.6×) |
| Input | Keyboard, mouse (`canvas` events), touch joystick + FIRE/BOMB buttons |
| Game state | `resetGame()`, `spawnEnemy()`, `spawnBoss()`, `maybeSpawnPowerUp()`, `fireWeapon()` |
| Boss patterns | `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`, `updateBoss()` |
| Main loop | `update(dt)` → `draw()` → `loop(now)`; `dt` is clamped to 0.05 s |
| End of game | `endGame()` saves a new best score and shows the overlay |

## Game rules (for reference when changing balance)

- **Difficulty** (`DIFFICULTIES`): easy / normal / hard change spawn rate, enemy speed, enemy fire chance, starting lives and bombs, and a boss HP multiplier. The difficulty buttons are ignored while a run is active; they can be changed on the overlay or while paused.
- **Stages**: a stage ends after `KILLS_PER_STAGE` (18) kills, which triggers a boss. Bosses cycle through `BOSSES` by `(stage - 1) % BOSSES.length`. HP is `(70 + stage × 35)` × difficulty multiplier × boss multiplier.
- **Enemies**: `grunt` (1 HP), `interceptor` (fast, weaving, 1 HP), `tank` (4 HP, from stage 2).
- **Power-ups** dropped by enemies (`maybeSpawnPowerUp`): life 6%, bomb 8%, weapon 20%.
- **Weapons**: VULCAN and LASER. A weapon power-up raises the level up to 3; past level 3 it switches weapon and resets to level 1.
- **Bombs** clear enemy bullets and all enemies on screen and trigger a shake and flash. They are capped by `maxBombs`.
- **Scoring**: kills add points, multiplied by the chain multiplier (`chain`, `chainTimer`, `multiplier`).

## Conventions

- Plain JavaScript (ES2015+), no modules, no TypeScript, no framework.
- Use `const`/`let`, arrow functions for callbacks, and `camelCase` names. Constants are `UPPER_SNAKE_CASE`.
- Game coordinates are in the 480 × 720 logical space. Do not use CSS pixels inside the game logic.
- Keep new per-frame logic inside `update()` / `draw()`, not in event handlers, so pause and game-over behave correctly.
- Respect the reduced-motion setting: effects that shake, flash, or pulse must go through the existing helpers (`screenShake`, `flashHit`, `flashBomb`) or check `reducedMotion`.
- Audio must start from a user gesture. Call `ensureAudio()` before creating or playing sounds.
- Accessibility: announce important state changes through `announce(text)` (the `#srLive` live region).
- Keep the single-file structure. Do not split the script or stylesheet into separate files unless asked.

## Verifying changes

Nothing is automated, so verify manually:

1. Open `index.html` in a desktop browser and check the console for errors.
2. Play through at least one stage and a boss fight on Normal. Test pause (`P`/`Esc`), bombs (`X`/`Shift`), and game over.
3. Check that the best score persists after a reload.
4. For layout changes, check a narrow viewport (phone width) and touch controls. Use a device emulator or DevTools touch mode.
5. For performance changes, watch the frame rate in DevTools. The adaptive quality system in `trackPerf` reacts to sustained slow frames.

## Git

- Development branch: `claude/gifted-tesla-w2zvlr`.
- Use clear, descriptive commit messages in the imperative mood, for example `Add Sky Fighter: ...`.
- Do not open a pull request unless one is requested.
