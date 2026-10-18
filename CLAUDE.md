# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a browser-based vertical shoot-'em-up game inspired by the classic arcade game Raiden. Players pilot a fighter jet to defend against waves of enemies, collect power-ups, deploy smart bombs, and defeat stage bosses. The entire game is implemented as a single-file HTML/CSS/JavaScript application.

### Key Features
- **Weapon System**: Switch between multiple weapon types with distinct firing patterns
- **Power-ups**: Collect items to increase firepower, add shields, and gain smart bombs
- **Smart Bombs**: Deploy area-of-effect attacks to clear the screen
- **Boss Encounters**: Multi-phase boss battles at the end of each stage
- **Health System**: Lives-based health with visual feedback (hearts)
- **Score & Multiplier**: Points system with multiplier bonuses
- **Mobile & Desktop Support**: Touch and mouse/keyboard controls
- **Responsive Design**: Runs on various screen sizes using viewport-relative units

## Technology Stack

- **Language**: HTML5, CSS3, vanilla JavaScript (ES6+)
- **Canvas API**: 2D game rendering with requestAnimationFrame
- **No external dependencies**: Pure vanilla implementation
- **Browser APIs**: Touch events, keyboard events, canvas, requestAnimationFrame

## Code Organization

### Single-File Architecture
The entire game (1669 lines) is contained in `index.html` with three main sections:
1. **HTML Structure**: Game container and HUD elements
2. **CSS Styling**: Game canvas, UI panels, animations, responsive design
3. **JavaScript Game Engine**: All game logic in `<script>` tag

### Key JavaScript Classes & Systems

#### Core Game Manager
- **GameEngine**: Main game loop controller using requestAnimationFrame
- **State Management**: Active game state, pause/unpause, game over, level progression

#### Entity Systems
- **Player**: Player spaceship with position, health, weapons, movement
- **Enemy**: Base enemy class with AI, position, health, collision detection
- **Boss**: Boss entities with multi-phase patterns, health bars, special attacks
- **Projectile**: Player shots and enemy bullets
- **Particle**: Visual effects (explosions, hits, item pickups)
- **PowerUp**: Collectible items (weapons, shields, bombs)

#### Game Mechanics
- **Collision Detection**: AABB (Axis-Aligned Bounding Box) collision system
- **Weapon System**: 
  - Default shot (basic bullets)
  - Spread weapon (fan-shaped pattern)
  - Missile weapon (homing projectiles)
  - Laser weapon (continuous energy beam)
- **Enemy AI**: Patterns for movement and firing based on enemy type
- **Score System**: Points from kills, multiplier bonuses from pickups

#### Rendering & Animation
- **Canvas Rendering**: 2D drawing primitives (rectangles, circles, paths)
- **Particle Effects**: Explosion animations, visual feedback for hits
- **HUD Updates**: Score, health, weapon display, multiplier counter
- **Vignette Effects**: Red screen tint when taking damage
- **Flash Effects**: Screen flash on hits and bomb detonation

#### Input Handling
- **Keyboard**: WASD for movement, mouse for aiming, spacebar for firing
- **Mouse**: Click to fire, mouse position for aiming
- **Touch**: Touch drag for movement, tap to fire (on mobile)
- **Mobile Gamepad**: Optional touch-based joystick area

## Game Mechanics

### Player Controls
- **Movement**: WASD keys or arrow keys to move (8-directional)
- **Aiming**: Mouse cursor or finger position
- **Fire**: Click/tap or hold spacebar
- **Weapon Switch**: Number keys (1-4) or by collecting weapon pickups
- **Smart Bomb**: B key or collected bomb items (max 3)
- **Pause**: ESC key

### Combat Flow
1. **Player Movement**: Avoid incoming fire while maintaining position
2. **Weapon Selection**: Pick optimal weapon for current enemy formation
3. **Targeting**: Aim manually or via cursor/touch position
4. **Damage Feedback**: Screen flash, vignette pulse, enemy knockback
5. **Death/Recovery**: Respawn mechanics, life counter display

