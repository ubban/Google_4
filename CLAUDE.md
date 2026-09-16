# Sky Fighter - Claude Development Guidelines

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game written in vanilla JavaScript and HTML5 Canvas. The entire game is contained in `index.html` with inline CSS and JavaScript.

- **Type**: Browser-based arcade game
- **Engine**: HTML5 Canvas
- **Language**: Vanilla JavaScript (ES6+)
- **Framework Dependencies**: None
- **Platform**: Web browsers (desktop and mobile)

## Core Game Features

- **Player Control**: Vertical-scrolling spaceship with WASD/Arrow keys or mouse/touch
- **Weapons System**: Multiple weapon types that can be switched mid-game
- **Power-ups**: Health, weapon upgrades, smart bombs
- **Enemy Waves**: Progressive difficulty with various enemy types
- **Boss Fights**: Stage bosses with unique attack patterns
- **HUD Display**: Score, health, current weapon, wave/stage info

## Architecture

The game uses a single-file architecture with these main components:

### Game Loop
- Canvas rendering and update cycle
- Input handling (keyboard, mouse, touch)
- Collision detection
- Entity management (player, enemies, projectiles, particles)

### Entity System
- **Player**: Spaceship controlled by the player
- **Enemies**: Various enemy types with different behaviors
- **Projectiles**: Player bullets and enemy attacks
- **Particles**: Visual effects (explosions, trails)
- **UI Elements**: Score, health, weapon indicators

### Game State Management
- Wave/stage progression
- Health and scoring system
- Current weapon selection
- Game over and victory conditions

## Coding Standards

### Style Guidelines
- Use camelCase for variables and functions
- Use UPPER_CASE for constants
- Keep functions focused and under 100 lines where practical
- Add comments for non-obvious logic (attack patterns, collision tweaks, etc.)
- No external dependencies or libraries

### Canvas Rendering
- Use ctx (canvas context) for all drawing operations
- Clear canvas each frame with appropriate background
- Group related drawing operations together
- Keep viewport-related calculations organized

### Game Logic
- Separate entity update logic from rendering
- Keep collision detection modular
- Use consistent coordinate system (0,0 at top-left)
- Define magic numbers as constants at the top of relevant sections

## Modification Guidelines

### Adding New Features

#### New Enemy Type
1. Define enemy properties (speed, health, attack pattern)
2. Add spawn logic to enemy wave generation
3. Implement update() method for movement and attacks
4. Implement draw() method for rendering
5. Handle collision and damage logic

#### New Weapon
1. Add weapon properties to the weapons collection
2. Implement firing logic in player's attack method
3. Define bullet behavior (speed, damage, spread)
4. Add visual feedback (trails, effects)
5. Include in weapon selection UI

#### New Power-up
1. Define power-up type and visual appearance
2. Add spawn logic to enemy/stage events
3. Implement collision detection and application logic
4. Update HUD to show effect
5. Include duration/cooldown if applicable

### Balancing

When adjusting difficulty:
- **Damage Values**: Stored in entity definitions, easy to tweak
- **Enemy Spawn Rates**: Controlled by wave generation logic
- **Player Speed/Firepower**: Defined in player initialization
- **Health Values**: Easy to adjust per entity type
- **Score Multipliers**: Located in score calculation sections

Keep playtesting in mind - changes can have cascading effects on game feel.

## Important Notes

### Performance Considerations
- Canvas rendering is efficient for this scale
- Collision detection is brute-force; optimize if adding hundreds of entities
- Particle effects should be pooled or limited to prevent slowdown
- Mobile performance: test on real devices due to rendering differences

### Browser Compatibility
- Requires HTML5 Canvas support
- Uses modern JavaScript (ES6+)
- Mobile touch handling is supported
- Responsive viewport with clamp() for scaling

### Testing
- Test on both desktop and mobile devices
- Test keyboard (WASD, Arrows) and mouse controls
- Test touch controls on tablets
- Verify boss fights don't become impossible
- Check score calculation and leaderboard accuracy

## Common Tasks

### Debugging
1. Open browser DevTools (F12)
2. Check Console tab for errors
3. Use breakpoints on game loop for stepping through logic
4. Log entity positions and velocities during updates

### Adding Debug Display
- Add temp overlay to show hitboxes: `ctx.strokeRect(x, y, w, h)`
- Log entity counts each frame: `console.log(enemies.length, projectiles.length)`
- Display collision boxes for troubleshooting

### Profiling
- Use browser DevTools Performance tab
- Record game session and analyze frame drops
- Check for garbage collection pauses
- Profile specific rendering or update sections

## File Organization

The entire codebase is in `index.html`:
- `<head>`: Meta tags, styling (CSS)
- `<body>`: Canvas container
- `<script>`: All game logic (JavaScript)

When making large changes, consider adding comments to separate logical sections:
- Constants and configuration
- Entity class definitions
- Game state
- Game loop
- Input handling
- Rendering functions

## Quality Standards

- **No external dependencies**: Keep it vanilla
- **No code duplication**: Extract common patterns to functions
- **Clear variable names**: Avoid single-letter vars except in loops
- **Performance**: Monitor FPS, target 60 FPS on most devices
- **Mobile-first**: Design for touch as the primary input
- **Accessibility**: Consider colorblind modes for future enhancement

## Future Enhancement Ideas

- Leaderboard system
- Multiple difficulty levels
- Additional weapon types (laser, missiles, spread shot)
- Boss unique attack patterns
- Particle effects for impacts
- Sound effects and background music
- Settings/controls configuration menu
- Achievements or progression system

---

**Last Updated**: 2026-09-16  
**Maintained by**: Claude Code Agent
