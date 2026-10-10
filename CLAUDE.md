# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game (Raiden-style) built entirely in a single HTML file with embedded JavaScript and CSS. The game features:
- Canvas-based 2D rendering at 480×720 resolution
- Web Audio API synthesized sound effects (no external audio files)
- Multiple enemy types with distinct behaviors
- Boss encounters with unique attack patterns
- Progressive difficulty ramping through stages
- Touch and keyboard controls with mouse steering
- Score/combo tracking with localStorage persistence
- Multiple difficulty levels

The game is self-contained in `index.html` (1669 lines) with no external dependencies.

## Architecture Overview

The codebase follows a classic game loop pattern organized into functional sections:

### Core Structure (in source order)
1. **DOM Setup & Canvas** (lines ~400-450): Canvas initialization, HUD elements
2. **Game Constants** (lines ~450-520): Enemy types, bosses, themes, difficulty settings, storage keys
3. **Audio System** (lines ~542-676): Web Audio synth, sound effect definitions
4. **State Initialization** (lines ~517-535): localStorage-based persistence for best score, mute, difficulty, reduced motion
5. **Canvas Sizing** (lines ~752-765): Responsive scaling with BASE_W=480, BASE_H=720
6. **Input Handling** (lines ~767-850): Keyboard, mouse, and touch (joystick) controls
7. **Game Update Loop** (lines ~1100-1302): Enemy spawning, collision detection, bullet/enemy updates
8. **Rendering** (lines ~1520+): Canvas draw calls with theme-based visuals

### Key Game Objects

**Player** (`player`): Position (x, y), dimensions, weapon type, level, invulnerability timer, health state

**Enemies** (`enemies[]`): Type-specific properties (grunt/interceptor/tank), health, position, speed, shoot patterns

**Boss** (`boss`): Design template, position, health, attack telegraph timer, pattern rotation, entrance animation

**Bullets** (`bullets[]`): Player projectiles with laser/vulcan type distinction, pierce count, hot particles on hit

**Enemy Bullets** (`enemyBullets[]`): Opponent projectiles (radial, spread, homing), collision checks trigger player damage

**PowerUps** (`powerUps[]`): Spawned on enemy death; types: life, bomb, weapon (affects player.weapon/level)

**Particles** (`particles[]`): Transient explosions and visual effects with velocity and lifetime decay

## Input Flow

**Keyboard/Gamepad Logic** (lines ~768-776):
- `Space`: Fire (hold)
- `KeyP` / `Escape`: Toggle pause
- `KeyX` / `Shift`: Use bomb
- Arrow keys / WASD: Movement (checked via `keys` object each frame)

**Touch Input** (lines ~814-834):
- **Left zone** (`moveZone`): Virtual joystick with magnitude/angle (max radius 50px)
- **Right zone** (`fireBtn`): Fire button
- **Bomb button** (`bombBtn`): Dedicated bomb trigger

**Mouse Steering** (lines ~836-850): `mouseTarget` glides toward pointer; only active when mouse over canvas. Keyboard/joystick input takes priority.

The input handler calls `ensureAudio()` on first interaction to unlock Web Audio context (required by iOS).

## Update & Collision System

**Main Update Loop** (`gameLoop()` at ~1100 lines):
- Spawn enemies based on difficulty ramp (decreasing spawn interval every 300 frames)
- Update enemy positions, apply weave sine oscillation for interceptors
- Fire enemy bullets based on patterns (spread, aimed, radial, homing)
- Update player position from input (keyboard/joystick takes priority over mouse)
- Rotate camera target toward mouse if active
- Remove off-screen bullets and enemies
- **Collision checks** (triple-nested loops):
  - Player bullets vs enemies: apply damage, spawn particles, score, remove/pierce bullet
  - Player bullets vs boss: apply 3-damage, spark particles, play SFX
  - Enemies vs player: trigger loseLife()
  - Enemy bullets vs player: trigger loseLife()
  - Boss vs player: trigger loseLife()
  - PowerUps vs player: apply effect (weapon/bomb/life)

**Invulnerability**: After losing life, `player.invuln = 1.5` (seconds) blocks further damage; player blinks during this window.

## Rendering Pipeline

**Draw Pass** (lines ~1520+):
1. **Screen shake**: Apply random translation if `shakeTime > 0` (decays each frame)
2. **Background**: Theme-based sky gradient with sun halo and scrolling clouds at two depth layers
3. **Player plane**: Draw from `drawPlane()` helper (triangular shape) or skip if invuln-blinking
4. **Mouse cursor** (if active): Crosshair overlay
5. **Bullets**: Yellow (vulcan) with shadow glow, or cyan (laser) with blue glow
6. **Enemy bullets**: Red circles
7. **Enemies**: Per-type custom draw (see `drawEnemy()` at ~1600)
8. **Boss**: Large colorful shape with health bar
9. **Particles**: Decay opacity and remove when life ≤ 0
10. **HUD overlays**: Boss name/health bar, stage banner, combo popups

Theme colors are stored in `THEMES` (array of sky, sun, cloud RGB tuples) and cycle per stage.

