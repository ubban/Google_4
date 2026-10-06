# Google_4 - Sky Fighter Game

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 Canvas with vanilla JavaScript. The game is a single-file implementation in `index.html` with complete game mechanics including weapon switching, smart bombs, enemy AI, and boss battles.

## Project Structure

```
Google_4/
├── index.html          # Complete game implementation (single file)
├── README.md           # Project metadata
└── Claude.md           # This file - development guide for Claude Code agents
```

## Game Features

### Player Mechanics
- **Movement**: WASD keys or arrow keys to move the player ship
- **Weapon System**: Multiple weapon types with cycling (Z/X keys)
  - Rapid fire (default weapon)
  - Spread shot
  - Laser beam
- **Smart Bombs**: Spacebar to activate - clears screen and damages bosses
- **Health System**: Player has 3 lives with shield mechanics

### Enemy Types
- **Standard Enemies**: Spawn in patterns, fire basic bullets
- **Variants**: Different movement patterns and bullet types
- **Boss Encounters**: Stage bosses with distinct attack patterns and health bars

### Game Progression
- **Stages**: Multiple levels with increasing difficulty
- **Score System**: Points awarded for enemy kills and bonus actions
- **Lives/Continues**: Player can lose lives and continue from safe point

## Technical Implementation

### Architecture
- **Single HTML File**: Entire game is embedded in `index.html` (58KB+)
- **Canvas Rendering**: Uses HTML5 Canvas for 2D graphics
- **Game Loop**: RequestAnimationFrame-based update/render cycle
- **Touch Support**: Mobile-friendly with touch controls and safe-area insets

### Key Systems

#### Game State
- Player position, health, weapons, bomb count
- Enemy array with position, health, bullet patterns
- Particle effects for visual feedback
- Game stages with wave/boss definitions

#### Rendering
- Canvas context stored globally for performance
- Layered rendering: background, gameplay objects, UI/HUD
- Dynamic scaling for responsive canvas sizing
- Text rendering for score, lives, weapon indicators

#### Input Handling
- Keyboard events (WASD, arrows, Z, X, spacebar)
- Touch/pointer events with coordinate mapping
- Mobile viewport optimization with safe-area-inset

### Performance Considerations
- Object pooling for bullets and particles (if implemented)
- Canvas rendering is highly optimized for arcade-style gameplay
- Collision detection uses bounding box calculations
- Enemy AI uses simple state machines

## Code Organization

The `index.html` file contains:

1. **HTML Structure** (head, body with canvas)
2. **CSS Styling** (responsive canvas, HUD layout, animations)
3. **JavaScript Game Logic** (in single `<script>` tag):
   - Game initialization and configuration
   - Player class and weapon system
   - Enemy class and spawning logic
   - Particle/effect system
   - Game state management
   - Input/event handling
   - Game loop and rendering
   - Collision detection
   - UI/HUD updates

## Working with Claude Code Agents

### When Adding Features

1. **Understand the Game Loop**: All game logic runs in the main update function called each frame
2. **Maintain Single-File Structure**: Keep all code in `index.html` - avoid splitting into multiple files
3. **Follow Existing Patterns**: 
   - Use the same class structure for new entities (player, enemies)
   - Follow the established naming conventions
   - Use Canvas methods consistently for rendering

### Making Changes

#### For Gameplay Changes
- Modify enemy spawn patterns in the stage/wave configuration
- Adjust weapon stats in the weapon definition objects
- Update collision detection logic in collision check functions
- Modify player or boss behavior in their respective classes

#### For Visual Changes
- CSS rules control layout and styling
- Canvas rendering code draws all game objects
- Particle effects add visual polish - enhance these for better feedback

#### For New Mechanics
- Keep state in the main game object
- Add to the game loop update phase
- Integrate with input handling system
- Add corresponding rendering code

### Testing Changes

1. Open `index.html` in a web browser
2. Test basic gameplay mechanics:
   - Player movement and collision
   - Weapon switching and firing
   - Smart bomb activation
   - Enemy spawning and patterns
3. Check mobile responsiveness
4. Verify performance (no stuttering, smooth 60fps)

### Git Workflow

- Commit messages should describe gameplay/feature changes clearly
- Keep commits focused on single features or fixes
- Test in browser before committing

## Code Style Guidelines

- **Naming**: camelCase for variables/functions, PascalCase for classes
- **Comments**: Add comments for complex game logic and state machines
- **Performance**: Avoid creating new objects every frame when possible
- **Responsive Design**: Use CSS viewport units (vw, vh, clamp()) for sizing
- **Touch Support**: Consider mobile controls in input handling

## Browser Compatibility

- Modern browsers with HTML5 Canvas support
- Mobile browsers (iOS Safari, Chrome Mobile) with touch support
- Safe-area-inset support for notched devices

## Development Tips

1. **Debugging Gameplay**: Use console.log() in the game loop (carefully - impacts performance)
2. **Visual Debugging**: Add debug mode that renders collision boxes
3. **Pause Feature**: Consider adding a pause toggle for testing
4. **Performance**: Profile with browser DevTools if adding complex features
5. **Mobile Testing**: Use browser developer tools device emulation or test on actual devices

## Known Patterns

- **Game State Initialization**: Sets up canvas, loads fonts, initializes objects in init()
- **Enemy Patterns**: Waves are defined as configurations; spawning follows time-based and count-based triggers
- **Weapon Cycling**: Player cycles through weapon array with weapon type indicators
- **Collision Detection**: Uses rectangular bounding boxes for simplicity and performance
- **Screen Wrapping**: Some objects wrap around screen edges or despawn off-screen

## Future Enhancement Ideas

- Add sound effects and music
- Implement difficulty modes
- Add high score persistence (localStorage)
- Create custom level editor
- Add more enemy types and boss variations
- Implement combo/chain scoring system
- Add achievements or unlockables
