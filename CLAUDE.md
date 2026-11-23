# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project overview

Google_4 is a small, dependency-free static web project. Its only real content is **Sky Fighter**, a Raiden-style vertical shoot-'em-up that runs in a single HTML file on a `<canvas>`.

- `index.html` — the entire game: inline `<style>`, markup, and one inline `<script>` (~1,670 lines).
- `README.md` — a one-line title.
- No build system, package manager, tests, or CI config exist yet.

## Running the project

No build step is needed. Open `index.html` directly in a browser, or serve the folder with any static server, for example:

```bash
python3 -m http.server 8000   # then visit http://localhost:8000/
```

## Code layout inside `index.html`

The script is plain browser JavaScript (no modules, no frameworks). Key sections, roughly in file order:

- **Config constants** (~line 463–530): `STORAGE_*` localStorage keys, `DIFFICULTIES` (spawn rates, enemy speed, lives, bombs, boss HP multiplier), `KILLS_PER_STAGE`, `THEMES`, `ENEMY_TYPES`, `BOSSES`.
- **Audio** (~line 545–680): lazily created `AudioContext` (`ensureAudio`), with `beep` and `noiseBurst` helpers for synthesized sound effects.
- **Layout and input** (~line 753–870): `resize()` scales a fixed `480×720` logical canvas (`BASE_W`/`BASE_H`) to the viewport; keyboard state lives in `keys` (keyed by `e.code`); touch joystick and fire/bomb buttons are wired here.
- **Game state and logic** (~line 875–1500): `resetGame`, `spawnEnemy`, `spawnBoss`, power-ups, `fireWeapon`, `useBomb`, `update(dt)`, `BOSS_PATTERNS` (bullet-pattern generators), `loseLife`, `endGame`, `togglePause`.
- **Rendering** (~line 913–1640): `drawPlane`, `drawBossShip`, `draw()`.
- **Main loop**: `loop(now)` drives `update` and `draw` each frame.

Persisted settings (all in `localStorage`, read defensively at load): best score, mute, difficulty, and reduced motion.

## Controls (for reference when testing)

- Move: arrow keys or WASD
- Fire: hold Space (or the on-screen fire button on touch devices)
- Bomb: X or Shift (or the on-screen bomb button)
- Pause: P or Escape

## Conventions

- Keep the game self-contained in `index.html` unless there is a clear reason to split it. If it is split, update this file's layout section.
- Do not add external CDN scripts or libraries without asking; the game currently has zero dependencies.
- Keep the fixed logical resolution (`BASE_W`/`BASE_H`) in mind when adding positions or sizes; rendering and collision are in that coordinate space.
- Preserve mobile support: touch controls, `touch-action: none`, safe-area insets, and the phone-width layout.
- Preserve accessibility hooks that already exist (the `announce` live region, reduced-motion handling, and the mute toggle).
- Guard `localStorage` and `AudioContext` access, since both can be unavailable or throw in private or restricted browsing modes.

## Verification

There is no automated test suite. To verify a change:

1. Load `index.html` in a browser and start a game.
2. Exercise the path you changed (movement, firing, bombs, a boss stage, pause, game over).
3. Check the layout at a phone-sized viewport as well as desktop.

## Git

- Development branch for this session: `claude/gifted-tesla-c4xrj5`.
- Commit messages should be short and describe the gameplay or code change.