## Enemy & Boss Systems

### Enemies
Three types defined in `ENEMY_TYPES`:
- **grunt** (hp=1, score=10): Basic red square, fires aimed shots
- **interceptor** (hp=1, score=15): Fast, orange, spread-pattern shots, weaves horizontally
- **tank** (hp=4, score=35): Large purple, slow, fires aimed shots

Spawn rate scales with difficulty; minStage caps lowest stages for each type.

### Bosses
Six designs in `BOSSES[]` (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with:
- Custom color schemes (hullTop/hullBottom/wing/core)
- Health multiplier (`hpMul`)
- Unique attack pattern arrays in `BOSS_PATTERNS` (lines ~1360+)

Boss spawn triggers when `stageKills >= KILLS_PER_STAGE` (line ~1204). Pattern rotation cycles through design's attack set; telegraph phase (22 frames) precedes each attack burst (85 frames). Boss auto-triggers on stage progression.

## Difficulty & Progression

Three difficulty presets stored in `DIFFICULTY`:
- **easy**: Lower health, slower spawn, higher score multiplier
- **normal**: Baseline
- **hard**: Higher health, faster spawn, lower score multiplier

Spawn timer ramps down: `spawnBase - (frame / 300) * 4` per difficulty.

Stage progression: Each stage spawns `KILLS_PER_STAGE` enemies, then a boss. Defeating boss increments stage, resets stageKills counter, shows "STAGE N" banner.

## Audio (Web Audio API)

**Synth-based SFX** (no external files):
- `beep(freq, duration, type, volume, opts)`: Oscillator with optional frequency sweep, filter, pan
- `noiseBurst(duration, volume, opts)`: Procedural noise with envelope for explosions
- `sfx` object: 12 pre-composed effects (shoot, laser, explode variants, hit, bomb, powerup, boss sounds, etc.)

All audio routed through `masterGain` → `compressor` → destination. Mute toggle disables all playback.

## State & Storage

**localStorage Keys** (lines ~480-490):
- `'best'`: Best score (integer)
- `'mute'`: Mute flag ('0' or '1')
- `'diff'`: Difficulty name ('easy'/'normal'/'hard')
- `'reduced'`: Reduced motion flag

**Game State Variables**:
- `running`: Actively playing (not paused/game-over)
- `gameOver`: Game-over overlay shown
- `paused`: Pause overlay shown
- `frame`: Global frame counter (incremented each tick)
- `score`, `kills`, `stageKills`: Scoring state
- `lives`, `bombCount`: Player resources
- `stage`: Current stage number (0-indexed)

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant (copy existing, adjust w/h/hp/speed/score/minStage/pattern if needed)
2. Add `drawEnemy()` case for the type (around line ~1620)
3. If new shoot pattern, add cases to enemy update logic (line ~1218)
4. Add explosion SFX entry to `EXPLODE_SFX` mapping (line ~678)

### Adding a New Boss
1. Add boss design to `BOSSES[]` array with name, colors, hpMul
2. Add attack pattern array to `BOSS_PATTERNS[]` (must match index in BOSSES)
3. Patterns are functions: `(boss) => { fireRadial(boss, ...); ... }` — see examples lines ~1360-1408
4. Add `drawBoss()` rendering case (around line ~1630)

### Adjusting Difficulty Curves
- Edit `DIFFICULTY` object (lines ~490-495) to change spawn rates, health multipliers, score scaling
- Ramp formula: `spawnBase - (frame / 300) * 4` — increase divisor for slower ramp
- Enemy health by type: `hp` field in `ENEMY_TYPES`

### Adding Visual Effects
- **Particle system**: `spawnExplosion(x, y, color, count)` (line ~1100); particles decay and render as fading circles
- **Screen shake**: `screenShake(magnitude, duration)` — applies random translation if not reduced-motion
- **Flash overlay**: `flashHit()` or `flashBomb()` — CSS-driven pulse effects
- **Combo popup**: `showComboPop(text)` — animated text at screen center

### Sound Design
- Edit `sfx` object properties to tweak frequencies, durations, filters
- `beep()` and `noiseBurst()` are the two synthesis primitives
- Use `panFor(x)` helper to spatialize shots based on world position

## Performance Notes

- **Canvas resolution**: Fixed 480×720, scaled via CSS to fit viewport (MAX 1.6× scale)
- **Collision checks**: Triple-nested loops (enemies × bullets, boss × bullets, etc.). Optimized with early break on hit.
- **Quality toggle**: `quality` variable gates shadow blur and glow effects; set via quality slider in UI
- **Reduced motion**: Disables shake, flash, and animations if user prefers (CSS + flag checks in code)
- **Particle cap**: No hard limit; culls by lifespan

## Debug Tips

- **Frame rate**: `frame` counter increments each update; logged in console during development
- **Pause**: Press `P` or `Escape` to pause and inspect state
- **Console**: `player`, `enemies`, `bullets`, `boss` are all global — inspect in DevTools
- **Themes**: Edit `THEMES` array to prototype new stage visuals
- **Best score reset**: `localStorage.clear()` or `localStorage.removeItem('best')`
