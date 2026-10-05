# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML, CSS, and JavaScript. The game runs entirely in a browser canvas with no external dependencies, featuring:

- Player-controlled fighter aircraft
- Weapon switching system (Vulcan, Laser, Missile)
- Smart bomb mechanics
- Enemy waves and boss encounters
- Collision detection and damage system
- Score multiplier system
- Responsive touch and mouse controls

## Project Structure

```
Google_4/
├── index.html          # Main game file (HTML + CSS + JavaScript)
├── README.md           # Project timestamp/metadata
└── CLAUDE.md           # This file
```

## File Organization

### `index.html` (Master Game File)

The entire game is contained in a single HTML file with embedded CSS and JavaScript sections:

- **HTML Section**: Canvas container and HUD elements
  - `#container`: Main game viewport
  - `#gameCanvas`: 800x600px game rendering surface
  - `#hud`: Heads-up display showing health, bombs, weapon, multiplier
  - `#vignette`: Screen damage effect overlay
  - `#flash`: Screen flash effect for hits/bombs

- **CSS Section**: 
  - Game styling and responsive design
  - Animation keyframes (heart beat, screen vignette pulse, hit/bomb flash)
  - HUD styling with semi-transparent panels
  - Safe-area insets for mobile devices
  - Cursor styling and touch action handling

- **JavaScript Section**: Complete game engine
  - Canvas rendering and animation loop
  - Game state management
  - Collision detection system
  - Enemy spawning and AI
  - Weapon systems and effects
  - Sound effects (optional, may use Web Audio API)
  - Input handling (keyboard, mouse, touch)

## Game Systems

### Core Gameplay Loop

1. **Render Loop**: 60 FPS canvas animation frame
2. **Update Phase**: Move entities, check collisions, update state
3. **Draw Phase**: Clear canvas, render all game objects, HUD

### Entity Systems

- **Player**: Controllable fighter at bottom of screen
  - Position: Constrained to bottom third
  - Health: 3 hits by default
  - Weapons: Vulcan (spread), Laser (focused), Missile (homing)
  - Smart Bombs: Limited use, screen-wide damage

- **Enemies**: Spawned in waves
  - Basic fighters: Standard attack patterns
  - Formations: Coordinated movement
  - Boss encounters: Larger health pools, pattern attacks

- **Projectiles**: 
  - Player shots: Multiple per frame depending on weapon
  - Enemy shots: Vary by enemy type
  - Collision system removes on hit

- **Effects**:
  - Explosions: Visual feedback on destruction
  - Damage vignette: Screen shake effect on hit
  - Screen flash: White flash on hit/bomb

### HUD Display

- **Hearts**: Visual health indicator (3 default)
- **Bombs**: Smart bomb count
- **Weapon**: Current weapon type (VULCAN, LASER, MISSILE)
- **Multiplier**: Score multiplier (increases on consecutive hits)

## Development Guidelines

### Adding New Features

1. **New Enemies**: Add to `spawnWave()` function
   - Define movement pattern in `update()` method
   - Set health and damage output
   - Add sprite/rendering logic

2. **New Weapons**: Add to weapon cycling system
   - Define firing pattern and projectile behavior
   - Update weapon display in HUD
   - Balance damage vs. fire rate

3. **New Effects**: Leverage existing animation keyframes
   - Create new CSS animations if needed
   - Update canvas rendering for visual effects
   - Consider performance impact on mobile

### Performance Considerations

- **Canvas Size**: Fixed 800x600px for consistent gameplay across devices
- **Max Entities**: Monitor concurrent enemies/projectiles
- **Animation**: Use CSS animations for UI, canvas for gameplay
- **Mobile**: Touch-action handling, safe-area insets, viewport meta tags

### Testing Strategy

1. Test gameplay on various screen sizes (mobile, tablet, desktop)
2. Verify touch controls work correctly
3. Test weapon switching and bomb mechanics
4. Check multiplier system progression
5. Validate enemy spawning patterns and boss encounters

## Code Style

- Single HTML file for simplicity and deployment
- Vanilla JavaScript (no frameworks)
- Canvas API for rendering
- CSS Grid/Flexbox for layout
- Comments for complex logic (collision detection, AI patterns)

## Deployment

The game is ready to deploy as a static file:
- Copy `index.html` to any web server
- No build process required
- No dependencies to install
- Works offline (except optional Web Audio API features)

## Future Enhancements

- Additional difficulty levels
- Leaderboard system
- More boss variations
- Power-up system
- Sound effects and music
- Mobile app wrapper

---

**Last Updated**: 2026-10-05  
**Maintained By**: Claude Code