### Progression System
- **Stages/Levels**: Each stage has unique enemy waves
- **Boss Battles**: Multi-phase encounters with pattern-based attacks
- **Wave Difficulty**: Enemy density and firing rate increase per wave
- **Multiplier**: Bonus multiplier for consecutive hits without taking damage

## File Structure

```
Google_4/
├── index.html          # Complete game implementation (HTML + CSS + JS)
├── README.md          # Project metadata
└── CLAUDE.md          # This documentation file
```

## Development & Testing

### Running the Game
1. Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge)
2. Game starts immediately with initial stage
3. Use keyboard/mouse or touch controls
4. Close the browser tab to quit

### Browser Compatibility
- **Modern Browsers**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Canvas API**: Required for 2D graphics
- **Touch Events**: For mobile device support
- **requestAnimationFrame**: For smooth animations

### Performance Considerations
- **Canvas Size**: Dynamically scales to window size with max-width/max-height
- **Entity Pooling**: Reuse projectile and particle objects to reduce GC pressure
- **Rendering**: Draw order matters (background → entities → UI)
- **Collision Detection**: AABB system scales with entity count

## Development Guidelines

### Code Style
- **Naming**: camelCase for variables/functions, PascalCase for classes
- **Comments**: Document complex algorithms and game rules, not obvious code
- **Functions**: Keep methods focused on single responsibilities
- **Constants**: Use CONST_CASE at module scope for magic numbers and game parameters

### Adding New Features
1. **New Enemy Type**: Extend Enemy class with new AI pattern in update()
2. **New Weapon**: Add to weapon constants, implement fire() logic in Player
3. **New Boss**: Create Boss subclass with phase transitions
4. **Visual Effects**: Add particles/animations in particle system
5. **Balance Changes**: Adjust constants (damage, health, speed, spawn rates)

### Testing Checklist
- [ ] Game starts and runs without errors
- [ ] Player movement (all 8 directions) works smoothly
- [ ] Weapon firing and switching works
- [ ] Collision detection works (bullets hit enemies)
- [ ] Damage/death mechanics trigger correctly
- [ ] Enemies spawn and follow patterns
- [ ] Power-ups spawn and are collectible
- [ ] Boss phases transition correctly
- [ ] Score/multiplier updates display properly
- [ ] Mobile touch controls work (if modified)
- [ ] Performance is smooth at 60 FPS

### Common Tweaks
```javascript
// Game balance parameters (search in index.html for these)
const PLAYER_SPEED = 4;           // Spaceship movement speed
const PLAYER_HEALTH = 3;          // Starting lives
const WEAPON_FIRE_RATE = {        // Shots per second by weapon type
  default: 8,
  spread: 5,
  missile: 3,
  laser: 20
};
const BOSS_HEALTH = [100, 150];   // Health per stage boss
const SPAWN_RATE = 2;             // Enemies spawned per wave
```

## Future Enhancement Ideas
- [ ] Level/stage system with progression
- [ ] Additional weapon types (plasma, chains, etc.)
- [ ] Enemy variety and special attack patterns
- [ ] Audio system (background music, SFX)
- [ ] Save/high score system with localStorage
- [ ] Tutorial/help screen
- [ ] Visual polish (particle effects, screen shake)
- [ ] Difficulty settings (easy/normal/hard)
- [ ] Leaderboard/online multiplayer (future)

## Troubleshooting

### Game not rendering
- Ensure browser supports HTML5 Canvas
- Check browser console for JavaScript errors
- Verify index.html is being served correctly

### Lag or low FPS
- Close other browser tabs
- Check device performance (GPU usage)
- Reduce particle effect count if needed
- Profile with browser DevTools (F12)

### Controls not working
- Ensure browser window has focus
- Check if keyboard/touch events are being captured
- Verify input handler functions are attached to window

### Mobile issues
- Ensure viewport meta tag is present
- Test on various device sizes
- Check touch event handling in input system

## Session Notes

This document serves as a reference for Claude Code sessions working on the Sky Fighter project. It should be updated as new features are added or architectural changes are made.

Last Updated: 2026-10-18
Project Repository: ubban/google_4
Active Branch: claude/inspiring-planck-5skiw2
