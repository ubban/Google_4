# Codebase Claude.md - Sky Fighter

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS3, and JavaScript. The game runs entirely in the browser on a single `index.html` file that contains all HTML, CSS, and JavaScript code.

- **Language**: HTML5, CSS3, JavaScript (ES6+)
- **Architecture**: Single-file application with embedded styles and scripts
- **Platform**: Web-based (browser)
- **Game Type**: Vertical scrolling shooter with boss battles

## Repository Structure

```
/
├── README.md                 # Project readme with timestamp
├── index.html               # Main game file (all code embedded)
└── Codebase Claude.md       # This documentation file
```

## Game Architecture

### Core Components (inside index.html)

#### 1. **HTML Structure**
- Single container div for the game canvas
- HUD (Heads-Up Display) section with stats and indicators
- Visual effects layers (vignette, flash)

#### 2. **CSS Styling**
- Dark theme with gradient backgrounds
- Responsive design using CSS clamp() for mobile compatibility
- Custom animations (heartBeat, vignettePulse, flashHit, flashBomb)
- Safe area insets for notched devices
- Touch optimization (no tap highlight, no text selection)

#### 3. **JavaScript Game Engine**
The JavaScript portion includes:
- **Canvas Setup**: Game canvas initialization and rendering context
- **Game States**: Menu, playing, paused, boss battle, game over
- **Player Control**: Keyboard and mouse/touch input handling
- **Entity System**: 
  - Player ship with multiple weapons and bombs
  - Enemy types (basic enemies, bosses)
  - Projectiles (player bullets, enemy fire)
  - Visual effects (explosions, impacts)
  - Collectibles (power-ups, score multipliers)
- **Physics**: Movement, collision detection, projectile trajectories
- **Rendering Loop**: RequestAnimationFrame-based game loop
- **Audio**: (If implemented) Sound effects and background music
- **UI/HUD**: Score, health, weapon indicator, bomb counter, multiplier display

## Key Game Mechanics

### Weapons System
- Multiple weapon types player can switch between
- Weapon upgrades via pickups
- Each weapon has unique firing pattern and damage

### Bomb System
- Smart bombs that clear screen of enemies
- Limited quantity (shown in HUD with bomb indicators)
- Can be collected as powerups

### Health & Lives
- Player health shown as heart indicators
- Multiple lives represented by heart count
- Critical status animation when health is low

### Scoring System
- Score multiplier mechanic
- Bonus points for specific actions
- Score persisted during game session

### Boss Battles
- Special stage bosses with unique patterns
- Boss-specific mechanics and attack patterns
- Victory conditions to advance levels

## Development Guidelines

### Code Organization
- Single file with embedded CSS and JavaScript
- Keep HTML, styles, and logic clearly separated by comments/sections
- Use semantic HTML5 elements where possible

### Naming Conventions
- **Classes**: CamelCase (e.g., `PlayerShip`, `EnemyBullet`)
- **Functions**: camelCase (e.g., `updatePlayerPosition()`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_PLAYER_HEALTH`)
- **HTML/CSS IDs**: kebab-case (e.g., `#game-canvas`, `#hud`)

### Performance Considerations
- Minimize DOM manipulation - primarily uses canvas
- Use requestAnimationFrame for smooth 60 FPS rendering
- Pool objects for frequently created/destroyed entities
- Limit simultaneous active entities

### Browser Compatibility
- Modern browser features (ES6+, CSS Grid/Flexbox)
- Touch events for mobile support
- Viewport meta tags for mobile optimization
- CSS safe area insets for notched devices

### Mobile Responsiveness
- Uses CSS clamp() for fluid sizing
- Touch-action: none on canvas to prevent scrolling
- Max-width/max-height constraints for canvas
- Responsive font sizes with vw units

## Important Files & Sections (in index.html)

### meta tags
- Dark color scheme
- Viewport configuration for mobile
- Game description
- Custom favicon

### Canvas Setup
- Game canvas rendering with 2D context
- Gradient background (sky effect)
- Max-width/height for responsive display

### HUD Elements
```html
#hud          - Main HUD container
  .stat       - Individual stat boxes
  #hearts     - Player health indicator
  #bombs      - Bomb counter
  #scoreStat  - Score display
  #weaponStat - Current weapon display
  #multStat   - Score multiplier display
```

### Visual Effects Layers
- `#vignette`: Red vignette effect (damage state)
- `#flash`: Screen flash effects (hit/bomb animations)

## Development Workflow

### Running the Game
1. Open `index.html` in a modern web browser
2. Game loads automatically
3. Controls typically: Arrow keys or WASD for movement, Mouse for aiming, Space/Click for shooting, Keyboard keys for weapon switching

### Testing
- Test on multiple browsers (Chrome, Firefox, Safari, Edge)
- Test on mobile devices (iOS Safari, Chrome Mobile)
- Check responsive behavior at various viewport sizes
- Verify touch controls on mobile

### Building & Deployment
- No build process required - single HTML file
- Deploy to web server or GitHub Pages
- Ensure MIME types are correct (text/html)

## Debugging Tips

### Browser DevTools
- Console for logging and errors
- Performance tab for FPS and rendering analysis
- Canvas debugger for rendering issues
- Network tab for any remote resources

### Common Issues
- Canvas not scaling properly: Check viewport meta tags
- Touch not working: Verify touch-action CSS
- Performance drops: Profile with DevTools Performance tab
- Mobile controls unresponsive: Check touch event handling

## Modification Guidelines

### Adding New Features
1. Keep all code in index.html unless size becomes prohibitive
2. Add new entity types to the entity management system
3. Update HUD display section if adding new stats
4. Test thoroughly on both desktop and mobile
5. Maintain performance with existing mechanics

### Updating Styles
- Use CSS variables for colors if multiple theme support is needed
- Keep responsive sizing with clamp()
- Test animations on lower-end devices
- Maintain dark theme aesthetic

### Extending Game Logic
- Keep the game loop in requestAnimationFrame
- Maintain collision detection performance
- Use object pooling for frequently created objects
- Add state machine for game states (playing, paused, game-over, etc.)

## Performance Targets

- **Frame Rate**: 60 FPS target
- **Load Time**: < 1 second
- **File Size**: Single file < 200KB ideal
- **Mobile Support**: Smooth gameplay on mid-range devices

## Accessibility Considerations

- Use semantic HTML5
- Ensure sufficient color contrast
- Support keyboard controls for desktop
- Provide touch controls for mobile
- Consider colorblind-friendly palettes
- Test with screen readers if UI text is important

## Dependencies

- **None**: Game uses only vanilla HTML5, CSS3, and JavaScript
- No external libraries or frameworks
- No network requests required for core game
- Self-contained and can work offline

## Notes for Claude Code Agents

- This is a single-file web application
- Any changes should maintain the embedded code structure
- Always test changes in a browser before committing
- Keep performance in mind - canvas rendering can be CPU intensive
- Mobile testing is important for user experience
- The file size should be monitored as features are added
- Ensure all new features work without external dependencies
