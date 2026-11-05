# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

**Google_4** is a small, dependency-free browser game: **Sky Fighter**, a Raiden-style vertical shoot-'em-up.

The entire game lives in a single file, `index.html`, which contains inline `<style>` and a single inline `<script>` (no build step, no bundler, no external assets). Audio is synthesized with the WebAudio API. `README.md` is a placeholder and contains no setup instructions.

Repository layout:

- `index.html` — the whole game (markup, CSS, and JS, ~1,670 lines)
- `README.md` — minimal placeholder
- `CLAUDE.md` — this file

## Running and testing

- There is no package manager, build, lint, or test tooling. Do not invent npm/build commands.
- To run locally, open `index.html` in a browser, or serve the directory with any static server (e.g. `python3 -m http.server`).
- Verify changes by loading the page in a browser and playing. For gameplay changes, check both desktop (keyboard/mouse) and touch controls, since the game has separate input paths for each.

## Code structure inside `index.html`

The script is one IIFE-style block. Major sections, in roughly file order:

1. **DOM references** — `getElementById` lookups for HUD, overlays, and touch controls.
2. **Storage keys** — `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`.
3. **Adaptive performance** — `trackPerf` lowers render `quality` (0 full / 1 medium / 2 low) when frame times are poor.
4. **Config tables**
   - `DIFFICULTIES` (easy / normal / hard): spawn rates, enemy speed and fire chance, starting lives and bombs, boss HP multiplier.
   - `KILLS_PER_STAGE` (18) — kills needed before a boss spawns.
   - `THEMES` — six sky palettes, cycled by `(stage - 1) % THEMES.length`.
   - `ENEMY_TYPES` — `grunt`, `interceptor` (weaving), `tank` (4 HP, appears from stage 2).
   - `BOSSES` — six boss designs, cycled by `(stage - 1) % BOSSES.length`. Boss HP is `(70 + stage * 35)` scaled by the boss's `hpMul` and the difficulty's `bossHpMul`.
5. **Persistent settings** — best score, mute, difficulty, and reduced-motion, all read from and written to `localStorage`.
6. **Audio** — `ensureAudio`, `beep`, `noiseBurst`, and the `sfx` table. Audio is created lazily on first user gesture.
7. **Canvas sizing** — fixed logical resolution `BASE_W × BASE_H` = 480 × 720, scaled to fit the viewport (capped at 1.6×).
8. **Input** — keyboard (`keys[e.code]`), a touch joystick plus fire/bomb buttons, and mouse steering toward the pointer. Keyboard and joystick input take priority over mouse steering.
9. **Game state and loop** — `resetGame`, `useBomb`, `killBoss`, `loseLife`, power-ups (`maybeSpawnPowerUp`, `applyPowerUp`: life, bomb, weapon), and the per-frame update/render.

Controls (keyboard): arrows/WASD to move (via `keys`), Space to fire, `X` or Shift for a smart bomb, `P` or Esc to pause.

## Conventions

- Keep everything in `index.html` unless there is a clear reason to split files. Adding a separate asset or module is a larger architectural change and should be discussed first.
- No external dependencies or CDN scripts. Keep the game working offline from a single file.
- Game balance lives in the config tables above. Prefer tuning those constants over adding special-case branches in the loop.
- Persist user settings only through the existing `STORAGE_*` keys. Wrap `localStorage` access so the game still runs when storage is unavailable (private mode, blocked site data).
- Respect `reducedMotion` for new screen-shake, flash, or particle effects.
- Keep the layout usable at phone width: the canvas scales to the viewport and the touch controls must not overlap the play area in a way that blocks play.
- Match the existing style: 2-space indentation, single quotes, semicolons, and short section comments in the `// ---------- Name ----------` form.

## Git workflow

- Develop on the branch designated for the task (currently `claude/gifted-tesla-9hffde`). Do not push to other branches without explicit permission.
- Commit with clear, descriptive messages. Push with `git push -u origin <branch>`.
- Do not open a pull request unless explicitly asked.
