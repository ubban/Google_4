# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas. Players control a fighter aircraft, destroy enemies, collect power-ups, and face boss battles across multiple stages.

### Key Features
- **Vertical scrolling shooter** with progressive difficulty
- **Multiple weapon systems** (Vulcan, Missile, Laser) with upgradeable levels
- **Smart bombs** for powerful screen-clearing attacks
- **Boss battles** with health bars and special mechanics
- **Combo/score multiplier system** for consecutive hits
- **Mobile-friendly** with touch controls and responsive design
- **Accessibility features** including reduced motion support
- **Audio controls** with mute functionality
- **High score tracking** using browser localStorage

## Technology Stack

- **Language**: Vanilla JavaScript (ES6+)
- **Rendering**: HTML5 Canvas 2D context
- **Markup**: Semantic HTML5
- **Styling**: CSS3 with custom properties, animations, and gradients
- **Storage**: Browser localStorage for persistence
- **Audio**: Web Audio API (if implemented)
- **Responsive Design**: CSS `clamp()` and viewport units for mobile compatibility

## Project Structure

```
Google_4/
├── index.html          # Single-file game (1669 lines)
│   ├── HTML markup
│   ├── Embedded CSS styles
│   └── Embedded JavaScript game engine
└── README.md           # Basic project information
```

## Architecture Overview

### Game Loop & Rendering
The game runs on a canvas-based render loop using `requestAnimationFrame`. The main rendering pipeline:
1. Clear canvas with sky gradient background
2. Update game state (player, enemies, projectiles)
3. Detect collisions
4. Render all game objects
5. Update HUD displays

### Core Game Objects & Classes

The codebase contains multiple class definitions for different game entities:

- **Player** - Controllable fighter aircraft with movement, firing, and bomb mechanics
- **Enemies** - Various enemy types with different behaviors and drop patterns
- **Projectiles** - Bullets and missiles with collision detection
- **Explosions & Particles** - Visual feedback system
- **Bosses** - Special enemy types with health bars and unique attack patterns
- **Power-ups** - Weapon upgrades, health restoration, bombs

### Game State Management

Key state variables track:
- Player position, health, bomb count
- Active weapons and upgrade levels
- Current score and score multiplier
- Enemy and projectile collections
- Game phase (playing, paused, game over)
- Stage/level progression

### Input Handling

**Keyboard Controls:**
- Arrow keys or WASD for movement
- Space for firing
- B for bomb deployment
- P for pause
- M for mute
- X for motion toggle

**Mouse/Touch Controls:**
- Mouse position aims continuously
- Click/touch fires
- Touch zones for mobile (left: movement, right: fire/bomb)
- Joystick simulation on touch devices

## Key Game Mechanics

### Weapon System
- **Vulcan** (Rapid fire, close range)
- **Missile** (Guided, medium range, higher damage)
- **Laser** (Piercing beam, long range)
- Weapons upgrade through collected power-ups
- Multiple levels increase fire rate, damage, and spread

### Scoring & Multipliers
- Base points vary by enemy type
- Combo multiplier increases with consecutive hits (x1 to x10+)
- Boss kills award bonus points
- Power-up collection multiplies incoming reward values

### Health & Damage System
- Player starts with 3 hearts
- Enemy projectiles cause one-hit damage
- Power-ups restore health
- Game over on losing all health
- Vignette effect (red pulse) indicates critical health (<2 hearts)

### Boss Mechanics
- Health bar displayed at top of screen
- Unique bullet patterns
- Boss-specific AI behavior
- Defeat triggers stage progression

### Difficulty & Progression
- Multiple stages with increasing enemy density
- Boss encounters between stages
- Progressive weapon availability
- Enemy bullet patterns become more complex

## Development Guidelines

### Code Style & Conventions

1. **Variable Naming**: camelCase for variables/functions, PascalCase for classes
2. **Code Organization**: Group related functionality within class blocks
3. **Comments**: Add comments for non-obvious game logic or mathematical calculations
4. **Constants**: Define game parameters (speeds, sizes, colors) at the top of relevant sections

### Adding New Features

#### Adding a New Enemy Type
1. Define enemy class with properties: position, size, health, movement behavior
2. Implement `update()` method for position changes
3. Implement `draw()` method for canvas rendering
4. Add collision detection logic in the main update loop
5. Define drop table for power-ups on defeat

