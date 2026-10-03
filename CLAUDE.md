# Sky Fighter: Raiden-Style Vertical Shoot-'Em-Up

## Project Overview

Sky Fighter is a browser-based Raiden-style vertical shoot-'em-up game implemented in vanilla JavaScript with HTML5 Canvas. The game features weapon switching, smart bombs, stage bosses, and progressive difficulty increases.

## Technical Stack

- **Frontend**: HTML5, CSS3, vanilla JavaScript (ES2024+)
- **Rendering**: Canvas 2D API
- **Build System**: None (single-file game)
- **Deployment**: Static HTML file (index.html)

## Repository Structure

```
Google_4/
├── README.md          # Timestamp metadata file
├── index.html         # Main game file (complete game implementation)
├── .git/              # Git history
└── CLAUDE.md          # This documentation file
```

## Game Architecture

### Core Game Loop
The game runs a continuous animation loop that:
1. Updates game state (player position, bullets, enemies, effects)
2. Handles collisions and damage
3. Renders all visual elements
4. Manages HUD (health, bombs, weapon, multiplier)

### Key Game Systems

#### Player System
- **Health**: 5 hearts (displayed as ❤️)
- **Movement**: Mouse or touch-based aiming and movement
- **Weapons**: 
  - Basic shots (standard firepower)
  - Spread weapon (cone of fire)
  - Wave weapon (oscillating shots)
- **Smart Bombs**: Limited-use area-of-effect attacks (3 available)
- **Invulnerability**: Temporary protection after taking damage

#### Enemy System
- **Basic Enemies**: Spawned in waves with increasing difficulty
- **Boss Enemies**: Larger, armored enemies with more health
- **Behavior**: Predictable movement patterns with collision physics
- **Projectiles**: Return fire at the player

#### Scoring System
- Points from destroyed enemies
- Multiplier bonus (increases with consecutive hits, resets on damage)
- Visual feedback via HUD updates

#### Effects & Visuals
- **Explosions**: Particle-based destruction effects
- **Screen Effects**: 
  - Red vignette when critical health (2 or fewer hearts)
  - White flash on hit/bomb detonation
- **Responsive UI**: Scales elements based on viewport size

### Game States
- **Menu/Idle**: Waiting for player input
- **Playing**: Active gameplay
- **Game Over**: Loss condition triggered
- **Victory**: Level/stage complete (if implemented)

## File Structure Details

### index.html
Single comprehensive file containing:

1. **HTML Structure**
   - Viewport meta tags for mobile responsiveness
   - Canvas element for game rendering
   - HUD display elements (health, bombs, weapon, multiplier)
   - Overlay elements (flash effect, vignette)

2. **CSS Styling**
   - Dark blue space-themed color scheme (#05122b, #0d2f5e)
   - Responsive sizing using clamp() for mobile/desktop support
   - Glassmorphic HUD panels with backdrop blur
   - Safe area insets for notched devices
   - Animations for heart pulsing and vignette effects

3. **JavaScript Game Code**
   - Player class: Movement, weapons, health management
   - Enemy class: Spawning, movement patterns, health
   - Projectile class: Bullet physics and rendering
   - Particle system: Explosion effects
   - Game manager: State, collisions, scoring, level progression
   - Input handling: Mouse, touch, keyboard controls
   - Collision detection: AABB (Axis-Aligned Bounding Box)

## Game Controls

- **Mouse/Touch**: Move and aim (player rotates toward cursor)
- **Left Click / Touch Hold**: Fire weapons
- **Space Bar / Touch Tap**: Deploy smart bomb
- **R Key / Double Tap**: Restart game (if implemented)

## Development Guidelines

### Code Organization Principles
- Single-file architecture keeps dependencies minimal
- All game logic contained in one index.html file
- Class-based structure for entities (Player, Enemy, Projectile)
- Functional approach for game loop and state management

### Adding New Features

#### Weapon Types
Add to weapon switch logic:
```javascript
case 'newWeapon':
  // Modify shot pattern and damage in player.shoot()
```

#### Enemy Types
Create new Enemy subclass or variation:
```javascript
class SpecialEnemy extends Enemy {
  // Override movement, health, drops
}
```

#### Visual Effects
Add to particle system or create new effects:
```javascript
particles.push(new Particle(x, y, type));
```

### Performance Considerations
- Canvas-based rendering (no DOM overhead)
- Efficient collision detection (spatial partitioning possible)
- Particle pooling recommended for scale
- Mobile optimization: Hardware acceleration enabled

## Deployment

### Play Online
Simply open `index.html` in a modern web browser.

### Mobile
Game is touch-optimized:
- Responsive viewport scaling
- Touch controls for movement and shooting
- Safe area awareness for notched devices

## Browser Support

- **Modern Browsers**: Chrome, Firefox, Safari, Edge (latest 2+ versions)
- **Mobile**: iOS Safari, Android Chrome
- **Minimum Requirements**: HTML5 Canvas, ES2024 JavaScript support

## Known Limitations & Future Improvements

- Single level/infinite waves (no stage progression yet)
- No persistent high score storage
- No difficulty selection menu
- Could optimize particle rendering with requestAnimationFrame pooling
- Boss variety could be expanded
- Mobile control scheme could use joystick UI

## Git Workflow

- **Branch**: `claude/inspiring-planck-dgphyn`
- **Primary Branch**: Main repository branch
- All development happens on the designated feature branch
- Commit format: Clear, descriptive messages with feature/fix prefixes

## Debugging Tips

1. **Performance**: Check FPS in browser DevTools, monitor particle count
2. **Collision Issues**: Visual bounding boxes can be rendered for debugging
3. **Weapon Balance**: Adjust damage/fire rate in Player.shoot() method
4. **Enemy Difficulty**: Modify wave structure and spawn timing in game loop

## Attribution

- Project: Sky Fighter - Raiden-style vertical shoot-'em-up
- Implementation: JavaScript/Canvas 2D
- Date Started: 2026-02-23
