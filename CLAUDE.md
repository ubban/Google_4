# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and vanilla JavaScript. The entire game is contained in a single `index.html` file (~1669 lines) that combines HTML markup, CSS styling, and JavaScript game logic.

Game features:
- Vertical scrolling arcade shooter with stage-based progression
- Multiple enemy types (grunt, interceptor, tank) with different behaviors
- Boss encounters at the end of each stage with pattern-based attacks
- Weapon upgrade system (Vulcan, Laser, Spread, Homing)
- Combo/kill-chain multiplier system for score bonuses
- Smart bomb power-ups
- Procedurally generated audio (no external audio files)
- Touch controls and desktop mouse/keyboard support
- Three difficulty settings (Easy, Normal, Hard)
- Persistence of best score and settings via localStorage

## Architecture

### Game State Structure

The game maintains state in the following top-level collections and variables:

- **Player entity** (`player`): Position, dimensions, health, weapon, invulnerability frames, fire cooldown
- **Enemies** (`enemies`): Array of active enemy objects with position, health, type, behavior pattern
- **Bullets** (`bullets`): Player projectiles with position, velocity, piercing capability, laser flag
- **Enemy bullets** (`enemyBullets`): Enemy projectiles with simpler physics
- **Power-ups** (`powerUps`): Dropped items (life, weapon upgrades, bombs)
- **Particles** (`particles`): Visual effects (explosions, debris) with velocity and lifetime
- **Boss** (`boss`): Current boss entity if active, with position, health, pattern state, animation

Additional state:
- `frame`: Frame counter for frame-dependent logic and animations
- `kills`, `stageKills`: Kill counters for difficulty scaling and stage progression
- `chainTimer`: Tracks active kill-chain for combo multiplier
- `spawnTimer`: Controls enemy spawn rate

### Core Game Loop

Entry point: `loop(now)` (line 1640) — called via `requestAnimationFrame`

1. Calculates delta time (`dt`) from previous frame
2. Calls `update(dt)` to process all game logic
3. Calls `draw()` to render the current frame state
4. Updates HUD display elements

### Major Systems

**Update System** (`update(dt)`, line 1148):
- Player movement from keyboard/joystick/mouse input
- Weapon firing and cooldown
- Enemy and boss spawning with difficulty scaling
- Projectile movement and filtering (off-screen removal)
- Collision detection (6 separate pass-through checks):
  - Player bullets → enemies
  - Player bullets → boss
  - Enemy bullets → player
  - Enemy contact → player
  - Boss contact → player
  - Power-ups → player
- Kill-chain timer countdown
- Particle updates

**Enemy System**:
- Three enemy types with different appearance, health, and behavior:
  - `grunt`: Basic weaving enemies, slow-firing
  - `interceptor`: Fast-moving, spread-shot fire
  - `tank`: Heavily armored, rare, high score
- Enemies spawn via `spawnEnemy()` at rates controlled by difficulty
- Weaving animation uses `Math.sin` with per-enemy seed for variation
- Shoot pattern: either fixed spread or aimed at player

**Boss System**:
- Five bosses (accessed via `BOSSES` array, line 508)
- Multiple firing patterns via `BOSS_PATTERNS` object (line 1372):
  - `fireRadial`: Shots in all directions
  - `fireAimedSpread`: Spread shots aimed at player
  - `fireColumns`: Vertical columns
  - `fireSpiralBurst`: Rotating spiral
  - `fireCross`: Cardinal directions
  - `fireDoubleRing`: Two concentric circles at different speeds
  - `fireHomingSeeded`: Homing projectiles
- Boss enter animation and exit behavior
- Pattern transitions based on phase (health thresholds)

**Weapon System**:
- Four weapon types in `player.weapon`:
  - `vulcan`: Basic projectiles, fast fire rate
  - `laser`: Piercing beam, slower fire rate, limited pierce
  - `spread`: Multiple projectiles per shot
  - `homing`: Seeks nearest enemy (unused in current version but defined)
- Fire delay varies by weapon type
- Upgrades stack (`player.weaponLevel` increases damage and behavior)

**Collision & Hit Detection**:
- `rectsOverlap(a, b)`: Rectangle AABB collision (line 1069)
- Reverse iteration in collision loops prevents array mutation issues
- Boss bullets use different damage threshold (3 vs 1 for regular enemies)

