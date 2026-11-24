# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

`Google_4` is a tiny static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a browser canvas.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>`. Roughly 1,670 lines.
- `README.md` — a stub with only the project title.
- No build system, package manager, bundler, tests, or external assets. Audio is synthesized with WebAudio, and the favicon is an inline SVG data URI.

## Working with this repo

- **Run it:** open `index.html` directly in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`). There is nothing to install or compile.
- **Verify changes:** there is no test suite or linter. Check gameplay changes by loading the page in a browser and playing. Check layout changes at phone width as well, since the game supports touch.
- **Keep it a single file.** The project is deliberately dependency-free. Do not add a build step, framework, or CDN script unless the user asks for it.

## Architecture of `index.html`

The script is plain global JavaScript with no modules. Key pieces, in source order:

1. **DOM references and HUD** — elements such as `stageBanner`, `bossBarWrap`, hearts, bombs, and stat pills.
2. **Persistence** — `localStorage` keys prefixed `skyfighter.`: `bestScore`, `muted`, `difficulty`, `reducedMotion`. Every read and write should tolerate storage being unavailable.
3. **Tuning tables** — these are the main knobs for balance:
   - `DIFFICULTIES` (`easy` / `normal` / `hard`): spawn rate, enemy speed, shot chance, lives, bombs, and boss HP multiplier.
   - `KILLS_PER_STAGE` — kills needed before a boss spawns.
   - `THEMES` — six sky palettes, cycled by stage.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), and `tank` (4 HP, appears from stage 2).
   - `BOSSES` — six boss designs, cycled by stage. Boss HP is `(70 + stage × 35) × difficulty × design multiplier`.
4. **Audio** — `ensureAudio()` creates the AudioContext lazily, so it must run after a user gesture. `sfx` holds the sound effects. Respect the `muted` flag.
5. **Input** — keyboard, touch joystick (`moveZone`), touch and mouse fire button, and mouse aim on the canvas.
6. **Game state and loop** — `resetGame()` initializes state. `update(dt)` runs each frame and handles spawning, collisions, power-ups, bosses, and stage progression. Drawing functions (`drawPlane`, `drawBossShip`, and others) follow. An adaptive quality level (`quality`, from `trackPerf`) lowers effects when frame time is poor.

### Controls

| Action | Keyboard | Touch |
|---|---|---|
| Move | Arrow keys or WASD | Left joystick |
| Fire (hold) | Space | Fire button |
| Smart bomb | X or Shift | Bomb button |
| Pause | P or Esc | — |

## Conventions

- Keep all CSS and JavaScript inline in `index.html` unless the user asks for a split.
- Match the existing style: 2-space indentation, `const` and `let`, single quotes in JavaScript, and section comments such as `// ---------- Audio ... ----------`.
- Use the constants above when changing difficulty or progression instead of hard-coding numbers in logic.
- Make sure new HUD or overlay elements work at phone width and don't block touch controls (`pointer-events: none` is already used on HUD elements).
- Respect the reduced-motion setting when adding animation, and keep the mute setting honored for any new sound.

## Git

- Develop on the branch the session was started on, and push with `git push -u origin <branch>`.
- Do not open a pull request unless the user asks for one.
- Do not commit secrets or local settings files.
