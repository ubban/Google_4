# Claude.md - Sky Fighter Game Project

## Project Overview
**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game features:
- Player-controlled aircraft with weapon switching mechanics
- Enemy waves and stage bosses
- Smart bomb system
- Score multiplier system
- Health/lives system
- Responsive touch and mouse controls

## Technical Stack
- **Language**: JavaScript (vanilla, no frameworks)
- **Graphics**: Canvas 2D API
- **Deployment**: Single HTML file (`index.html`)
- **Target Platforms**: Desktop browsers, mobile devices (touch-enabled)

## Code Guidelines

### General Principles
- Keep the game self-contained in a single file for easy deployment
- Maintain responsive design that works on both desktop and mobile
- Prioritize gameplay performance and smoothness
- Use clear, descriptive variable and function names

### Game Architecture
The game uses a state-based loop with:
- **Game Loop**: requestAnimationFrame for smooth 60+ FPS
- **Game States**: Menu, Playing, Pause, GameOver, Victory
- **Entity System**: Player, enemies, projectiles, effects
- **Event Handling**: Keyboard, mouse, and touch inputs

### Code Style
- Use descriptive names: `playerHealth` not `pH`
- Keep functions focused and under 50 lines when possible
- Add comments for non-obvious game logic (physics, collision detection, AI)
- Use magic numbers sparingly; define constants at the top of sections
- Maintain consistent indentation (2-space preferred for web code)

### Performance Considerations
- Profile frequently: use browser DevTools to check frame rates
- Minimize DOM operations (game runs entirely on canvas)
- Reuse objects where possible; avoid creating garbage during game loop
- Lazy-load resources and assets when feasible
- Monitor memory usage during extended play sessions

### Testing & Quality
- Test on multiple browsers (Chrome, Firefox, Safari, Edge)
- Test responsive behavior on mobile devices (landscape orientation preferred)
- Verify touch controls work smoothly without lag
- Check audio/visual feedback works as intended
- Ensure game remains playable at 60 FPS

## Common Tasks

### Adding a New Enemy Type
1. Define enemy properties (speed, health, weapon pattern)
2. Add sprite/visual representation
3. Implement AI behavior in the enemy update loop
4. Add to enemy spawn logic in stage definitions
5. Test collision detection and damage

### Adding Weapon Types
1. Define weapon stats (rate of fire, damage, spread pattern)
2. Add weapon visual and sound effects
3. Implement switching mechanism (number keys or UI buttons)
4. Ensure HUD displays current weapon correctly
5. Balance against existing weapons

### Modifying Game Balance
- Adjust enemy health/damage values conservatively
- Test difficulty progression across all stages
- Monitor score multiplier balance
- Ensure difficulty doesn't spike unexpectedly

## Git Workflow
- Commit after completing discrete features (new enemy type, weapon, level)
- Use descriptive commit messages: "Add laser weapon system" not "update"
- Keep commits atomic and logically grouped
- Test locally before committing

## File Organization
```
.
├── index.html (single file containing all code)
├── README.md
└── Claude.md (this file)
```

## Key Sections in index.html
- **Head**: Meta tags, styles, favicon
- **Body**: Game canvas and HUD elements
- **Script Tag**: All game logic including:
  - Constants and configuration
  - Game state management
  - Entity classes (Player, Enemy, Projectile, etc.)
  - Input handling
  - Rendering and physics
  - Audio management

## Debugging Tips
- Use browser console for logging game state
- Canvas DevTools can inspect draw calls and performance
- Check browser DevTools Performance tab for bottlenecks
- Test on actual mobile devices for touch responsiveness

## Future Enhancements
When adding features, consider:
- Progressive gameplay with increasing difficulty
- Special power-up items
- Combo/scoring system refinements
- Accessibility features (colorblind modes, text scaling)
- Save game progress
- Sound volume controls

## Attribution
When making commits:
```
Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
```
