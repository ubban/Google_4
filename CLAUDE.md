# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/JavaScript application. The game features:
- Player-controlled aircraft with weapon switching mechanics
- Enemy formations and bosses
- Smart bomb mechanics
- Canvas-based rendering
- Touch and mouse controls
- Stage-based progression with bosses

**Repository:** `ubban/google_4`  
**Primary File:** `index.html` (complete game implementation)  
**Language:** HTML5, JavaScript, CSS3

## Project Structure

```
Google_4/
├── index.html          # Main game file (complete implementation)
├── README.md           # Project metadata
└── .git/              # Git repository
```

## Game Architecture

### Core Game Elements

The game is implemented entirely within `index.html` with the following major components:

1. **Canvas Rendering** - Uses HTML5 Canvas for all graphics
2. **Game Loop** - requestAnimationFrame-based update and render cycle
3. **Entity System** - Players, enemies, projectiles, explosions managed as entity objects
4. **Input Handling** - Mouse/touch support for player control
5. **Game States** - Menu, playing, game over, etc.
6. **Audio** - Sound effects for actions and events (optional implementation)

### Key Game Mechanics

- **Weapons System**: Multiple weapons types with switching capability
- **Smart Bombs**: Special attacks available at intervals
- **Enemy AI**: Formation-based movement and attack patterns
- **Boss Encounters**: Stage bosses with unique attack patterns
- **Scoring**: Score tracking with combo multipliers
- **Lives/Health**: Player health management and game over conditions

## Development Guidelines

### Code Style & Standards

- **Single File**: All code is in `index.html` - keep it organized with clear sections
- **Modular Sections**: Use HTML comments to separate:
  - Styles (CSS)
  - Game logic (JavaScript)
  - Game constants (configuration)
- **Variable Naming**: Use clear, descriptive names (e.g., `playerX`, `enemyBullets`, `currentWeapon`)
- **Functions**: Keep functions focused on single responsibilities
- **Performance**: Canvas operations can be expensive - profile before optimizing

### Testing the Game

The game runs directly in any modern browser:

```bash
# Open the file in a browser (works with any HTTP server)
python3 -m http.server 8000
# Then navigate to http://localhost:8000/index.html
```

Or open the file directly in a browser:
- macOS: `open index.html`
- Linux: `xdg-open index.html`
- Windows: `start index.html`

### Making Changes

When modifying the game:

1. **Locate the Code Section**: Find the relevant part in the JavaScript section
2. **Test Changes**: Run the game in a browser after each logical change
3. **Performance Check**: Ensure frame rate remains smooth (target 60 FPS)
4. **Cross-Browser**: Test in multiple browsers if adding new features

### Common Modification Areas

- **Game Balance**: Adjust difficulty constants (enemy spawn rates, projectile speeds, health values)
- **Visuals**: Modify canvas drawing code, colors, particle effects
- **Game Mechanics**: Add new weapons, enemy types, or boss patterns
- **Audio**: Add or modify sound effects
- **UI/HUD**: Adjust on-screen information display

## Code Organization Conventions

When working with this codebase, follow these conventions:

### Constants Section
```javascript
// Define all game constants at the top for easy tweaking
const PLAYER_SPEED = 5;
const ENEMY_SPAWN_RATE = 0.02;
// ...
```

### Entity Objects
Game objects (player, enemies, bullets) use simple object structures:
```javascript
{
  x: number,
  y: number,
  width: number,
  height: number,
  // ... other properties
}
```

### Game State
Track game state in a central object:
```javascript
const gameState = {
  score: 0,
  level: 1,
  lives: 3,
  // ...
};
```

## Git Workflow

- **Branch**: Develop on `claude/inspiring-planck-5qzs24`
- **Commits**: Make clear, descriptive commit messages
- **Push**: Push to the designated branch when changes are complete
- **Attribution**: Include "Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>" in commit messages

## Common Tasks

### Adjust Game Difficulty
- Modify enemy spawn rates at the top constants
- Change enemy projectile speeds or damage
- Adjust player health or weapon damage

### Add a New Enemy Type
1. Create an enemy factory function
2. Add to spawn logic
3. Implement AI behavior in game loop
4. Add drawing code in render function

### Modify Weapon Balance
- Adjust weapon fire rates and damage
- Change projectile speeds
- Modify spread patterns

### Add Visual Effects
- Modify explosion particle systems
- Add screen shake effects
- Adjust colors and gradients

## Performance Notes

- Canvas rendering is generally performant for this game scale
- Limit particle count to maintain 60 FPS
- Use object pooling for frequently created/destroyed entities (bullets, particles)
- Profile with browser DevTools before optimizing

## Browser Compatibility

- Modern browsers (Chrome, Firefox, Safari, Edge)
- Requires HTML5 Canvas support
- Touch controls for mobile/tablet
- Mouse controls for desktop

## Future Enhancement Areas

Potential improvements for the game:
- Persistent high score storage (localStorage)
- Additional stage designs
- More enemy and boss variety
- Power-up system
- Music/background audio
- Pause functionality
- Settings menu (difficulty, sound volume)

## Resources & References

- HTML5 Canvas API: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API
- Game Development Patterns: Consider entity-component-system for larger games
- Performance: Use Chrome DevTools Performance tab to profile

---

Last Updated: 2026-10-01
