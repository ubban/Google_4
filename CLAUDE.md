# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/JavaScript project. The entire game runs in the browser with no build process, external dependencies, or asset files.

## File Structure

- `index.html` – Complete game implementation (HTML, CSS, and JavaScript)
- `README.md` – Project metadata

The game is entirely self-contained in a single file.

## Architecture

The game uses a single-file architecture organized into logical sections within `index.html`:

### Game Loop & State
- **Main Loop**: `loop()` function at ~1640 – Requestanimationframe-based loop handling timing and calling update/draw
- **Game State**: Variables at ~864 (player, bullets, enemies, boss, score, lives, etc.)
- **Update**: `update(dt)` function at ~1148 – Physics, collision, enemy AI, spawning
- **Render**: `draw()` function at ~1520 – Canvas drawing with theme/particle system

### Player & Controls
- **Player Object**: Properties at ~877 (position, weapon, level, cooldown)
- **Input Handling**: ~768-861 – Keyboard (WASD/Arrows), mouse (steering), touch (joystick + buttons)
- **Weapon System**: `fireWeapon()` at ~1124 – Vulcan and Laser weapons with upgrade levels
- **Firing**: Player fires on Space/click/touch, bombs with X/Shift/touch

### Enemies & Boss
- **Enemy Types**: ENEMY_TYPES constant at ~502 (grunt, interceptor, tank with unique properties)
- **Enemy Spawning**: `spawnEnemy()` at ~1022 – Stage-aware spawning with difficulty curves
- **Boss System**: BOSSES constant at ~508 (6 boss designs), `spawnBoss()` at ~1043, `updateBoss()` at ~1411
- **Boss Patterns**: BOSS_PATTERNS at ~1372 – 6 pattern sets (radial, aimed spread, columns, spirals, homing)

### Combat & Collision
- **Collision**: `rectsOverlap()` at ~1069 – AABB detection for all objects
- **Damage**: Enemy/Boss HP tracked, bullets pierce based on weapon
- **Explosions**: `spawnExplosion()` at ~1073 – Particle bursts with color/count based on difficulty

### Scoring & Difficulty
- **Scoring**: `addScore()` at ~1087 – Chain/combo system with 1-10x multiplier
- **Difficulty Levels**: DIFFICULTIES at ~485 (easy/normal/hard) – Affects spawn rate, enemy speed, projectile count
- **Scaling**: Boss HP and enemy spawning scale with stage number

### Audio
- **WebAudio Synthesis**: `ensureAudio()` at ~545 – Creates audio context on first interaction
- **Sound Effects**: sfx object at ~624 – Procedurally generated beeps/noise for all events
- **Muting**: Toggle with mute button, persisted to localStorage

### Visual Systems
- **Themes**: THEMES at ~493 – 6 color schemes cycling by stage (sky gradient, sun glow, clouds)
- **Screen Shake**: `screenShake()` at ~727 – Magnitude/duration based on events, respects reduced-motion
- **UI Elements**: HUD at ~367, boss bar at ~376, overlay/menus at ~401
- **Animations**: CSS keyframes for UI (heartBeat, cardIn, stagePulse, etc.)

### Persistence & Settings
- **Local Storage Keys**: STORAGE_* constants at ~463-466
  - Best score, mute state, difficulty, reduced-motion preference
- **Accessibility**: Screen reader announcements via `announce()` at ~468, reduced-motion support

## Development Notes

### Running the Game
- Open `index.html` in a browser – no build step required
- Game is touch-friendly (mobile + desktop)
- Use responsive canvas sizing (clamped to 1.6x base 480x720)

### Making Changes

**Adding Features**:
1. State variables go in the "Game state" section (~864)
2. Update logic goes in `update(dt)` (~1148)
3. Drawing logic goes in `draw()` (~1520)
4. Keep physics frame-rate independent using `dt` (delta time)

**Difficulty Tuning**:
- Modify DIFFICULTIES at ~485 (spawn rate, enemy speed, enemy shoot chance, lives, bomb count, boss HP)
- Boss HP formula at ~1046: base HP * difficulty multiplier * boss design multiplier

**Adding Enemies**:
- New enemy type: Add to ENEMY_TYPES (~502) with w, h, hp, speedMul, color, score, minStage
- Spawning weights: Modify `spawnEnemy()` at ~1028-1030
- Explosion SFX: Add to EXPLODE_SFX at ~678

**Adding Bosses**:
- New boss design: Add to BOSSES (~508)
- Pattern set: Add 3-pattern array to BOSS_PATTERNS (~1372)
- Boss will cycle through designs by `(stage - 1) % BOSSES.length`

**Visual Polish**:
- CSS styles at ~10-358
- Drawing functions: `drawPlane()` (~913), `drawBossShip()` (~970), `drawClouds()` (~1507)
- Particles and explosions: `spawnExplosion()` particles have life, color, velocity

**Sound Design**:
- Beep/noise functions at ~566-623
- Sound effects use frequency sweeps, filters, panning
- Add new sounds to `sfx` object following existing patterns (frequency, duration, filters)

### Known Constraints
- Single-file architecture means no code splitting (intentional for zero-dependency deployment)
- Canvas rendering (no WebGL) – quality levels adjust particle count for performance
- Adaptive quality system at ~474-483 lowers visual fidelity if frame time exceeds 1/40s average
- Touch input requires user interaction to resume audio context (browser security)

### Performance
- Frame time tracking at ~476 (45-frame rolling average)
- Quality degradation: full → medium (60% particles) → low (35% particles)
- Boss shadow blur and glow effects disabled at quality > 0
- Particle count scaled per explosion based on quality level

### Testing the Game
While there are no automated tests, verify changes by:
1. Running the game in a browser
2. Testing golden path: start game → defeat enemies → reach boss → defeat boss → next stage
3. Edge cases: losing all lives, collecting power-ups, difficulty changes, pause/resume
4. Input methods: keyboard, mouse, touch
5. Audio: mute toggle, sound effects play correctly
6. HUD: score updates, multiplier displays, weapon upgrades show
