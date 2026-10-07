# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, CSS3, and JavaScript. Players control a fighter aircraft, navigate waves of enemies, collect power-ups, switch weapons, and defeat stage bosses.

## Project Structure

```
Google_4/
├── README.md           # Basic project info
├── Claude.md          # This file - AI agent documentation
├── index.html         # Complete game implementation (HTML + CSS + JS)
└── .git/              # Version control
```

## Technical Stack

- **HTML5 Canvas**: Core graphics rendering engine
- **Vanilla JavaScript**: No frameworks or dependencies
- **CSS3**: Styling, animations, and responsive layout
- **Single-file architecture**: All code embedded in index.html

## Key Game Components

### Canvas & Graphics
- Canvas ID: `#gameCanvas`
- Resolution: Responsive, scales to fill viewport
- Background: Gradient from dark blue (#0d2f5e) to light blue (#7fc1ff)
- Visual effects: Vignette overlay, screen flash animations

### Game States
1. **START**: Menu screen before gameplay
2. **PLAYING**: Active gameplay
3. **GAME_OVER**: Game over screen
4. **VICTORY**: Boss defeated, next stage ready

### Player Character
- Position: Center-bottom of screen
- Health: 3 hearts (visible in HUD)
- Movement: Keyboard controls (arrow keys)
- Weapons: Multiple types with limited ammo
- Smart bombs: Limited use, devastating area damage
- Hitbox: Small for dodging

### Enemy System
- Wave-based spawning
- Enemy types: Regular enemies with varied patterns
- Boss enemies: Larger, multi-phase battles
- Projectile patterns: Bullet patterns and waves

### HUD Elements
Located at top-center of screen:
- Heart counter (#hearts): Displays remaining lives
- Bomb counter (#bombs): Shows available smart bombs
- Weapon stat (#weaponStat): Current weapon info
- Multiplier stat (#multStat): Score multiplier display

### Game Physics
- Bullet speed and direction calculations
- Collision detection between player/enemies/projectiles
- Screen boundary collision handling
- Smooth movement interpolation

## Gameplay Mechanics

### Controls
- **Arrow Keys**: Move player left/right/up/down
- **Z Key**: Fire weapon
- **X Key**: Deploy smart bomb

### Weapons
- Default weapon: Standard bullets
- Alternative weapons: Acquired from pickups
- Each weapon has unique firing pattern and ammo

### Scoring
- Enemy kills: Base points + multiplier bonus
- Weapon pickups: Add ammunition
- Multiplier: Increases with consecutive hits, resets on damage taken

### Difficulty
- Progressive waves increase in intensity
- Boss fights for stage completion
- Health management is critical

## Important Code Sections

### Game Loop
- Canvas rendering at 60 FPS
- Update game state each frame
- Render all entities (player, enemies, bullets, effects)

### Collision Detection
- Player-to-enemy collision
- Player-to-projectile collision
- Bullet-to-enemy collision
- Uses bounding box calculations

### Enemy Wave System
- Predefined wave patterns
- Boss encounters at stage end
- Enemy spawning positions and timing

### UI/HUD System
- Real-time stat updates
- Animation effects (heart beat, vignette pulse)
- Responsive font sizing with clamp()

## Development Notes for AI Agents

### When Making Changes

1. **Canvas Rendering**: Test visual changes by running the game in browser
2. **Game Balance**: Adjust enemy health, spawn rates, weapon damage carefully
3. **Performance**: Monitor canvas operations, avoid excessive redraws
4. **Responsive Design**: Use CSS clamp() for mobile-friendly scaling
5. **Animations**: Test vignette and flash effects at various screen sizes

### Testing Checklist

- [ ] Game starts and renders correctly
- [ ] Player movement responds to keyboard input
- [ ] Enemy waves spawn and behave as expected
- [ ] Collision detection works accurately
- [ ] HUD displays correct stats
- [ ] Weapons fire and reload properly
- [ ] Boss fight is challenging but fair
- [ ] Game over and victory states display correctly
- [ ] Mobile/responsive scaling works on different devices

### Common Modifications

- **Adjust difficulty**: Modify enemy spawn rate, speed, health in wave definitions
- **Weapon balance**: Change ammo amounts, firing speed, damage multiplier
- **Game speed**: Adjust frame-time constants and entity speed values
- **Visuals**: Modify gradient colors, particle effects, HUD styling
- **Game duration**: Adjust number of waves, boss difficulty

## Browser Compatibility

- Requires HTML5 Canvas support
- ES6 JavaScript features used throughout
- Mobile touch support for responsive gameplay
- Tested on modern browsers (Chrome, Firefox, Safari, Edge)

## Building & Running

No build process required:
1. Open `index.html` in a web browser
2. Game starts automatically
3. Use keyboard controls to play

## Resources & References

- HTML5 Canvas API for graphics
- Keyboard event handling for controls
- CSS animations for visual effects
- Responsive design principles

## Notes

- All game code is self-contained in index.html
- No external dependencies or libraries
- Single-player game with local scoring
- Optimized for desktop and mobile browsers
