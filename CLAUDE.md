# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 application. The game features:

- Dynamic difficulty levels (Easy, Normal, Hard)
- Multiple weapon systems that can be upgraded
- Boss battles with unique attack patterns
- Combo/multiplier system for consecutive hits
- Touch and keyboard controls
- Sound effects and visual feedback systems
- Score tracking with personal bests
- Stage/level progression

## Architecture

The entire game is implemented as a single `index.html` file (~1669 lines) with embedded CSS and JavaScript. The application uses a **game loop pattern** with canvas-based 2D rendering.

### Core Structure

```
HTML5 Canvas-based Game Engine
├── UI Layer (HUD, Menus, Overlays)
├── Game State Management
├── Entity Systems
│   ├── Player
│   ├── Enemies
│   ├── Bosses
│   ├── Projectiles
│   ├── Particles
│   └── Visual Effects
├── Input Handling
│   ├── Keyboard
│   ├── Mouse/Touch
│   └── Virtual Joystick
└── Audio System
```

## Key Game Systems

### Player System
- Position-based movement controlled by mouse cursor or touch
- Health system (hearts as visual indicator)
- Weapon system with switchable weapons and levels
- Bomb/smart bomb system (limited quantity)
- Collision detection with enemies and projectiles

### Weapon System
Multiple weapon types:
- **VULCAN** - Rapid-fire projectiles (base weapon)
- **SPREAD** - Fan-like spread pattern
- **LASER** - Concentrated beam damage
- **MISSILE** - Homing projectiles

Each weapon has upgradeable levels that increase damage/fire rate.

### Enemy System
- Wave-based spawning patterns
- Multiple enemy types with different behaviors
- Loot drops (health, bombs, weapon upgrades)
- Collision and damage calculations

### Boss System
- Boss encounters trigger at the end of stages
- Boss data structure includes:
  - Name
  - Health/damage values
  - Attack patterns
  - Theme configuration
  - Victory conditions

### Scoring System
- Base points for enemy defeats
- Multiplier system (combo tracking)
- Consecutive hit bonuses
- Stage completion bonuses
- Personal best score tracking (via localStorage)

### Visual/Audio Systems
- Screen shake and flash effects
- Particle systems for explosions and impacts
- Sound effect library (SFX)
- Theme switching between stages
- Accessibility-friendly visual modes

## File Organization

Since this is a single-file application:

- **index.html** - Complete game implementation
  - HTML structure (canvas, HUD, menus)
  - CSS styling (dark theme optimized for game)
  - JavaScript game engine

## Development Notes

### Key Constants & Configuration

```javascript
DIFFICULTIES = { easy, normal, hard }  // Difficulty settings
THEMES = [...]                         // Stage visual themes
BOSSES = [...]                         // Boss definitions
```

### Input Handling
- **Keyboard**: Arrow keys or WASD for movement
- **Mouse**: Position determines player position
- **Touch**: Virtual joystick on mobile devices
- **Buttons**: Fire, bomb, pause, mute controls

### Game Loop
- Frame timing tracking via `frameTimes` array
- 60fps target
- Update -> Draw cycle per frame
- Collision detection and response

### State Management
- Game state: "menu" → "playing" → "gameOver" → "menu"
- Pause state handling
- Difficulty selection persistence
- Score persistence (localStorage)

## Running the Game

Simply open `index.html` in a modern web browser:
```bash
# Option 1: Direct file opening
open index.html

# Option 2: Local server
python3 -m http.server 8000
# Then visit http://localhost:8000/
```

### Browser Support
- Modern browsers with HTML5 Canvas support
- Touch-enabled browsers for mobile gameplay
- Web Audio API for sound effects

## Notable Features

### Accessibility
- Screen reader support (aria-live regions)
- Reduced motion option (respects prefers-reduced-motion)
- Keyboard navigation support
- Visual contrast optimized for gameplay

### Performance
- Canvas-based rendering for efficiency
- Throttled UI updates
- Sound effect pooling
- Frame time monitoring

### Mobile Optimization
- Responsive canvas sizing
- Touch controls with virtual joystick
- Safe area insets for notched devices
- Proper viewport configuration

## Common Development Tasks

### Adding a New Enemy Type
1. Define enemy properties (speed, health, sprite behavior)
2. Add spawn logic in stage configuration
3. Add collision/damage behavior
4. Add loot drop handling

### Adding a New Weapon
1. Define weapon properties in constants
2. Add firing logic in update loop
3. Define projectile behavior
4. Add visual/audio feedback

### Adjusting Difficulty
1. Modify DIFFICULTIES object
2. Adjust enemy spawn rates
3. Adjust boss health/damage
4. Adjust score multipliers

### Tuning Game Balance
- Enemy health and damage values
- Weapon damage and fire rates
- Boss patterns and health
- Loot drop frequencies
- Score multipliers per difficulty

## Known Limitations & Future Improvements

- Single-file architecture limits modularity
- No persistent high-score cloud sync
- Limited to single-player mode
- Mobile performance depends on device hardware

## Testing

The game is primarily tested through:
- Manual gameplay testing across difficulties
- Browser compatibility verification
- Mobile/touch control testing
- Audio playback validation

To test new features:
1. Load the game in browser
2. Test on intended difficulty
3. Verify visual/audio effects
4. Check touch controls on mobile
5. Validate score calculations

---

**Last Updated**: 2026-10-16
**Repository**: ubban/google_4
**Active Branch**: claude/inspiring-planck-yo49fr
