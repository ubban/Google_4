# Claude.md - Sky Fighter Game Project

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The game features:
- Player-controlled aircraft with weapon switching
- Enemy waves and boss battles
- Smart bomb mechanics
- Responsive controls (keyboard, mouse, and touch)
- Dynamic scoring system

## Project Structure

```
Google_4/
├── index.html          # Main game file (HTML + CSS + JS combined)
├── README.md           # Project metadata
└── Claude.md           # This file - development guidelines
```

## Development Guidelines

### Code Style
- Use vanilla JavaScript (no frameworks required)
- Follow ES6+ conventions
- Keep functions focused and well-named
- Use const/let, avoid var
- Add comments only for non-obvious logic or workarounds

### Game Architecture
The index.html file contains:
1. **HTML Structure**: Canvas element and HUD (heads-up display)
2. **CSS Styling**: Dark theme with Raiden-inspired visual design
3. **JavaScript Game Logic**:
   - Game state management (score, lives, waves, bosses)
   - Player mechanics (movement, weapon firing, smart bombs)
   - Enemy AI (basic movement patterns, firing)
   - Collision detection
   - Rendering loop using requestAnimationFrame

### Key Game Systems

#### Player
- Position and health tracking
- Multiple weapon types with switching
- Smart bomb capability (limited use)
- Death and respawn logic

#### Enemies
- Wave-based spawning system
- Basic AI for movement and firing
- Boss encounters at stage ends
- Projectile management

#### Collision Detection
- Player vs enemy
- Player vs enemy projectiles
- Weapons vs enemies
- Boundary checking

#### Scoring
- Points for destroying enemies
- Combo multipliers
- Boss bonuses
- Lives system

## Testing

### Manual Testing
1. Open `index.html` in a web browser
2. Test core mechanics:
   - Arrow keys or WASD for movement
   - Click or spacebar to fire
   - B key for smart bombs
   - Mouse movement alternative controls
3. Verify across devices:
   - Desktop (keyboard + mouse)
   - Mobile (touch controls)
   - Different screen sizes (responsive design)

### Test Scenarios
- Player movement and boundaries
- Weapon switching and firing
- Enemy spawning and AI behavior
- Collision detection (all types)
- Smart bomb usage limits
- Level progression and boss encounters
- Score calculation
- Game over and restart

## Build & Deployment

No build process required. The game is a single HTML file that can be:
- Opened directly in a browser
- Served via any static file server
- Deployed to any web hosting platform

### Browser Requirements
- HTML5 Canvas support
- ES6 JavaScript support
- Recommended: Chrome, Firefox, Safari, Edge (modern versions)

## Code Conventions

### Naming
- Use camelCase for variables and functions
- Use PascalCase for class/constructor names
- Use UPPER_SNAKE_CASE for constants
- Descriptive names: `updatePlayerPosition()` not `upd()`

### Structure
- Initialize game state at the top
- Separate game logic into logical sections
- Use functions to compartmentalize behavior
- Keep the game loop (requestAnimationFrame) clean and focused

### Comments
- Explain WHY, not WHAT (good code is self-documenting)
- Use comments for:
  - Complex algorithms
  - Non-obvious game mechanic interactions
  - Workarounds and edge cases
  - Business logic that isn't obvious from the code

## Performance Considerations

- Use requestAnimationFrame for smooth 60 FPS rendering
- Manage object pooling for bullets and particles if needed
- Minimize DOM operations (all rendering on canvas)
- Profile using browser DevTools if performance issues arise

## Common Modifications

### Adding New Weapons
1. Define weapon properties (damage, fire rate, spread)
2. Add rendering logic in the main game loop
3. Implement collision detection for the new weapon type
4. Update weapon switching logic

### Adding New Enemy Types
1. Create enemy properties (speed, health, weapon)
2. Implement AI behavior in the update loop
3. Add rendering logic
4. Update spawn patterns and boss mechanics

### Adjusting Game Balance
- Enemy health/damage values
- Weapon power levels
- Score multipliers
- Bomb recharge rates
- Wave difficulty progression

## Debugging Tips

- Use `console.log()` to track game state
- Browser DevTools Canvas inspection for rendering issues
- Check mouse/touch event listeners if controls feel unresponsive
- Monitor FPS using Chrome DevTools Performance tab
- Test across different browsers for compatibility

## Future Enhancements

Potential improvements for consideration:
- Power-up system (shields, weapon upgrades, speed)
- Multiple difficulty levels
- Leaderboard/high score persistence
- Additional visual effects and sound
- More boss encounters
- Stage selection/unlocking system
- Mobile app deployment

---

This Claude.md was generated on 2026-09-19 to provide context and guidelines for Claude Code development on the Sky Fighter project.
