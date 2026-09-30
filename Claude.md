# Claude.md - Sky Fighter Project Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML5 canvas game. It features player aircraft combat, multiple enemy types, boss battles, weapon variety, and power-up systems.

## Key Features

- **Player Mechanics**: Vertical movement, weapon switching (Bullets, Missiles, Laser)
- **Enemy Types**: Spawners (weak, regenerating), Tanks (heavy armor), and Stage Bosses
- **Weapon System**: 
  - Bullets: Fast, weak projectiles
  - Missiles: Slow, explosive, multi-hit
  - Laser: Continuous beam weapon with overheating
- **Power-ups**: Life recovery, bombs (screen-clear smart bomb), and weapon upgrades
- **Boss Battles**: Multi-stage boss encounters with distinct attack patterns
- **Game States**: Title screen, gameplay, level completion, game over, pause menu
- **Visual Effects**: Particle system, screen shake, flash effects, vignette damage indicator
- **Audio**: Sound effects and background music (via Web Audio API)
- **Performance Tracking**: FPS monitoring and performance metrics

## File Structure

The project is a **single-file application**:

```
Google_4/
├── index.html        (1669 lines) - Complete game implementation
├── README.md         - Project metadata
└── Claude.md         - This file
```

## Architecture Overview

### index.html Structure

The game is organized as a single-page HTML5 application with embedded CSS and JavaScript:

1. **HTML Structure** (lines 1-68)
   - Canvas element for rendering
   - HUD elements for stats display
   - Visual effects layers (vignette, flash)

2. **CSS Styling** (lines 10-275)
   - Game canvas styling
   - HUD UI with responsive design
   - Animations (heart beat, vignette pulse, flash effects)
   - Dark theme for space combat aesthetic

3. **JavaScript Game Engine** (lines 277-1669)

### Core Game Systems

#### Game State Management
- `gameState`: Tracks current game phase (title, playing, boss, complete, over)
- `level`: Current stage number (increases with each completed boss)
- `running`, `paused`: Game execution flags

#### Entity Management
- `player`: Player aircraft with position, velocity, health, weapons
- `enemies`: Array of spawned enemies (spawners, tanks)
- `boss`: Current boss entity (if active)
- `playerBullets`: Player's projectiles
- `enemyBullets`: Enemy projectiles
- `powerUps`: Collectible items (life, bombs, weapons)
- `particles`: Visual effects for explosions and impacts

#### Core Functions

**Initialization & Setup:**
- `initGame()` - Reset game state for new playthrough
- `initLevel()` - Spawn stage enemies and setup boss data

**Update Loop:**
- `update(dt)` - Main game logic update (60 FPS target)
  - Player input and movement
  - Bullet/projectile physics
  - Collision detection
  - Enemy AI and attacks
  - Particle effects and cleanup

**Rendering:**
- `draw()` - Render all game elements to canvas
- `drawPlane()` - Generic plane sprite renderer
- `drawBossShip()` - Boss-specific rendering with multi-segment visualization

**Core Gameplay:**
- `fireWeapon(ship, type)` - Generate projectiles for weapon type
- `clearScreen()` - Smart bomb effect (all enemies die, screen flash)
- `checkCollisions()` - Collision detection between all entities
- `spawnPowerUp(x, y)` - Create collectible items

**Input Handling:**
- Mouse movement for aiming
- Click for weapon fire
- Keyboard: Space (bomb), P (pause), R (restart)
- Touch support for mobile devices

**Audio System:**
- `playSound(type)` - Sound effect playback
- Background music during gameplay
- Context-sensitive audio (hit, explosion, power-up)

#### Constants & Configuration

**ENEMY_TYPES:**
- `spawner`: Weak regenerating enemies
- `tank`: Heavy armored foes with health bars
- Enemy spawn patterns and behaviors defined per level

**BOSS_DATA:**
- Stage-specific boss configurations
- Health, size, attack patterns
- Phase-based behavior changes

**Game Parameters:**
- Canvas dimensions
- Player speed, health, weapon cooldowns
- Enemy spawn rates and patterns
- Physics constants (gravity, collision sizes, etc.)

## Development Guidelines