#### Adding a New Weapon
1. Create weapon configuration object (fire rate, damage, projectile speed)
2. Implement firing logic in player update
3. Add visual feedback (muzzle flash, sound if available)
4. Add weapon selection UI to HUD
5. Update power-up system to include new weapon drops

#### New Stage/Boss
1. Define stage wave patterns (enemy sequences, timing)
2. Create boss class with health, attack patterns, phases
3. Add stage banner announcement
4. Configure difficulty progression
5. Update stage progression logic

### Responsive Design Considerations

- Use CSS `clamp()` for fluid sizing across devices
- Test on multiple viewports (mobile, tablet, desktop)
- Ensure touch zones are at least 44x44px (accessibility standard)
- Respect `prefers-reduced-motion` media query for animations
- Support safe area insets for notched devices (`env(safe-area-inset-*)`)

### Performance Optimization

- Canvas operations are grouped to minimize repaints
- Object pooling should be considered for high-frequency projectiles
- Collision detection uses spatial partitioning where possible
- Animation frames are throttled to 60fps via requestAnimationFrame
- Image assets (if added) should be preloaded and cached

### Accessibility Features

- **Screen Reader Support**: Live region (`#srLive`) for important game events
- **Reduced Motion**: `prefers-reduced-motion` media query disables animations
- **High Contrast**: White text on dark background, good color contrast
- **Keyboard Navigation**: All controls accessible via keyboard
- **ARIA Labels**: Buttons have titles and semantic markup

## Testing & Debugging

### Manual Testing Checklist
- [ ] Game initializes correctly
- [ ] Player movement and firing work
- [ ] Collision detection functions properly
- [ ] Scoring and multipliers calculate correctly
- [ ] Boss battles trigger and complete
- [ ] High score persistence across sessions
- [ ] Touch controls work on mobile
- [ ] Pause/resume functionality
- [ ] Mute toggle works
- [ ] Reduced motion respects user preference
- [ ] Responsive design at various screen sizes

### Browser Console Debugging

The game engine exposes certain objects globally for debugging:
- Check console for error messages
- Performance can be profiled using Chrome DevTools Performance tab
- Memory usage should be monitored during long play sessions

## Game Balance Parameters

Key tuning values to adjust difficulty/feel:
- Enemy spawn rates and patterns (waves configuration)
- Player fire rate and damage per weapon
- Enemy projectile speed and density
- Boss health pools and attack frequency
- Power-up drop rates
- Score multiplier caps
- Player speed and acceleration

These are typically defined as constants in their respective class definitions.

## Future Enhancement Ideas

- [ ] Sound effects and background music
- [ ] Particle effects system (currently basic)
- [ ] More weapon types and upgrade paths
- [ ] Leaderboard integration
- [ ] Multiple difficulty modes
- [ ] Story/narrative elements
- [ ] More boss varieties
- [ ] Enemy variety and unique patterns
- [ ] Special events and power-up combinations
- [ ] Persistent progression/campaigns

## File Locations & Code Organization

All game code is embedded in `index.html`. The file structure is:
1. **Lines 1-358**: CSS styles and animations
2. **Lines 361-399**: HTML markup and DOM structure
3. **Lines 400+**: JavaScript game engine (classes and game loop)

## Common Tasks

### To modify weapon properties:
Look for weapon configuration objects in the player class section.

### To change game colors:
Update CSS custom properties in `:root` selector (~line 11) or inline styles.

### To adjust difficulty:
Modify wave patterns, enemy spawn rates, and boss configurations.

### To add UI elements:
Update HTML in container div and corresponding CSS styles.

### To change keyboard/input:
Find event listeners in input handling section of JavaScript.

## Performance Notes

- Game runs at 60fps target via requestAnimationFrame
- Canvas size adapts to viewport for optimal performance
- Large number of simultaneous objects (enemies, projectiles) may impact performance on low-end devices
- Consider implementing object pooling for high-frequency spawned objects if experiencing lag

## Version & Attribution

- **Project**: Sky Fighter
- **Type**: HTML5 Vertical Shooter
- **Architecture**: Single-file vanilla JavaScript
- **Last Updated**: 2026-10-06
