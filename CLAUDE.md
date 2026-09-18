# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file web application. The entire game (HTML, CSS, JS, audio synthesis) is contained in `index.html` with no external dependencies or build process.

## Running the Game

- **Browser**: Open `index.html` directly in any modern browser (desktop or mobile)
- **Local server** (optional): Serve via `python -m http.server` or `npx serve` to avoid CORS issues if testing features that might need it
- **No dependencies**: The game is self-contained; nothing to install or build

## Architecture Overview

The game is structured as a single-file IIFE (Immediately Invoked Function Expression) for encapsulation. All state and logic lives within one scope.

### Core Subsystems

**Game Loop** (line 1640–1652)
- `requestAnimationFrame` drives the loop at 60fps target
- Delta-time based updates for frame-rate independence
- Adaptive performance tracking adjusts rendering quality based on frame times

**Player Ship** (line 876–877, 1124–1146)
- Position, weapon state, health, invulnerability timer
- Dual input: keyboard/gamepad (WASD/arrows) + mouse steering (gliding toward pointer) + touch joystick
- Two weapon types: Vulcan (spread fire, 3 levels) and Laser (piercing, 3 levels, higher fire rate)
- Weapon switching via power-ups; levels affect bullet patterns and damage

**Enemy System** (line 1022–1040, 1210–1227)
- Three enemy types (grunt, interceptor, tank) with min-stage unlock gates
- Spawn rate decreases (difficulty ramps) as stage progresses
- AI: two fire patterns (spread vs. aimed) plus weaving movement for interceptors
- Enemy despawn when off-bottom or destroyed

**Boss System** (line 1043–1055, 1411–1438)
- Six boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) cycle per stage, each with unique color and HP multiplier
- Attack patterns defined in `BOSS_PATTERNS` (line 1372–1409): each boss has 3 distinct patterns that rotate
- Pattern functions emit bullet formations: radial, aimed spreads, columns, homing, spirals, teleport dashes
- Boss telegraphs before attacking (visual/audio cue) to signal incoming attack
- Health bar tracks damage in real-time

**Collision Detection** (line 1069–1071, 1232–1295)
- AABB rect overlap for all collisions (bullets ↔ enemies, bullets ↔ boss, enemies ↔ player, bullets ↔ player, power-ups ↔ player)
- Player invulnerability frames after hit prevent rapid damage stacking

**Difficulty System** (line 485–489)
- Three presets: easy, normal, hard with tuned spawn rates, enemy speed, shoot chance, HP, and boss multipliers
- Persisted to `localStorage` for resume on reload

**Visual Effects**
- Parallax scrolling clouds at two speeds/opacities per theme
- Screen shake on hits and bomb usage
- Flash overlay (white) on damage and bomb detonation
- Particles (small circles) on all explosions with trajectory and fade
- Theme system: six procedurally-selected sky gradients + cloud/sun color palettes cycle per stage
- Boss core glow intensifies during telegraph

**Audio** (line 542–676)
- WebAudio synthesis, no external files
- Dynamicscompressor + master gain for mix control
- `beep()` for pitched sounds (parametric sweep, filter, pan)
- `noiseBurst()` for impact/explosion texture
- Context is lazy-initialized on first user input (Web Audio policy)
- Muted state persists to localStorage

**UI & UX**
- Overlay card system: start menu, pause menu, game-over screen
- HUD: score, best score, multiplier (combo), weapon name/level, hearts, bombs, boss name/health bar
- Screen-reader live region for announcements
- Reduced-motion preference respected; animations toggled on/off via motion button
- Responsive canvas scaling: maintains 480×720 game resolution, scales to viewport with max 1.6× zoom

**Input Handling** (line 768–861)
- Keyboard: arrow keys or WASD for movement, Space/click to fire, X/Shift for bomb, P/Esc to pause
- Mouse: aim by moving pointer (player glides toward target when no keyboard input active); click to fire
- Touch: left 55% of screen is movement joystick (visual base + stick), right 45% has FIRE and BOMB buttons; drag anywhere on left to steer
- Mouse and keyboard are prioritized over each other intelligently (latest input wins, never conflict)