### Code Style & Organization

1. **Variable Naming**: Clear, descriptive names
   - `playerBullets`, `enemyBullets` (plural for arrays)
   - `boss`, `enemies` (singular/plural as appropriate)
   - `frameCount`, `shakeTime` (temporal variables)

2. **Function Organization**: Grouped by responsibility
   - Setup functions (init*)
   - Game logic (update*)
   - Rendering (draw*)
   - Input handling (event listeners)
   - Utility functions (at end)

3. **Canvas Drawing**: Immediate mode (no retained scene graph)
   - Use `ctx.save()` / `ctx.restore()` for state management
   - Group related drawing operations
   - Apply transforms for rotation and scaling

### Common Tasks & Patterns

#### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant
2. Define properties: `hp`, `speed`, `w` (width), `h` (height), `color`, etc.
3. Add spawn logic in `initLevel()` or during gameplay
4. Add AI behavior in `update()` enemy movement section
5. Add rendering in `draw()` with special handling if needed

#### Adding a Weapon Type
1. Add weapon name to player's weapon cycle
2. Create firing logic in `fireWeapon()`
3. Handle weapon cooldown and special mechanics
4. Add projectile class with unique behavior
5. Update HUD weapon display

#### Modifying Boss Behavior
1. Edit `BOSS_DATA[stage]` configuration
2. Adjust `boss.attack` phase logic in `update()`
3. Modify boss drawing in `drawBossShip()` if visual changes needed
4. Adjust health/difficulty as needed

### Performance Considerations

- **Particle Cleanup**: Dead particles are removed from array each frame
- **Collision Detection**: O(n²) but reasonable for ~50 entities
- **Canvas Rendering**: No layers - single pass draw
- **FPS Target**: 60 FPS with `requestAnimationFrame`
- **Performance Monitoring**: Built-in `trackPerf()` system

### Testing the Game

1. **Browser Testing**:
   ```
   Open index.html in a modern browser (Chrome, Firefox, Edge)
   Test on desktop and mobile devices
   ```

2. **Gameplay Testing Checklist**:
   - Player movement (arrow keys / mouse)
   - Weapon switching and firing (click, space bar)
   - Enemy spawning and behavior
   - Collision detection (bullets, enemies, power-ups)
   - Boss encounter progression
   - Level completion and advancement
   - Game over and restart flow

3. **Cross-Platform**:
   - Desktop: Keyboard + mouse
   - Mobile: Touch movement and tap to fire
   - Tablet: Responsive canvas scaling

4. **Performance**:
   - FPS counter visible in HUD stats
   - Monitor for lag during heavy particle effects
   - Check memory usage with dev tools

## For Claude Code Agents

### When Making Changes

1. **Preserve Game Balance**: Test gameplay loops when adjusting difficulty
2. **Mobile Compatibility**: Ensure touch controls work alongside mouse input
3. **Performance**: Keep entity counts reasonable for 60 FPS
4. **Visual Clarity**: Maintain readable HUD and clear enemy distinctions
5. **Audio Context**: Web Audio API requires user gesture; handle audio initialization

### Testing Strategy

- Always test full gameplay loop: title → level → boss → completion → next level
- Verify collision detection edge cases
- Test rapid input sequences (spam clicking, fast movement)
- Check mobile viewport scaling
- Monitor performance metrics with built-in FPS tracking

### Common Pitfalls

- **Collision Bugs**: Remember hitboxes are centered; use `x ± w/2`, `y ± h/2`
- **State Management**: Reset appropriate variables in `initGame()` and `initLevel()`
- **Audio Context**: Audio must be initialized on first user interaction
- **Canvas Size**: Always account for `devicePixelRatio` for crisp rendering on high-DPI displays
- **Particle Cleanup**: Remove dead particles to prevent memory leaks

### File Size Consideration

The single-file approach keeps deployment simple but the HTML is now 58KB. If adding significant features:
- Consider modularization only if file becomes unmaintainable
- Current approach works well for single-player, self-contained game

## Branch Information

Development happens on: `claude/inspiring-planck-agtwnr`

## Last Updated

Generated by Claude Code - 2026-09-30
