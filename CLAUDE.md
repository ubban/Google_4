# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built as a single-file HTML/CSS/JavaScript application. The game features dynamic difficulty, multiple weapon types, boss encounters, and a scoring system with combo multipliers.

- **Technology**: HTML5 Canvas, CSS3, Vanilla JavaScript
- **Architecture**: Monolithic single-file (`index.html`)
- **Target**: Modern web browsers with canvas support

## File Structure

```
Google_4/
├── index.html       # Complete game implementation (1669 lines)
├── README.md        # Project description
└── CLAUDE.md        # This documentation file
```

## Architecture Overview

The game is implemented entirely within `index.html` with three main sections:

### 1. HTML Structure (lines 1-200)
- Game canvas container
- HUD elements (heads-up display)
- Boss health bar
- Control buttons (pause, settings)
- Overlay for menus and game-over screens
- Touch controls (for mobile)

### 2. CSS Styling (lines 200-500)
- Responsive design with `clamp()` for fluid typography
- Dark theme with blue/cyan color scheme
- Animations for UI feedback (combo pop, stage banner, heart beat)
- Safe-area-inset support for notched devices
- Canvas and HUD positioning
- Button and card styling with glass-morphism effects

### 3. JavaScript Game Logic (lines 500-1667)

#### Core Game Objects

**Player Object**
- Position: `x`, `y`
- Dimensions: `w` (width), `h` (height)
- State: `hp`, `weapon`, `invuln` (invulnerability timer)
- Properties: velocity, acceleration

**Enemy Objects**
- Types: `plane`, `tank`, `missile-launcher`
- Properties: position, dimensions, HP, color, rotation
- Behavior: spawning waves, collision detection

**Boss Object**
- Special large enemy with distinct visual and behavior
- Health bar display
- Pattern-based attack sequences

**Bullet Objects**
- Player bullets: `laser` or standard
- Enemy bullets: projectiles with physics
- Collision detection with enemies and player

**Power-up Objects**
- Types: `life` (health), `bomb` (smart bomb), `weapon` (weapon upgrade)
- Rotation animation when rendering

**Particle System**
- Explosion effects
- Color-based visual feedback
- Fade-out animation with alpha blending

#### Key Functions

**Game State Management**
- `resetGame()` - Initialize/restart the game
- `update(dt)` - Main update loop (called per frame)
- `draw()` - Render all game objects
- `loop(now)` - RequestAnimationFrame handler

**Player Control**
- Keyboard input: Arrow keys for movement, Space for shoot, Z for bomb
- Mouse input: Click to target, move to aim
- Touch input: Virtual joystick on mobile

**Collision Detection**
- Enemy-bullet collision
- Enemy-player collision
- Player-bullet collision (enemy bullets)
- Power-up collection

**Weapon System**
- **Standard**: Rapid-fire bullets
- **Laser**: Wider damage area with cyan glow

**Boss System**
- Boss triggers at end of stage
- Pattern-based attacks with visual tell
- Health bar display

**Audio System**
- Sound effects for: shooting, hits, enemy defeat, power-ups, UI
- Uses Web Audio API or fallback

**Difficulty System**
- Easy, Normal, Hard modes
- Affects enemy spawn rates and patterns
- Player can select difficulty at game start

**UI/Overlay System**
- Main menu with difficulty selection
- Game-over screen with stats and high-score display
- Pause menu
- HUD showing: health (hearts), bombs, weapon type, combo multiplier

#### Visual Features

**Theming System**
- Multiple color themes for different stages
- Sky gradient backgrounds
- Cloud parallax scrolling at different speeds
- Sun glow effect
- Screen shake on player hit/boss defeat

**Particle Effects**
- Explosion particles on enemy defeat
- Color-coded based on enemy type

**Visual Feedback**
- Screen flash on damage
- Vignette effect when health is critical
- Combo pop animation (yellow text)
- Stage banner animation at level start

#### Performance Optimization

- `quality` setting (0 = low, affects glow effects)
- Shadow effects optional based on quality level
- Efficient circle drawing using canvas `arc()`
- Delta-time based physics (frame-rate independent)

## Data Structures

### Constants

**Enemy Types** (ENEMY_TYPES object)
- Definition of stats for each enemy type
- HP, spawn rate, score value, appearance

**Themes** (THEMES array)
- Color palettes for each stage
- Sky gradients, sun color, cloud color

**Difficulty Settings**
- Easy, Normal, Hard configurations
- Spawn rate multipliers, enemy count variations

## Game Loop Flow

1. **Input Handling** - Detect keyboard, mouse, touch input
2. **Player Update** - Move player based on input
3. **Enemy Logic** - Spawn waves, move enemies, attack
4. **Boss Logic** - Boss patterns and attacks
5. **Bullet Updates** - Move bullets, check collisions
6. **Collision Detection** - All collision checks
7. **Power-up Logic** - Spawn and collect power-ups
8. **Particle Updates** - Update and animate particles
9. **Rendering** - Draw entire scene to canvas
10. **UI Update** - Update HUD with current state

## Key Features

### Gameplay Mechanics
- **Multiple Stages** - Each stage has a boss
- **Weapon Switching** - Pick up weapons to change attack type
- **Smart Bombs** - Limited use bombs clear screen
- **Health System** - Start with 3 lives
- **Combo Multiplier** - Kill enemies in sequence for score multiplier
- **Boss Encounters** - Pattern-based boss fights at stage end

### User Experience
- **Responsive Design** - Works on desktop, tablet, mobile
- **Accessibility** - Safe-area-inset for notched devices
- **Touch Controls** - Virtual joystick for mobile
- **Difficulty Selection** - Choose challenge level
- **High Score Tracking** - LocalStorage persistence
- **Pause Functionality** - Pause/resume game

### Visual Polish
- Smooth animations and transitions
- Particle effects and screen shake
- HUD with glass-morphism style
- Color-coded UI elements
- Responsive font sizing

## Development Notes

### Technical Debt / Optimization Opportunities
1. Consider splitting into multiple files for maintainability
2. Implement object pooling for bullets and particles
3. Add level editor or data-driven stage definitions
4. Separate concerns (game logic, rendering, input)

### Browser Compatibility
- Requires HTML5 Canvas support
- Uses modern CSS features (clamp, grid, backdrop-filter)
- Touch event support for mobile
- Web Audio API for sound (with fallback)

### Performance Considerations
- 60 FPS target (delta-time capped at 50ms)
- Canvas shadow effects optional (quality setting)
- Efficient particle lifecycle management
- No external libraries or dependencies

## Controls

**Keyboard**
- Arrow Keys: Move
- Space: Shoot
- Z: Drop bomb
- P: Pause

**Mouse**
- Move: Aim
- Click: Shoot

**Touch**
- Virtual joystick (left side of screen): Movement
- Touch screen (right side): Auto-shoot

## Customization

### Color Customization
Colors defined in:
- CSS variables (root)
- Theme color objects in JavaScript
- Individual UI element styles

### Game Balance
Difficulty parameters in:
- `DIFFICULTY_CONFIG` object
- `ENEMY_TYPES` definitions
- Boss attack patterns

### Audio
- Sound effects defined in `sfx` object
- Can be toggled via quality setting
- Uses Web Audio API

## Future Enhancements

1. **Level Progression** - More stages and bosses
2. **Power-up Variety** - Additional power-up types
3. **Leaderboard** - Global/local high scores
4. **Sound Options** - Volume control, music toggle
5. **Mobile Optimization** - Better touch controls
6. **Accessibility** - Colorblind modes, audio descriptions
7. **Modding System** - Data-driven content