**Procedural Audio** (`sfx` object, line 624):
- No external audio files; all sounds generated via Web Audio API
- Beep synthesis for UI and projectiles
- Noise bursts for explosions
- Frequency-based spatial audio panning via `panFor(x)`
- Mute state persisted to localStorage

**Visual Effects**:
- Screen shake magnitude and duration: `screenShake(mag, dur)`
- Flash vignette overlay for damage/critical moments
- White screen flash on hit
- Particle spawning on explosions: `spawnExplosion(x, y, color, count)`
- Combo pop-up animations

### Difficulty System

Three difficulty modes (line 485) control:
- `spawnBase`: Base spawn interval
- `spawnMin`: Minimum spawn interval (difficulty scaling floor)
- `enemyHp`: Enemy health multiplier
- `enemyScore`: Score multiplier

Difficulty is persisted to localStorage and affects both enemy spawning and balance.

### Input Handling

**Keyboard**: Tracked in `keys` object, updated via `keydown`/`keyup` (WASD + Arrow keys)

**Mouse**: 
- `mouseActive` flag when mouse moves over canvas
- `mouseTarget` used for smooth player following (pulled toward cursor)
- `canvasPointFromEvent()` handles canvas-relative coordinates

**Touch/Joystick** (for mobile):
- Virtual joystick on left side (move zone)
- Fire and bomb buttons on right
- Joystick logic in `joystickStart/Move/End()`
- Automatically enabled if `isTouchDevice` is true

**UI Controls**:
- Pause via `pauseBtn` (toggles `paused` flag; pause hides HUD updates but game still runs)
- Mute toggle for audio
- Motion reduction toggle (affects shake, flash, pulse intensity)
- Difficulty selection (buttons disabled during active gameplay)

### Game Flow

1. **Init**: `resetGame()` initializes all state, clears arrays, resets player
2. **Running**: Main loop executes update/draw until `gameOver` flag is set
3. **End Game**: `endGame()` stops loop, shows overlay with final score
4. **Restart**: Player clicks "Start" button to call `resetGame()` and begin loop again

## Code Organization

The entire codebase is within an IIFE (lines 423–1669) to avoid global namespace pollution.

**Sections** (rough order):
1. DOM element queries (lines 426–461)
2. Constants and configuration (lines 463–520)
3. Utility functions: audio, visual effects (lines 468–730)
4. Canvas and input setup (lines 753–840)
5. Game entity spawning and destruction (lines 1022–1103)
6. Main game logic: update, draw, boss AI (lines 1148–1638)
7. Event listeners and game loop (lines 1640+)

## Development Notes

### Modifying Game Balance

- **Enemy spawning**: Adjust `DIFFICULTIES` object and `KILLS_PER_STAGE` (line 491)
- **Boss health/damage**: `BOSSES` array (line 508) and `updateBoss()` (line 1411)
- **Weapon behavior**: Edit projectile spawn logic in `fireWeapon()` (line 1124) or pattern functions
- **Scoring**: `addScore(base)` (line 1087) multiplies by combo; adjust base values in enemy spawn logic

### Adding New Features

- **New enemy type**: Add to `ENEMY_TYPES` object, spawn logic in `spawnEnemy()`, draw logic in `draw()`
- **New weapon**: Add to weapon name/label logic, fire behavior in `fireWeapon()`, HUD display
- **New boss pattern**: Add function following the pattern of existing ones (e.g., `fireRadial`), reference in `BOSS_PATTERNS`

### Canvas Rendering

- **Base resolution**: 480×720px (`BASE_W`, `BASE_H`, line 753)
- **Scaling**: Responsive via `resize()` function, maintains aspect ratio
- **Rendering order**: Background clouds → enemies → player → bullets → effects → HUD
- **Drawing functions**: `drawPlane()` (player ship), `drawBossShip()` (boss), custom rect/circle primitives

### Known Limitations

- Single-file architecture means any feature addition increases file size
- No module system; all functions share a common scope
- Particle effects are simple colored circles; complex visual effects would require canvas compositing
- No persistent save system beyond localStorage (best score, mute, difficulty)

### Testing & Debugging

- Open browser DevTools (F12) to access console
- `frame` counter and `frameTimes` array can be logged to monitor performance
- `announce()` function (line 468) sends debug messages to accessibility live region
- Screen shake, flash, and vignette can be disabled via motion reduction toggle for reduced-motion testing
