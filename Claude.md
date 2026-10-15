# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application. The game features weapon switching, smart bombs, stage progression with boss encounters, multiplier combos, and responsive controls for both desktop and mobile devices.

## Architecture

### Single-File Design
The entire codebase (`index.html`) combines:
- **HTML Structure**: Canvas container, HUD elements, overlay UI
- **CSS Styling**: Game canvas, HUD displays, animations, responsive design
- **JavaScript Logic**: Game engine, entity systems, input handling, game state management

### Technology Stack
- **HTML5 Canvas**: 2D graphics rendering
- **Vanilla JavaScript**: No external dependencies; pure ES6+ code
- **CSS3**: Animations, gradients, flexbox layouts, backdrop filters
- **Responsive Design**: Supports desktop and mobile with touch controls

## Game Systems

### Core Game Loop
The game runs on a standard game loop pattern using `requestAnimationFrame`:
- Update phase: Process input, update entities, handle collisions
- Render phase: Clear canvas and redraw all game objects
- Manages frame-dependent timing for smooth gameplay

### Entity System
Entities include:
- **Player**: The protagonist ship with health and weapon state
- **Enemies**: Regular enemies with health, movement patterns, and shooting behavior
- **Bosses**: Stage bosses with multiple attack patterns and significantly more health
- **Projectiles**: Player bullets, enemy bullets, and bomb effects
- **Effects**: Visual effects like explosions and combo popups

### Game State Management
States include:
- **Menu**: Initial game start state
- **Stage**: Active gameplay
- **Boss Battle**: Special state when facing stage bosses
- **Game Over**: End state when player loses all lives
- **Victory**: Transition between stages or game completion

## Key Features

### Weapon System
- **Standard Gun**: Default weapon with rapid fire
- **Spread Shot**: Fires multiple projectiles in a spread pattern
- **Beam Weapon**: Continuous laser with high damage
- **Missiles**: Homing projectiles that track enemies
- **Weapon Switching**: Press keys 1-4 or click weapon buttons to switch
- **Ammo Management**: Each weapon has limited ammo; replenish from power-ups

### Smart Bomb System
- **Activation**: Press spacebar or click bomb button
- **Effect**: Clears all on-screen enemies and projectiles; damages bosses
- **Limited Uses**: Players have 3 bombs per stage
- **Visual Feedback**: White flash and sound effect on activation

### Combo Multiplier
- **Accumulation**: Killing consecutive enemies without taking damage increases multiplier
- **Display**: Shows "×2", "×3", etc. above enemies as multiplier increases
- **Score Impact**: Higher multipliers greatly increase point values
- **Reset**: Taking damage reduces multiplier back to ×1

### Stage Progression
- **Multiple Stages**: Game contains sequential stages with increasing difficulty
- **Stage Bosses**: Each stage culminates in a boss battle
- **Difficulty Curve**: Enemy density and attack patterns increase with each stage
- **Stage Banners**: Display current stage number with visual fanfare

### Boss System
- **Health Tracking**: Boss health bar displayed at top of screen
- **Attack Patterns**: Bosses use varied attack patterns (spread shots, homing, etc.)
- **Phase Transitions**: Some bosses may have multiple attack phases
- **Victory Rewards**: Defeating bosses grants significant bonus points

### HUD (Heads-Up Display)
- **Lives/Hearts**: Visual representation of remaining player lives
- **Bomb Count**: Shows available bombs with bomb emoji indicators
- **Current Weapon**: Displays active weapon name with ammo count
- **Score**: Current and high score tracking
- **Boss Health**: Bar shows boss health during boss battles
- **Combo Counter**: Displays current kill combo multiplier

### Input Handling
- **Keyboard**: WASD or Arrow Keys for movement, spacebar for bomb, 1-4 for weapons
- **Mouse**: Click to aim (crosshair cursor)
- **Touch**: Drag finger to move, tap for bomb, weapon buttons visible
- **Responsive Controls**: Adapts to device input method

## Visual Design

