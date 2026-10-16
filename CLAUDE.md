# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Sky Fighter: Game Architecture

Sky Fighter is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML5 canvas file. The game features weapon upgrades, stage progression, boss battles, combo mechanics, and multiple difficulty levels.

## Running the Game

The game is self-contained and requires no build tools:

1. **Local Development**: Open `index.html` directly in a web browser
2. **Server (Recommended for Touch Testing)**: Run a local HTTP server:
   ```bash
   python3 -m http.server 8000
   # Then visit http://localhost:8000
   ```
3. **Live Testing**: The game automatically saves high scores to localStorage

## Core Game Architecture

### Game Loop & State Management
- **Main Loop**: `gameFrame()` runs at ~60fps using `requestAnimationFrame`
- **Game States**:
  - Title screen (before `startGame()`)
  - Active gameplay (between `startGame()` and `endGame()`)
  - Game over screen (after `endGame()`)

### Entity System
The game maintains several lists of active entities:
- **`player`**: Single player object with position, velocity, health, invulnerability frames, and weapon state
- **`enemies`**: Array of enemy ships with health, movement patterns, and bullet spawning
- **`playerBullets`**: Array of projectiles fired by the player (velocity-based)
- **`enemyBullets`**: Array of projectiles fired by enemies (velocity-based with special radial bullets)
- **`particles`**: Temporary visual effects (explosions, debris) that fade out after their TTL

### Collision System
- **Broad-Phase**: Spatial checks based on bounding boxes (AABB collision)
- **Hit Detection**:
  - `checkPlayerBulletEnemyCollision()`: Player bullets vs enemies
  - `checkEnemyBulletPlayerCollision()`: Enemy bullets vs player
  - `checkPlayerEnemyCollision()`: Direct contact damage
  - Collision checks run once per frame in the main update loop

### Stage & Difficulty Progression
- **Stages 1-20**: Each stage increments difficulty with more enemies and tighter patterns
- **Themes**: 5 theme cycles (e.g., "Azure Sky", "Crimson Storm") repeating every 5 stages
- **Enemy Spawning**:
  - `spawnEnemy()` called based on `stageTimer`
  - Spawn count increases with stage: `Math.floor(stage / 4) + 1`
  - Enemy health scales: `5 + stage / 3`
- **Boss Stages**: Bosses appear at stages 1, 3, 5, 7, and then every 3 stages
  - 6 unique boss designs with their own bullet patterns
  - Boss patterns cycle through in `BOSS_PATTERNS` array

### Weapon System
- **5 Weapon Types**:
  1. **Laser**: Fast single-shot projectiles
  2. **Missiles**: Slow homing projectiles
  3. **Spread**: Triple-shot fan pattern
  4. **Plasma**: Explosive arc trajectory
  5. **Cannon**: Wide spread with high damage
- **Weapon Upgrades**: Levels 0-5, affecting damage and fire rate
- **Weapon Drops**: Enemies drop weapon pickups when killed
  - Pickup logic: 40% chance, random weapon type
  - `fireWeapon()` handles projectile creation per weapon type

### Scoring & Combo System
- **Base Points**: Enemies = 10 pts, Bosses = 500 pts
- **Combo Multiplier**: `comboPower` starts at 1.0
  - Increases by 0.15 per consecutive hit (max 3.0)
  - Resets on missed shot or taking damage
  - Displayed in HUD with orange "x" indicator
- **Score Storage**: High score saved to localStorage as `bestScore`

### Boss Battle System
Each boss has:
- **3 Attack Patterns**: Each pattern fires a different bullet arrangement
- **Telegraph System**: 22-frame visual warning before attack (red outline pulse)
- **Attack Timing**: 85 frames between pattern cycles
- **Movement**: Horizontal bouncing between canvas edges
- **Health**: Bosses have 40-60 HP depending on stage

**Boss Patterns Available**:
- `fireRadial()`: Bullets in all directions
- `fireAimedSpread()`: Aimed fan shots toward player
- `fireColumns()`: Vertical columns of bullets
- `fireHomingSeeded()`: Missiles that track player
- `fireSpiralBurst()`: Spiral wave pattern
- `fireDoubleRing()`: Concentric rings of bullets
- `fireCross()`: 8-directional cross pattern

