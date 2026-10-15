# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla JavaScript with HTML5 Canvas. The entire game is implemented in a single `index.html` file (~1670 lines) with no build system or external dependencies.

## Running the Game

Open `index.html` directly in a web browser. The game is fully playable in any modern browser that supports:
- HTML5 Canvas API
- Web Audio API (for sound effects)
- Touch events and mouse input

The game runs at 60 FPS with adaptive quality settings based on performance metrics.

## Game Architecture

### Core Game Loop
The game uses `requestAnimationFrame()` with three main phases:
1. **Update**: Game state changes, collision detection, entity movement
2. **Draw**: Render everything to canvas
3. **Performance tracking**: Monitor frame time and adjust quality dynamically

### Key Game Systems

**Game State**
- `running`, `paused`, `gameOver`: Control game flow
- `lives`, `score`, `level`: Player progression
- `playerWeapon`: Current weapon type (0=bullets, 1=laser)
- `quality`: Graphics quality (0=glow effects, 1=simplified)

**Entity Types**
- **Player**: Single aircraft controlled by mouse/touch, can switch weapons
- **Enemies**: Multiple types defined in `ENEMY_TYPES` (normal, tank, boss)
- **Bullets**: Player projectiles (regular bullets and lasers)
- **Enemy Bullets**: Projectiles fired by enemies
- **Power-ups**: Life and bomb pickups with animated rotation
- **Particles**: Visual effects for explosions and impacts
- **Boss**: Special boss entity that appears at stage progression

**Collision System**
- Simple rectangular collision detection
- Separate collision checks for:
  - Player bullets hitting enemies
  - Enemy bullets hitting player
  - Power-ups hitting player
  - Boss collision detection

### Visual Rendering

**Canvas Rendering Details**
- Sky gradient background: `#0d2f5e` to `#7fc1ff`
- Multiple rendering passes: enemies → bullets → boss → power-ups → particles
- Adaptive glow effects controlled by `quality` variable
- Shadow/glow effects using Canvas `shadowColor` and `shadowBlur`
- Particle system for visual feedback on impacts

**HUD Elements**
- Lives counter, score, current level displayed at top
- Game over screen with final stats and best score tracking
- Performance metrics (FPS, frame time) in bottom-right corner
- Crosshair reticle following mouse position

### Audio System

The game uses Web Audio API with:
- `ensureAudio()`: Initializes audio context (needed for browser autoplay policies)
- Sound effects for shooting, explosions, and UI interactions
- Background music support

## Development Workflow

### Making Changes

1. **Edit directly in `index.html`**: Make changes to the code sections
2. **Test immediately**: Open the file in a browser to see changes live
3. **Performance impact**: Be mindful of the update/draw functions — complex changes may require quality setting adjustments

### Key Code Sections (by line ranges, approximate)

- **Game configuration**: ENEMY_TYPES, PLAYER_TYPES constants near the top
- **Game state initialization**: `resetGame()` function
- **Update logic**: `update(dt)` function handles physics and collision
- **Rendering**: `draw()` function and helper functions (`drawPlane()`, `drawBossShip()`)
- **Input handling**: Mouse/touch listeners for player control
- **Game loop**: `loop(now)` function drives the animation frame

### Common Modifications

**Adjust difficulty**
- Modify `ENEMY_TYPES` to change enemy speed, health, fire rate
- Adjust `spawnInterval` to control enemy spawn frequency
- Tweak `enemyBulletSpeed` for incoming fire difficulty

**Add new weapons**
- Extend `PLAYER_TYPES` with new weapon definitions
- Implement shooting pattern in the appropriate section of `update()`
- Ensure collision detection includes new bullet types

**Customize visuals**
- Sky gradient colors in `draw()` background setup
- Entity colors defined in `ENEMY_TYPES` and `drawPlane()` function
- Shadow/glow colors for bullet rendering

**Optimize performance**
- The `quality` variable controls glow effects (0 = glow enabled)
- Particle count can be reduced for lower-end devices
- Adjust canvas rendering batching if needed

### Testing Changes

Since there are no automated tests, manual testing is the validation method:
1. Open the game in browser with DevTools open (F12)
2. Check Console for any JavaScript errors
3. Monitor Performance tab to ensure frame rates stay above 30 FPS
4. Test on multiple devices/browsers, especially mobile touch input

## Browser Compatibility Notes

- Requires modern browser with Canvas and Web Audio API support
- Touch input fully supported for mobile/tablet play
- Mouse input for desktop play with crosshair reticle
- Falls back gracefully if audio context unavailable (game still playable)
- Uses `env(safe-area-inset-*)` for notch/safe area handling on mobile

## Performance Considerations

- Frame time tracking in `trackPerf()` function monitors game health
- Dynamic quality adjustment keeps game playable on lower-end devices
- Particle effects automatically scaled based on performance
- Enemy bullet count grows with stage progression — be cautious with difficulty scaling