**Scoring & Chain System** (line 1087–1101)
- Base score per kill type (grunt=10, interceptor=15, tank=35, boss=500, bomb=5 per kill)
- Kill chain: multiplier increases 1.1× every 5 consecutive kills
- Chain resets if player takes damage or times out (2.2s idle)
- Combo popup shows milestone (every 10 kills)

**Progression** (line 491–515, 1449–1453)
- Stages: kill 18 enemies to trigger boss warning, then boss spawns
- Stage 2+ unlocks tank enemies
- Six thematic sky variants cycle; boss designs also cycle
- No hard stage cap; can progress infinitely

**Power-ups** (line 1058–1066, 1304–1322)
- 6% drop: extra life (cap at max)
- 8% drop: extra bomb (cap at max)
- 20% drop: weapon level +1, or if at level 3, toggle weapon type and reset to level 1
- Spawn on enemy kill; fall down screen at 90 pixels/sec

**State Management**
- Game runs in one of three states: running (play), paused, game-over
- On game-over: score persists to best-score in localStorage; overlay shows stats + difficulty picker
- Pause preserves all state; resume continues from exact frame

## Performance Optimizations

- **Adaptive quality** (line 474–482): tracks frame times; downgrades from full quality (0) → medium (1, 60% particles) → low (2, 35% particles) if FPS drops below 40
- **Glow effects** conditional on quality > 0 (shadows disabled at full quality)
- **Particle pooling**: particles are simple objects reused from array, filtered out when dead
- **Collision batching**: nested loops with early break on first hit per bullet
- **Canvas resize**: only on window resize/orientation events, not every frame

## Modifications & Extensibility

**To adjust difficulty tuning:**
Modify `DIFFICULTIES` object (line 485–489): `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.

**To add new enemy type:**
Add entry to `ENEMY_TYPES` (line 502–506), set `minStage` to gate when it appears, implement spawn logic in `spawnEnemy()`, and draw via `drawPlane()` call.

**To add new boss pattern:**
Add a new entry to `BOSS_PATTERNS` (line 1372–1409) with three pattern functions. Each function emits bullets via helpers (`fireRadial`, `fireAimedSpread`, etc.) or direct enemy bullet creation.

**To change controls/keybinds:**
Modify keyboard event listeners (line 769–775) for movement keys and action keys. Touch zones are defined by `#moveZone` and `#actionZone` CSS (line 299–325).

**To add new theme:**
Push object to `THEMES` array (line 493–500) with sky gradient stops, cloud/sun color strings, and name. Themes auto-cycle per stage.

## Testing / Debugging

- Open browser DevTools (F12) and check Console for any JS errors
- Inspect game state: open DevTools → Console, type `player`, `boss`, `score`, etc. to inspect real-time values
- Toggle audio mute button (🔊) to verify Web Audio is working
- Test all three difficulties and all six bosses (requires reaching stage 2–7)
- Verify touch controls on mobile by tapping movement zone and action buttons
- Reduced-motion toggle (🎆 button) should disable all animations when on
- High-score persistence: reload page and verify best-score persists and difficulty is remembered

## Key Implementation Notes

- **Lag/low FPS behavior**: Adaptive quality scales down visual fidelity (particle count, shadows); game physics remain smooth due to delta-time based updates
- **Boss attack patterns are deterministic** per stage, but bullet timing and aim are frame-dependent, so each run differs
- **Bomb clears all on-screen enemy bullets** but does not prevent new boss patterns; strategic timing is key
- **Invulnerability frames (1.5s after hit) prevent stun-locking** but player can still move and fire
- **Weapon persistence across stages**: picking up weapon power-up upgrades your current weapon or swaps to the other type
- **Canvas resolution**: fixed 480×720 internal, then CSS scales to fit viewport with max 1.6× magnification to keep responsive on large displays

## Storage Keys

All persisted to `localStorage`:
- `skyfighter.bestScore` – high score
- `skyfighter.muted` – audio mute toggle
- `skyfighter.difficulty` – last selected difficulty
- `skyfighter.reducedMotion` – user motion preference override
