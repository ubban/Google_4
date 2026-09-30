# Sky Fighter - Codebase Guide

## Project Overview
Sky Fighter is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features player-controlled aircraft, enemies, weapons, and boss battles with a responsive design supporting both desktop and mobile platforms.

**Repository:** ubban/google_4
**Type:** Browser-based game (HTML5 Canvas)
**Architecture:** Single-file application with embedded HTML, CSS, and JavaScript

## File Structure
```
index.html (1669 lines)
├── Meta tags & viewport setup
├── Styles
│   ├── Root color scheme (dark theme)
│   ├── Canvas & HUD styling
│   ├── Game UI elements
│   └── Responsive layout (clamp() for scaling)
├── JavaScript (ES6 Classes)
│   ├── Game state & configuration
│   ├── Entity classes (Player, Enemy, Projectile, etc.)
│   ├── Manager classes (GameManager, InputManager, SoundManager)
│   └── Game loop & event handlers
└── HTML
    └── Canvas container & HUD elements
```

## Key Classes

### Core Game Management
- **GameManager** - Main game state, rendering loop, boss spawning
- **InputManager** - Keyboard/touch input handling for cross-platform support
- **SoundManager** - Audio effects and background music management

### Entity Classes
- **Player** - Controlled aircraft with weapon system, health, movement bounds
- **Enemy** - AI-controlled enemies with movement patterns and health
- **Projectile** - Bullets and missiles with collision detection
- **Particle** - Visual effects (explosions, smoke)
- **Boss** - Special enemy type with boss-specific behavior

### UI & Configuration
- **HUD** - Head-up display (score, health, wave info)
- **WaveConfig** - Enemy wave definitions and progression
- **SoundConfig** - Sound effect and music definitions

## Coding Conventions

### Style & Naming
- Use camelCase for variables and methods
- Use PascalCase for class names
- Prefix private fields with underscore (e.g., `_health`)
- Use meaningful names that clearly indicate purpose

### Class Pattern
```javascript
class EntityName {
  constructor(x, y, options) {
    this.x = x;
    this.y = y;
    // initialization
  }
  
  update(deltaTime) {
    // game logic per frame
  }
  
  draw(ctx) {
    // canvas rendering
  }
  
  // getter/setter pattern
  get isAlive() { return this.health > 0; }
}
```

### Canvas Drawing
- Store canvas context in variables (avoid repeated queries)
- Use `ctx.save()` and `ctx.restore()` for state management
- Apply transformations before drawing primitives
- Clear canvas at start of render loop with `clearRect()`

### Game Loop
- Fixed timestep using `deltaTime` parameter (seconds)
- Call `update()` then `draw()` for each entity
- Handle collisions after entity updates, before rendering

### Input Handling
- Cross-platform support: keyboard + touch events
- Mouse for desktop (cursor following)
- Touch for mobile (primary touch point tracking)
- Normalize input into `inputState` object

## Development Guidelines

### Adding Features
1. Create new class in script section following existing pattern
2. Add update() and draw() methods if entity is rendered
3. Register in appropriate manager (GameManager, InputManager, etc.)
4. Test on both desktop and mobile viewports
5. Update this Claude.md with new class descriptions

### Modifying Game Balance
- Enemy stats: damage, health, speed in Enemy class constructor
- Wave configuration: WaveConfig array defines progression
- Weapon properties: projectile damage, fire rate in Player class
- Particle effects: visual feedback without affecting gameplay

### Testing Checklist
- [ ] Game runs in modern browsers (Chrome, Firefox, Safari, Edge)
- [ ] Mobile responsiveness verified (landscape & portrait)
- [ ] Canvas scales properly with window resize
- [ ] Touch controls work on mobile devices
- [ ] Audio plays without errors (graceful fallback if unavailable)
- [ ] Boss wave appears after normal enemy waves
- [ ] Collision detection is accurate
- [ ] Performance remains smooth at 60fps

### Performance Considerations
- Canvas rendering is the main performance bottleneck
- Reuse object pools for frequently created entities (projectiles)
- Batch canvas operations to minimize context switches
- Throttle sound effect triggering to avoid audio saturation
- Monitor frame rate on target devices

## Deployment
- Single file deployment: just commit and serve `index.html`
- No external dependencies (self-contained)
- Works on any static hosting (GitHub Pages, Netlify, etc.)
- Mobile-friendly with viewport and touch-action settings

## Common Tasks

### To add a new weapon type
1. Add weapon configuration constant
2. Modify Player.shoot() method with new weapon case
3. Update HUD to show weapon selection
4. Test collision and damage values

### To adjust difficulty
1. Modify WaveConfig enemy counts and properties
2. Adjust boss health and damage values
3. Change projectile properties
4. Test with fresh game session

### To add visual effects
1. Create Particle class instances in collision/damage events
2. Implement draw() for visual representation
3. Add update() logic for animation
4. Remove when animation completes

## Notes for Claude Code
- This is a single-file application; all changes go to `index.html`
- Maintain embedded CSS and JavaScript structure
- Preserve responsive design patterns (clamp() for sizing)
- Keep canvas rendering optimized
- Test changes in browser before committing
- Use meaningful commit messages describing game logic changes