### Visual & Audio Systems
- **Canvas Rendering**: 2D context used for all drawing
  - Background: Gradient (sky to ocean blue)
  - Sprites: Simple geometric shapes (triangles for ships, squares for bullets)
  - HUD: Overlays for score, health, weapons (positioned with safe-area insets for mobile)
- **Visual Effects**:
  - `screenShake(duration, intensity)`: Temporary camera jitter
  - `flashHit()` / `flashBomb()`: White screen flash on events
  - `spawnExplosion()`: Particle burst with fade animation
  - Vignette effect during critical health (pulsing red overlay)
- **Sound Effects**: Via `sfx` object (methods like `sfx.shoot()`, `sfx.hit()`, `sfx.bossTelegraph()`)

### Input Handling
- **Keyboard**: Arrow keys (up/down/left/right), Space (bomb)
- **Touch/Mobile**: 
  - One-finger drag to move player
  - Two-finger tap for bomb (or on-screen button)
  - Crosshair cursor follows mouse/touch position
- **Touch-Action**: Prevented on canvas to avoid browser interference

### Mobile Responsiveness
- Canvas scales to fit viewport (`max-width: 100%`, `max-height: 100%`)
- Safe area insets respected for notches/unsafe zones
- Responsive font sizes using `clamp()`
- HUD elements reflow on smaller screens

## Key Game Constants

```javascript
const WEAPON_NAMES = ['Laser', 'Missiles', 'Spread', 'Plasma', 'Cannon'];
const BOSSES = ['VANGUARD', 'BEHEMOTH', 'SPECTER', 'DREADNOUGHT', 'WRAITH', 'TITAN'];
const THEMES = [
  { name: 'Azure Sky', ... },
  { name: 'Crimson Storm', ... },
  // ... etc
];
```

## Common Editing Tasks

### Adding a New Enemy Type
1. Add logic in `spawnEnemy()` for spawn conditions
2. Define movement/AI in `updateEnemies()` loop
3. Create firing pattern via `fireWeapon()` or custom bullet logic
4. Set initial health and sprite appearance

### Adding a New Boss Pattern
1. Create a new pattern function (e.g., `fireNewPattern()`)
2. Add to the appropriate boss's pattern array in `BOSS_PATTERNS`
3. Set telegraph and attack timing via `b.telegraph` and `b.attackTimer`

### Adjusting Difficulty
- **Enemy Spawn Rate**: Modify `spawnTimer` increment in main loop
- **Enemy Health**: Change base health formula in `spawnEnemy()`
- **Bullet Density**: Adjust loop counts in firing pattern functions
- **Boss Health**: Modify `boss.maxHp` assignment

### Adding Visual Polish
- Screen shake calls: `screenShake(duration, intensity)`
- Particle effects: `spawnExplosion(x, y, color, count)`
- Flashes: Add `flashHit()` or `flashBomb()` calls on events
- Animations: Modify keyframes in `<style>` section

## File Structure

- **index.html** (1669 lines): Complete game implementation
  - Lines 1-400: HTML structure + CSS styling + safe-area insets
  - Lines 400-1100: Game logic, entity management, collision
  - Lines 1100-1400: Boss patterns and AI
  - Lines 1400-1669: Main game loop, input handling, event listeners

## Testing & Debugging

- **Browser Console**: Access game state via `player`, `boss`, `stage`, `lives`, `score`
- **LocalStorage**: High scores stored in `bestScore`
- **Performance**: Monitor frame rate in DevTools; `gameFrame()` should stay ~60fps
- **Mobile Testing**: Use DevTools device emulation or run on actual device via local server

## Notes for Future Development

- The game is intentionally self-contained in one file for simplicity and portability
- All game state is maintained in global variables for easy access during debugging
- Particle and bullet systems use direct array manipulation for performance
- Canvas is not scaled; coordinates are raw pixel values (not using canvas scaling transforms)
- Sound effects are abstracted via `sfx` object but not fully implemented in this version
