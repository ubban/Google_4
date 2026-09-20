# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML5 Canvas application. All code is self-contained in `index.html` (1669 lines).

## Running the Game

- **To play locally**: Open `index.html` directly in a modern web browser (Chrome, Firefox, Safari, Edge)
- **No build process required**: This is a standalone HTML file with embedded JavaScript and CSS
- **Browser requirements**: HTML5 Canvas support, Web Audio API (for sound), touch/mouse input

## Game Controls

- **Mouse**: Move to aim, click to shoot
- **Touch**: Drag to aim, tap to shoot
- **Keyboard**: Spacebar to toggle pause, arrow keys for movement (if implemented)
- **Weapons**: Cycle with weapon switch (default weapon and laser modes)
- **Bombs**: Smart bombs (limited quantity per stage)

## Code Architecture

The game is structured as a single JavaScript application (IIFE pattern) with the following main components:

### Core Systems

1. **Game State Management**
   - `running`, `paused` flags for game lifecycle
   - `score`, `combo`, `lives`, `bombs` tracking
   - Stage progression and boss stages

2. **Entity Systems** (all stored as simple object arrays)
   - `player`: Single plane object with position, health, weapons
   - `enemies`: Array of enemy planes with varying types (fighter, tank, boss)
   - `bullets`: Player projectiles (laser and regular types)
   - `enemyBullets`: Incoming enemy fire
   - `powerUps`: Life and bomb pickups
   - `particles`: Visual effects (explosions, hits)

3. **Enemy Types**
   - Regular fighters: Basic enemy planes
   - Tanks: Armored enemies with health bars
   - Boss: Special stage boss with custom behavior

4. **Game Loop**
   - `loop(now)`: RequestAnimationFrame-based game loop
   - `update(dt)`: Physics, collision, AI, spawning
   - `draw()`: Canvas rendering using fillRect and arc primitives

5. **Input Handling**
   - Mouse tracking for aim cursor
   - Touch event handling for mobile
   - Click/tap for weapon firing

6. **Audio**
   - Web Audio API used for sound effects
   - Loaded dynamically with audio fallbacks

7. **Performance Optimization**
   - Dynamic quality settings (glow effects disabled on low quality)
   - Delta time clamping to prevent frame skip physics
   - Early rejection rendering optimization

## Key Implementation Details

### Collision Detection
- Uses AABB (axis-aligned bounding box) collision with simple distance checks
- Implemented in `update(dt)` collision loop

### Rendering
- Canvas 2D context drawing
- All entities drawn as simple shapes (rectangles, circles, triangles)
- Player and enemy planes drawn with custom `drawPlane()` function
- Boss ship has special `drawBossShip()` rendering
- Layering with z-index via drawing order and `ctx.globalAlpha`

### Weapon System
- Two weapon types: standard bullets and lasers
- Weapon switching in `fireWeapon()`
- Different bullet properties (speed, damage, spread pattern)

### Screen Shake and Visual Feedback
- `shakeTime` and `shakeIntensity` variables for hit feedback
- Flash overlay with white screen on bomb detonation
- Vignette effect (red screen edge) on critical health

## Development Notes

### Adding New Features

**New Enemy Type:**
1. Add entry to `ENEMY_TYPES` object with stats (hp, speed, score, drop rates)
2. Update `spawnEnemy()` to include the new type
3. Add unique behavior in `update(dt)` enemy movement section if needed
4. Add rendering logic in `draw()` if it needs special visuals

**New Weapon:**
1. Extend weapon property in player object
2. Add bullet type handling in `fireWeapon()`
3. Update bullet properties array for spread/speed/damage
4. Render new bullet style in `draw()` bullets section

**Visual Effect:**
1. Particle effects are simple (x, y, vx, vy, life, maxLife, color)
2. Spawn new particles in `update()` during events (hits, explosions)
3. Particles auto-decay based on life counter

### Code Style

- **Vanilla JavaScript**: No dependencies or frameworks
- **Immediate invocation**: Code wrapped in `(function() { ... })()` IIFE
- **Canvas primitives**: Uses basic 2D canvas methods (fillRect, arc, moveTo, lineTo)
- **Object arrays**: Entities managed as simple arrays of objects with numeric properties
- **Delta time**: All movement uses `dt` (delta time) parameter for frame-rate independence

### Common Development Tasks

**Testing gameplay changes:**
- Edit `index.html` directly in any text editor
- Reload browser to see changes immediately
- Use browser DevTools console to inspect game state variables
- Adjust numerical constants (speeds, damages, spawn rates) at top of game code

**Debugging:**
- Open browser DevTools (F12)
- Game state variables are in function scope but can be logged in `update()`
- Use `console.log()` to trace entity arrays and debug logic
- Check browser console for any JavaScript errors

**Performance tuning:**
- The quality variable controls glow effects (set to 0 for low quality)
- Frame time is tracked in `trackPerf(dt)`
- Canvas size is responsive (uses max-width/max-height CSS)

## Browser Compatibility

- Requires HTML5 Canvas and Web Audio API
- Works on desktop (Chrome, Firefox, Safari, Edge) and mobile (iOS Safari, Chrome Android)
- Touch controls optimized for mobile devices with viewport meta tags

## File Structure

```
Google_4/
├── index.html          (1669 lines - complete game)
├── README.md           (project metadata)
└── CLAUDE.md           (this file)
```