### Color Scheme
- **Background**: Dark blue (#05122b) with gradient sky during gameplay
- **Accents**: Gold (#ffd54f) for HUD values, orange for score multipliers
- **Boss Health**: Orange-red gradient for boss health bar
- **Effects**: White flashes for hits, red vignette for critical health

### Animations
- **Heart Beat**: Critical health causes heart animation
- **Combo Pop**: Floating text shows multiplier increases
- **Stage Banner**: Sliding text announcement for new stages
- **Vignette Effect**: Red screen edges indicate low health
- **Flash Effects**: White screen flash on damage, larger flash for bombs

### Responsive Layout
- Uses `clamp()` for font sizes that scale with viewport
- Safe area insets for notched devices
- Maximum canvas size maintains aspect ratio
- Touch-action and user-select optimizations for mobile

## Game Balance

### Difficulty Progression
- Early stages have slower enemies with simple attack patterns
- Mid stages introduce more enemy types and faster movement
- Late stages feature dense enemy formations and complex boss patterns
- Progressive health increases for player and enemies

### Scoring System
- Base points for enemy kills vary by enemy type
- Combo multiplier applied to all kills in sequence
- Boss defeats award bonus points
- Encourages skilled play and risk-taking

### Health and Lives
- Player starts with a certain number of lives (displayed as hearts)
- Each hit removes one life
- Health pickups restore lives (if available)
- Game ends when all lives are lost

## Code Organization

### Game Initialization
- Canvas creation and context setup
- Resource loading (fonts, sprites if any)
- Initial game state and player setup
- Event listener registration

### Main Update Loop
- `updateGame()`: Handles all game logic updates
- `renderGame()`: Draws all game objects to canvas
- Delta time handling for frame-rate independence

### Input System
- Keyboard event listeners for movement and actions
- Mouse movement for aiming
- Touch event handling for mobile
- Button click handlers for UI controls

### Collision Detection
- AABB (Axis-Aligned Bounding Box) collision checks
- Separate collision logic for player projectiles vs enemies
- Enemy projectile vs player collision detection
- Bomb blast radius collision checking

### Enemy AI
- Simple movement patterns (straight, sine wave, etc.)
- Basic shooting behavior tied to proximity and health
- Boss-specific attack routines
- Spawn patterns based on stage difficulty

## Development Notes

### Performance Considerations
- Canvas rendering optimized for 60 FPS target
- Minimal garbage collection through object pooling
- Draw order: background → entities → effects → HUD
- requestAnimationFrame for optimal timing

### Mobile Optimization
- Touch-action: none prevents default scrolling
- Viewport-fit: cover for notched devices
- Responsive font sizes with clamp()
- Optimized touch targets (36px minimum for buttons)

### Extensibility Points
- **New Weapons**: Add weapon definitions and switching logic
- **New Enemies**: Define new enemy types with custom AI
- **New Stages**: Add stage configurations with enemy spawn patterns
- **Visual Effects**: Expand particle systems and animation effects

## Future Enhancement Opportunities

1. **Power-ups**: Add temporary boosts (rapid fire, shield, time slow)
2. **Sound Effects**: Audio for shots, explosions, and music
3. **Difficulty Modes**: Easy, Normal, Hard with different enemy patterns
4. **Leaderboards**: Local or cloud-based high score tracking
5. **Particle Effects**: Enhanced explosion and weapon effects
6. **Boss Variations**: Multiple boss types with unique attack patterns
7. **Special Abilities**: Character-specific unique attacks
8. **Persistence**: Save/load progress between sessions
9. **Co-op Mode**: Multiplayer support for multiple players
10. **Story/Narrative**: Intro/outro cutscenes and dialogue

## Running the Game

1. Open `index.html` in a modern web browser
2. Use keyboard (WASD/Arrows) to move, spacebar for bomb, 1-4 to switch weapons
3. On mobile, use on-screen buttons and swipe controls
4. Defeat all stages to win the game
5. Try to achieve the highest score possible

## Testing

- Test on various screen sizes (mobile, tablet, desktop)
- Verify collision detection accuracy
- Check weapon switching responsiveness
- Validate combo multiplier calculations
- Test boss attack patterns and difficulty scaling

## Browser Compatibility

- Chrome/Chromium (recommended)
- Firefox
- Safari (including iOS Safari)
- Edge
- Requires HTML5 Canvas and ES6+ JavaScript support
