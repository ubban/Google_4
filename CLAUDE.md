# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up written as a single self-contained HTML5 canvas page (`index.html`, ~1,670 lines). `README.md` is a placeholder and holds no project documentation.

There is no build step, no package manager, no bundler, and no test suite. Do not add one unless asked.

## Repository layout

```
index.html   # The whole game: markup, inline <style>, inline <script>
README.md    # Placeholder
CLAUDE.md    # This file
```

Inside `index.html`:

- `<style>` (roughly lines 10–360): pill-style HUD, glassmorphic overlay card, touch controls, vignette/flash effects, keyframe animations.
- `<body>` markup (roughly lines 363–421): `#gameCanvas`, `#hud`, overlay/title card, touch controls, pause/mute/motion buttons, boss bar, `#srLive` (screen-reader live region).
- `<script>` (starts around line 422): one IIFE holding all game logic.

## Running and verifying changes

- Open `index.html` directly in a browser, or serve the directory with any static server (for example `python3 -m http.server`).
- There are no automated tests. After any gameplay or UI change, verify it in a real browser, including:
  - desktop keyboard controls,
  - touch controls at a phone-sized viewport,
  - pause/resume, and the difficulty buttons.
- Use the pre-installed Playwright/Chromium setup for screenshots or scripted checks. Do not reinstall Playwright.

## Architecture notes for the game script

- **Canvas scaling:** the logical play field is `BASE_W = 480`, `BASE_H = 720`. `resize()` scales the canvas to fit the window, capped at 1.6×. Keep game coordinates in base units.
- **Persistence:** `localStorage` keys are namespaced `skyfighter.*` (`bestScore`, `muted`, `difficulty`, `reducedMotion`). Reads and writes must tolerate storage being unavailable (private mode, blocked site data). Wrap them in `try/catch` or keep the game working without them.
- **Difficulty:** the `DIFFICULTIES` table (easy / normal / hard) sets spawn rates, enemy speed and fire chance, lives, starting bombs, and a boss HP multiplier. Tune balance there, not in scattered literals.
- **Stages:** reaching `KILLS_PER_STAGE` kills triggers the stage boss warning and fight. Stages cycle through `THEMES` (sky palette and cloud/sun colors). Bosses come from `BOSSES` via `(stage - 1) % BOSSES.length`, and each entry holds a name, colors, and `hpMul`.
- **Enemies:** `ENEMY_TYPES` defines `grunt`, `interceptor` (weaving), and `tank` (multi-hit, unlocked at stage 2). Add new types there and give them a `minStage`.
- **Input:**
  - Keyboard: arrows/WASD move, Space fires, X / Shift drops a bomb, P / Escape pauses.
  - Touch: a joystick in `#moveZone`, plus `#fireBtn` and `#bombBtn`.
  - Keyboard state is tracked in a `keys` map keyed by `event.code`.
- **Performance:** a `quality` level (0 full, 1 medium, 2 low) adapts to measured frame time through `trackPerf`. New visual effects should respect it, and should also respect `reducedMotion`.
- **Audio:** Web Audio API only, created lazily in `ensureAudio()` after a user gesture. The `sfx` object holds the sound effects. Mute is persisted.
- **Accessibility:** announcements go through `announce()` into `#srLive`. Keep the reduced-motion option working when adding animation, shake, or flash effects.

## Conventions

- Keep the game in a single file unless the user asks for a split. Preserve the existing style: vanilla JS, no frameworks, no external dependencies, and no network requests at runtime.
- Use `const` by default and keep the existing 2-space indentation.
- Don't add comments that restate what the code does. Comment only non-obvious intent.
- When changing a constant in `DIFFICULTIES`, `ENEMY_TYPES`, or `BOSSES`, check the other tables for consistency, since balance is spread across them.
- Keep all user-facing text in English.

## Git workflow

- Develop on the branch the environment specifies. Do not push to other branches without permission.
- Commit only when asked or when a task is finished. Write commit messages that say why the change was made.
- Do not open a pull request unless the user explicitly asks for one.

## Things to avoid

- Don't introduce a build system, framework, or external CDN dependency without an explicit request.
- Don't store secrets or personal data in the repository or in `localStorage` keys.
- Don't delete or rename the `skyfighter.*` storage keys without a migration. That would silently reset players' best scores and settings.
