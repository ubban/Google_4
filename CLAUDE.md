# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML file (`index.html`). It combines embedded CSS for styling and vanilla JavaScript for game logic, using the HTML5 Canvas API for rendering. The game runs directly in modern browsers with no build step required.

## Quick Start & Common Commands

Since this is a single-file game, there's no compilation or installation needed:

- **Run the game locally**: Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
- **Serve for testing**: Use any simple HTTP server:
  - Python: `python -m http.server 8000`
  - Node.js: `npx http-server`
  - Then navigate to `http://localhost:8000`
- **Deploy**: Simply upload `index.html` to any static hosting service

## Architecture & Code Structure

The game is organized within a single IIFE (Immediately Invoked Function Expression) starting at line 423. Key architectural layers:

### 1. **Game State & Constants**
- `ENEMY_TYPES` (line 502): Defines enemy properties (truck, tank, helicopter) including speed, health, fire patterns
- `WEAPON_TYPES`: Weapon system with different fire patterns (standard, laser, spread, homing, etc.)
- Global variables track: player state, entities (bullets, enemies, power-ups), score/combo, difficulty

### 2. **Game Loop (line 1148: `update()` + Rendering)**
- **Physics**: `update(dt)` handles all game state changes per frame
  - Player movement and collision
  - Bullet lifecycle and despawn
  - Enemy AI and movement
  - Power-up collection and effects
  - Combo/score system
- **Rendering**: `draw()` function renders the entire game to canvas
  - Player ship (`drawPlane`)
  - Boss ship (`drawBossShip`)
  - Bullets, enemies, particles, HUD overlays

### 3. **Core Game Systems**

#### Enemy & Boss System
- `spawnEnemy()` (line 1022): Spawns random enemy types with difficulty scaling
- `spawnBoss()` (line 1043): Initiates boss battle with multi-pattern AI
- `updateBoss()` (line 1411): Boss-specific movement and attack patterns
- Enemy bullet patterns: `fireRadial`, `fireAimedSpread`, `fireCross`, `fireHomingSeeded`, etc.

#### Weapon System
- `fireWeapon()` (line 1124): Fires player bullets based on current weapon type
- Each weapon has unique fire pattern and behavior (laser, spread, homing, etc.)
- `WEAPON_TYPES` defines pattern selection logic

#### Collision & Damage
- `rectsOverlap()` (line 1069): AABB collision detection
- Bullet-enemy collision checked every frame
- Player damage triggers `loseLife()` (line 1456) and visual effects

#### Power-Up System
- `maybeSpawnPowerUp()` (line 1058): Random drop after enemy kills
- Types: life (red), bomb (yellow), weapon (green)
- `applyPowerUp()` (line 1304): Applies power-up effects to player state

#### Visual Effects
- `spawnExplosion()` (line 1073): Particle system for explosions
- `screenShake()` (line 727): Camera shake effect on impact
- `flashHit()`, `flashBomb()`: Screen flashes for feedback
- HUD updates: score, combo, lives, bombs, weapon display (lines 680-719)

### 4. **Input Handling**
- Mouse/touch targeting: `canvasPointFromEvent()` (line 840)
- Joystick support: `joystickStart()`, `joystickMove()`, `joystickEnd()` (lines 785-807)
- Platform-specific: Detects desktop vs mobile and adapts controls
- Keyboard pause: Spacebar for pause/resume

### 5. **Audio System**
- Synthesized sound effects (no audio files)
- `beep()` (line 566): Tone generation for various sounds
- `noiseBurst()` (line 597): White noise for explosions
- `panFor()` (line 562): Stereo panning based on position
- `ensureAudio()` (line 545): Handles browser autoplay policies

### 6. **Responsive UI & HUD**
- Dynamic scaling for all UI elements (uses `clamp()` CSS and `vw` units)
- Safe area insets for notched devices
- `resize()` (line 754): Handles canvas scaling to window size
- Two UI modes: game HUD (during play) and overlay (menu/end screen)

### 7. **Difficulty & Progression**
- `setDifficulty()` (line 741): Changes spawning and behavior rates
- Three difficulty levels with different enemy density and speed
- Boss patterns scale with progression

## Key Editing Patterns

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` constant (line 502) with properties: `w`, `h`, `hp`, `speed`, `fire`, `color`, `rate`
2. Define fire pattern function (e.g., `fireRadial`, `fireColumns`)
3. Update spawn logic in `spawnEnemy()` if needed

### Adding a New Weapon
1. Define fire pattern function (follows same naming as enemy patterns)
2. Add to weapon switching logic in `fireWeapon()` (line 1124)
3. Update `weaponLabel()` (line 871) for HUD display
4. Optionally add visual effect differences in rendering

### Tweaking Game Balance
- Enemy spawn rate: Adjust `spawnRate` at line 1023
- Enemy/boss health: Modify `hp` values in `ENEMY_TYPES`
- Player damage: Change value in `loseLife()` function
- Combo multiplier: Adjust `scoreMult` increments
- Power-up spawn chance: Modify condition in `maybeSpawnPowerUp()`

### Performance Optimization
- Quality detection: `quality` variable (line 480) disables glow effects on low-end devices
- Particle limit: Check `particles` array length before spawning
- Canvas rendering is full-screen; adjust if targeting slower devices

## Code Style & Conventions

- **Naming**: camelCase for variables/functions, SCREAMING_SNAKE_CASE for constants
- **Comments**: Minimal; code is self-documenting with clear names
- **No frameworks**: Pure vanilla JavaScript with no dependencies
- **Canvas coordinate system**: Origin (0,0) at top-left; +x right, +y down
- **Time-based animation**: All movement uses `dt` (delta time) for frame-rate independence
- **Physics**: Simple Euclidean distance for checks; no complex collision response (just overlap detection)

## Browser Compatibility

- Requires: HTML5 Canvas, requestAnimationFrame, Web Audio API
- Works: Chrome, Firefox, Safari 11+, Edge, modern mobile browsers
- Does not support: IE11 (uses const/let, arrow functions, template literals)

## Testing Approach

While there are no automated tests, validate changes by:
1. Testing all difficulty levels
2. Playing through at least one boss fight
3. Checking edge cases: screen edges, rapid input, multiple simultaneous effects
4. Verifying on mobile (touch/accelerometer controls)
5. Testing audio behavior (muted tab, autoplay policy)
6. Checking combo and score calculations work correctly

## File Structure

```
/Google_4
  index.html          # Single-file game (all HTML, CSS, JS)
  README.md           # Project info
  CLAUDE.md           # This file
  .git/               # Version control
```

## Performance Metrics

The game includes basic performance tracking (`trackPerf` at line 476) that logs frame rate to console. Check console in browser DevTools for FPS and delta time metrics.
