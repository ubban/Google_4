# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, Canvas 2D API, and JavaScript. The game is delivered as a single-file HTML application (`index.html`) with no external dependencies.

**Genre**: Vertical Scroll Shooter  
**Platform**: Web (HTML5 Canvas)  
**Architecture**: Single-file monolithic application

## Game Features

### Core Gameplay
- **Player Aircraft**: Controllable player ship that moves horizontally across the canvas
- **Multiple Weapon Systems**: Vulcan, Laser, Spread, Homing missiles with upgradeable levels
- **Smart Bombs**: Limited-use powerful area-of-effect attacks
- **Health System**: Lives-based (hearts) with health bar visualization
- **Multiplier System**: Score multiplier increases with consecutive hits, resets on damage
- **Difficulty Levels**: Easy, Normal, Hard with different enemy spawn patterns and behaviors

### Game Progression
- **Stages**: Progressive waves of enemies with increasing difficulty
- **Boss Encounters**: Stage-end boss battles with unique attack patterns
- **Scoring**: Dynamic scoring based on enemy type, multiplier, and difficulty
- **Best Score Tracking**: Persistent high score storage in localStorage

### Visual & Audio
- **Screen Shake**: Tactical screen movement on significant events (hits, bombs)
- **Flash Effects**: White flash overlay for impact feedback
- **Red Vignette**: Critical health state indicator with pulsing animation
- **Audio**: Toggle-able background music and sound effects
- **Accessibility**: Reduced motion mode that disables screen shake, flash, and pulse effects
- **Responsive Design**: Adaptive UI and canvas scaling for different screen sizes

## File Structure

```
Google_4/
├── index.html          # Single-file game application (1669 lines)
├── README.md          # Project readme
└── CLAUDE.md          # This documentation file
```

## Technical Architecture

### Canvas Setup
- Game runs on HTML5 Canvas 2D context
- Responsive sizing with `max-width: 100%` and `max-height: 100%`
- Fixed aspect ratio maintained through viewport configuration
- Touch support with `touch-action: none` for mobile devices

### Main Game Objects

#### Player
- Position-based collision detection with enemies and projectiles
- Multiple weapon types with independent firing mechanisms
- Health/lives system with visual heart display
- Smart bomb charges (maximum 3)
- Weapon upgrade system (levels 1-5)

#### Enemies
- Various enemy types with different behaviors and point values
- Firing patterns and movement algorithms
- Collision detection for damage
- Spawning waves controlled by stage progression
- Boss enemies with larger health pools and special attacks

#### Weapons
- **Vulcan**: Rapid continuous fire, basic damage
- **Laser**: High single-damage shots, medium rate
- **Spread**: Multiple shots in an arc pattern
- **Homing**: Self-guided missiles that track enemies

#### Projectiles
- Player projectiles with collision detection
- Enemy projectiles with screen-wrapping behavior
- Particle effects and animation rendering

### Game Loop
- RequestAnimationFrame-based update/render cycle
- Physics-based movement with velocity vectors
- Collision detection using bounding box calculations
- State machine: Menu → Playing → Paused → Game Over

## Key Game Mechanics

### Weapon System
- Players cycle through 4 weapon types (Vulcan → Laser → Spread → Homing)
- Each weapon upgrades from level 1-5, improving fire rate and damage
- Weapon pickups automatically upgrade current weapon or switch if already max level

### Scoring & Multiplier
- Base score varies by enemy type
- Multiplier increases with consecutive hits without taking damage
- Multiplier resets to 1x on player hit
- Total score = base_score × difficulty_factor × multiplier

### Boss Encounters
- One boss per stage at predetermined spawn point
- Boss appears after defeating all wave enemies
- Boss defeat triggers stage completion and next wave

### Difficulty Scaling
- **Easy**: Reduced enemy count, slower spawn rate, lower boss health
- **Normal**: Standard gameplay balance
- **Hard**: Increased enemy difficulty, faster attacks, more numerous projectiles

## Game States & Screens

1. **Main Menu**: Difficulty selection and start button
2. **Game Playing**: Active gameplay with HUD (score, lives, weapon, multiplier)
3. **Paused**: Pause overlay with resume/quit options
4. **Game Over**: Final score display, high score comparison, restart option
5. **Stage Clear**: Between-stage screen (auto-transitions)

## HUD Elements

- **Score**: Current run score (yellow text)
- **Best**: High score from localStorage
- **Multiplier (x)**: Orange text, increases with hits
- **Weapon**: Current weapon name and level (lime green)
- **Hearts**: Life indicators (❤️ for alive, grayed for lost)
- **Bombs**: Smart bomb charges (💣 filled = available, unfilled = used)
- **Control Buttons**: Mute (🔊), Motion Toggle (🎆), Pause (⏸)

## Control Schemes

### Keyboard/Mouse
- Arrow Keys or WASD: Move player
- Space/Click: Fire weapon
- B: Drop smart bomb
- P: Pause game

### Touch/Mobile
- Swipe/touch-drag: Move player
- Tap: Fire weapon
- Two-finger tap: Smart bomb (alternative: B key or button)

## Performance Considerations

- Single canvas for all rendering (no layers)
- Efficient collision detection using bounding boxes
- Object pooling for projectiles to reduce GC pressure
- RequestAnimationFrame for smooth 60fps target
- Mobile optimization: reduced motion mode disables expensive animations

## Data Persistence

**LocalStorage Keys**:
- `skyFighterBest`: Best score
- `skyFighterVolume`: Audio volume setting (0-1)
- `skyFighterReducedMotion`: Reduced motion preference (boolean)

## Development Guidelines

### Adding New Enemies
1. Define enemy type object with:
   - `type`: Unique identifier
   - `health`: Hit points
   - `speed`: Movement velocity
   - `pointValue`: Score reward
   - `firePattern`: Attack algorithm
2. Add spawn logic to stage wave definitions
3. Add rendering sprite/shape in enemy draw function
4. Add collision response

### Adding New Weapons
1. Add weapon object to weapon system array
2. Define fire pattern and projectile behavior
3. Add weapon name/icon to HUD
4. Add upgrade progression (level 1-5 stats)

### Modifying Difficulty
1. Edit stage definitions for enemy spawn rates
2. Adjust damage values for enemy/player projectiles
3. Scale boss health and attack frequency
4. Modify multiplier thresholds if needed

### Performance Optimization Tips
- Limit on-screen projectiles with cleanup logic
- Use object pooling for frequently created objects
- Minimize DOM access, batch canvas operations
- Profile with DevTools Performance tab

## Browser Compatibility

- Requires HTML5 Canvas 2D API support
- Requires JavaScript ES6+ (arrow functions, const/let, template literals)
- Touch event support for mobile devices
- LocalStorage for persistent high scores
- Audio context API for sound effects (graceful degradation if unavailable)

## Known Limitations

- Single-file architecture limits code organization
- All game logic runs on main thread (no Web Workers)
- Audio may be disabled by browser autoplay policies on mobile
- No network/multiplayer features
- Screen capture/recording may trigger browser permissions

## Future Enhancement Possibilities

- Leaderboard system with cloud sync
- Custom difficulty modifiers
- Additional weapon types
- Power-up system beyond weapons/bombs
- Enemy/boss variety expansion
- Achievement/medal system
- Mobile app wrapper (PWA or native)

---

**Last Updated**: 2026-10-09  
**Created by**: Claude Code Agent
