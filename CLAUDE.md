# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-page web application. The entire game logic, rendering, and UI are contained in `index.html` with embedded JavaScript and CSS.

## Development

### Running the Game

Open `index.html` in a web browser. The game runs directly without any build process or external dependencies.

For local development with a simple server:
```bash
# Python 3
python -m http.server 8000

# Python 2
python -m SimpleHTTPServer 8000

# Node.js (if available)
npx http-server
```

Then navigate to `http://localhost:8000/index.html`

### No Build/Test Infrastructure

This is a single-file game without:
- Build tools or bundlers
- Test frameworks
- Linting or formatting tools
- Version control branches (code is directly in the HTML)

## Architecture Overview

### Game Loop and State

The game maintains several key state objects and variables:

- **Game Objects**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Game State**: `score`, `lives`, `maxLives`, `running`, `paused`, `gameOver`, `stage`, `stageKills`
- **Progression**: `kills`, `stageKills`, `spawnTimer`, `frame`, `bossWarned`
- **Scoring**: `chain` (combo counter), `chainTimer`, `multiplier` (based on difficulty and combo)
- **Equipment**: `bombCount`, `maxBombs` (initially 2, upgradeable via pickups)

### Key Rendering & Game Logic Functions

**Drawing Functions** (non-interactive rendering):
- `drawPlane(x, y, w, h, color, darkColor, flame)` - Renders player/enemy ships
- `drawBossShip(b)` - Boss-specific rendering with animated wings and weapons
- `spawnExplosion(x, y, color, count)` - Particle-based explosion effect

**Game Mechanics** (core gameplay):
- `spawnEnemy()` - Random enemy spawning based on stage and frame count
- `spawnBoss()` - Triggered when `stageKills >= killsPerStage` (currently 20)
- `maybeSpawnPowerUp(x, y)` - 15% chance on enemy death to spawn weapon/bomb pickup
- `rectsOverlap(a, b)` - Simple AABB collision detection for all interactions

**Difficulty System**:
- Three difficulty levels (easy, normal, hard) stored in localStorage
- Affects `multiplier`, spawn rates, enemy health, and boss behavior
- Selected via difficulty buttons in the top-right corner

### Input Handling

**Touch/Mobile Joystick**:
- `joystickStart/Move/End()` functions handle virtual joystick input
- Position relative to player for diagonal movement
- `joyVec` stores normalized input vector

**Mouse/Keyboard**:
- Mouse position maps directly to player X; holding down fires
- Arrow keys for movement, spacebar to fire (desktop)
- Double-tap for bombs (mobile); B key (desktop)

### Audio System

Web Audio API powered with procedural sound effects:
- `beep(freq, dur, type, vol, opts)` - Sine/square wave tones for UI and pickups
- `noiseBurst(dur, vol, opts)` - Noise-based sound for explosions
- `panFor(x)` - Stereo panning based on screen X position for spatial audio
- Global `audioCtx` and `masterGain` for initialization and mute control
- Mute state persisted in localStorage

### UI Elements

**HUD** (top center):
- Hearts for remaining lives
- Bomb count display
- Current weapon type
- Score multiplier (color-coded by difficulty)

**Vignette**: Red flash effect that pulses during boss fights
**Flash Effects**: White screen flash on hit or bomb explosion
**Combo Pop**: Floating text showing combo streak
**Stage Banner**: "STAGE X" text shown at stage transitions
**Boss Health Bar**: Centered top bar when boss is active

### Performance & Motion

- `trackPerf(dt)` - Frame rate quality tracking (full/medium/low)
- `reducedMotion` flag respects system preferences (`prefers-reduced-motion`)
- `screenShake(mag, dur)` - Time-based screen shake effect (intensity + duration)
- Disables visual effects on low-performance devices

### Data Persistence

Uses localStorage for:
- `STORAGE_BEST` - High score
- `STORAGE_MUTE` - Mute toggle
- `STORAGE_DIFF` - Current difficulty
- `STORAGE_REDUCED` - Reduced motion preference

## Gameplay Mechanics

### Player Progression
- Starts with 3 lives and basic weapon
- Each stage requires `killsPerStage` (20) enemy kills to spawn boss
- Boss fight starts when `stageKills >= killsPerStage`
- Advancing stage increases spawn rate and difficulty

### Weapon System
- **Default**: Single-stream bullets
- **Pickups**: Replace weapon with new types (multi-shot, spread, etc.)
- Weapon determined by `player.weapon` property
- Current weapon shown in HUD

### Combo System
- `chain` increments for each enemy killed
- `chainTimer` resets when an enemy is killed; decreases over time
- Chain breaks after ~3 seconds without a kill
- Multiplier scales with chain count: base multiplier × chain bonus

### Power-ups
- **Weapons**: Change firing pattern (chance to spawn on enemy death)
- **Bombs**: Max 2 pickups (deployable with bomb button, instant screen-clear)

## Code Structure Notes

The single-file structure means:
1. All global state is in the closure scope
2. Game loop runs via `requestAnimationFrame` in the update/render cycle
3. Event listeners attached to `document` and canvas for input
4. Canvas context (`ctx`) is the primary rendering interface
5. No external module dependencies

When extending the game:
- Add new functions after existing utility functions but before the game loop
- Update state variables at the top of the function list
- New game objects should follow the shape of existing enemies/powerups
- Use existing collision (`rectsOverlap`) and spawning patterns as templates
