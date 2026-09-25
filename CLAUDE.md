# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-page HTML5 application. The game features:
- A player-controlled fighter jet that moves vertically across the screen
- Multiple weapon types with switching mechanics
- Smart bomb mechanics for dealing with dense enemy formations
- Progressive difficulty with stage bosses
- Responsive canvas-based rendering for desktop and mobile

## Repository Structure

```
Google_4/
├── README.md          # Minimal project metadata
├── CLAUDE.md          # This documentation file
└── index.html         # Complete game implementation (single-file)
```

## Technology Stack

- **Language**: JavaScript (ES6+)
- **Rendering**: HTML5 Canvas API
- **Styling**: CSS3 (embedded in HTML)
- **Markup**: HTML5
- **Architecture**: Single-file application with modular game systems

## File Architecture

### index.html
The single source file contains:
1. **HTML Structure**
   - Meta tags for mobile responsiveness and SEO
   - Canvas element for game rendering
   - HUD (Heads-Up Display) overlay for stats

2. **CSS Styling**
   - Dark space-themed color scheme (#05122b, #0d2f5e)
   - Responsive canvas sizing with clamp() for mobile/desktop
   - HUD styling with flexbox layout
   - Gradient background (sky blue fade)

3. **JavaScript Game Engine**
   - Game state management
   - Input handling (keyboard and mouse/touch)
   - Entity systems (player, enemies, projectiles, effects)
   - Collision detection
   - Rendering loop with requestAnimationFrame
   - Audio/particle effects management
   - Boss AI and enemy pattern systems

## Game Systems

### Player System
- Position and velocity management
- Multiple weapon types (primary, secondary)
- Weapon switching mechanics
- Smart bomb deployment
- Health/shield mechanics
- Lives system

### Enemy System
- Enemy spawning with configurable patterns
- Multiple enemy types with different behaviors
- Boss encounters at stage progression
- Enemy projectile systems
- Scoring/point drops

### Weapon System
- Primary weapon (continuous fire)
- Secondary weapon (limited ammo, higher damage)
- Smart bombs (area damage, limited uses)
- Projectile pooling for performance

### Collision Detection
- Bounding box collision for entities
- Damage application and effect triggering

### HUD/UI
- Real-time stat display (score, lives, ammo, etc.)
- Message system for game events
- Game state display (paused, game over, level up)

## Development Guidelines

### Adding Features
1. The game logic is entirely contained in the HTML file
2. New features should maintain the single-file structure or be carefully integrated
3. Test responsive behavior on both desktop and mobile viewports

### Code Organization
- Initialize game constants and configuration at the top
- Define game classes/objects in logical sections
- Keep the render loop and update loop separate for clarity
- Use object pools for frequently created/destroyed entities (projectiles, particles)

### Performance Considerations
- Use requestAnimationFrame for smooth 60fps rendering
- Object pooling for bullets and particles
- Efficient collision detection (check only necessary pairs)
- Canvas state optimization (avoid unnecessary context changes)

### Mobile Compatibility
- Touch event handling in addition to keyboard
- Viewport meta tags ensure proper scaling
- Responsive font sizing using clamp()
- Safe area insets for notched devices

## Canvas Game Loop

The game uses a standard game loop pattern:
```
requestAnimationFrame:
  1. Update game state
  2. Check collisions
  3. Clear canvas
  4. Render entities
  5. Draw HUD
```

## Key Implementation Details

### Meta Tags
- `viewport-fit=cover` for notched device support
- `touch-action: none` to prevent browser touch behaviors
- `cursor: crosshair` for targeting feedback

### Color Scheme
- Background: `#05122b` (dark navy)
- Sky gradient: `#0d2f5e` to `#7fc1ff`
- Text: white with text-shadow for readability
- Theme color: `#0b1d3a`

### Responsive Design
- Max canvas size constrained by viewport
- Flexible HUD layout with CSS Grid/Flexbox
- Font scaling with clamp() function

## Branching Strategy

- Main development branch: `claude/inspiring-planck-dl1crk`
- Changes are committed and pushed to track development
- Attribution includes Claude Haiku 4.5 and session link

## Recent Changes

- **c4dc8f6**: Added Sky Fighter: Raiden-style vertical shoot-em-up game
- **4efdad1**: Initial commit

## Future Enhancement Areas

- Level progression system
- Enemy pattern variations
- Audio effects integration
- Leaderboard/high scores
- Different player ship options
- Power-up system
- Particle effects for explosions
- Screen shake effects

## Notes for Claude Code Users

- The entire game is self-contained in `index.html`
- No external dependencies required
- Can be tested by opening the file directly in a browser
- For development, consider using a local HTTP server for better resource loading
- Mobile testing recommended due to responsive design implementation
