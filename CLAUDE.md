# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built with vanilla HTML5, CSS3, and JavaScript. It's a browser-based game featuring:
- Classic vertical scrolling shoot-em-up gameplay
- Multiple weapon types with switching mechanics
- Smart bomb special attacks
- Stage-based bosses
- Lives/Health system
- Score multiplier progression
- Responsive design for desktop and mobile

## Project Structure

```
Google_4/
├── index.html          # Main game file (all HTML, CSS, and JavaScript)
├── README.md           # Basic project info
├── CLAUDE.md           # This file - codebase documentation
└── .git/               # Git repository
```

## Key Files

### index.html (Main Game File)
The entire game is contained in a single `index.html` file (~1500+ lines). Structure:

#### HTML Structure
- `<head>`: Meta tags, viewport configuration, favicon, title
- `<body>`: Game container with canvas and UI elements
  - `#container`: Main game wrapper
  - `#gameCanvas`: Canvas element for rendering (320x480 viewport)
  - `#hud`: Heads-up display showing stats (hearts, bombs, weapon, multiplier)
  - `#vignette`: Red vignette overlay (damage/alert indicator)
  - `#flash`: White flash overlay (hit/bomb effects)

#### CSS Sections
1. **Root & Global Styles**: Dark theme configuration, box-sizing reset
2. **Canvas & Container**: Fixed positioning, centered layout
3. **HUD Display**: Stat badges with backdrop blur, responsive font sizing
4. **Hearts/Lives Display**: Visual health indicator
5. **Bombs Display**: Smart bomb counter
6. **Animations**: Heart beat pulse, vignette pulse, flash effects
7. **Mobile/Responsive**: Safe area insets, touch handling

#### JavaScript Sections
The JavaScript implements the complete game engine:

1. **Canvas Setup**: 320x480 internal resolution, scales to fit window
2. **Asset Generation**: Procedural sprite generation for:
   - Player ship (blue triangle)
   - Enemy ships (red/orange variations)
   - Bullets and effects
   - Particles and explosions
3. **Game State**: 
   - Player position, health, weapon type, bombs
   - Enemies list with health and position
   - Bullets (both player and enemy)
   - Particles for visual effects
   - Score and multiplier
4. **Input Handling**:
   - Mouse movement for aiming
   - Keyboard controls (W/A/S/D for movement, SPACE for shoot, Z/X for weapon switch, E for bomb)
   - Touch controls for mobile (aim at touch point)
5. **Game Logic**:
   - Enemy spawning patterns
   - Collision detection (bullets vs enemies/player, player vs enemies)
   - Weapon system (multiple weapon types with different fire patterns)
   - Smart bomb mechanics (damages all on-screen enemies)
   - Score and multiplier calculation
6. **Rendering**:
   - Background gradient sky
   - Player ship rendering with rotation
   - Enemy rendering with health bars
   - Bullet rendering
   - Particle effects and explosions
   - UI rendering (HUD, health, bombs, weapon indicator)

## Game Mechanics

### Player Controls
- **Movement**: W/A/S/D keys or mouse cursor (cursor-based aiming)
- **Shoot**: SPACE bar (continuous fire)
- **Weapon Switch**: Z/X keys (cycle through weapon types)
- **Smart Bomb**: E key (damages all enemies on screen, limited use)
- **Mobile**: Touch to aim, UI buttons for actions

### Weapons
Multiple weapon types available:
- Standard bullets (single stream)
- Spread shot (multiple angles)
- Rapid fire (higher rate, lower damage)
- Other variations (determined by weapon switching)

### Health & Lives
- Player starts with multiple lives (shown as heart indicators)
- Taking damage reduces health
- Game Over when all lives lost
- Can be restored through pickups or power-ups

### Smart Bombs
- Limited quantity per stage/game
- Displayed as bomb counters in HUD
- Activates area damage to all visible enemies
- Special visual feedback (white flash overlay)

### Score System
- Points awarded for defeating enemies
- Multiplier increases with consecutive hits/destroys
- Multiplier displayed in HUD
- Affects final score calculation

## Development Notes

### Canvas Resolution
- Internal game resolution: 320x480 pixels
- Scales to fit window while maintaining aspect ratio
- Uses CSS `max-width: 100%; max-height: 100%;` for responsive scaling
- All coordinates and calculations use internal resolution

### Color Scheme
- Background: Dark blue gradient (`#0d2f5e` to `#1e6fd8` to `#7fc1ff`)
- Player: Blue (`#1976d2`)
- Enemies: Red/Orange variations
- Accent: Gold/Yellow (`#ffd54f`)
- Text: White with shadow

### Performance Considerations
- All sprites are procedurally generated (no external assets)
- Canvas rendering uses 2D context
- Efficient particle system for effects
- requestAnimationFrame for smooth animation
- Collision detection is basic bounding box (not pixel-perfect)

### Mobile Support
- Responsive viewport meta tags
- Touch action none on canvas
- Safe area insets for notched devices
- No horizontal scroll
- Touch-based aiming and controls

## How to Run

1. **Open in Browser**: Simply open `index.html` in any modern web browser
2. **Local Development**: Can be served via any HTTP server (for better performance)
3. **No Build Process**: Single file, no build tools required
4. **No Dependencies**: Pure vanilla JavaScript, no libraries

## Game Flow

1. **Initialization**: Game loads with player at bottom center
2. **Stage Start**: Enemies spawn in waves
3. **Combat**: Player shoots and maneuvers to avoid enemy fire
4. **Boss Encounters**: Special enemy waves with boss-level challenges
5. **Victory/Defeat**: Stage completes or game over
6. **Progression**: Score and multiplier persist through stages

## Code Organization

The game uses a state-based architecture with:
- Single canvas and 2D context
- Main game state object storing all active entities
- Animation loop (requestAnimationFrame) that:
  1. Clears canvas
  2. Updates game logic (player input, enemy AI, collision detection)
  3. Renders all sprites and effects
  4. Updates HUD

## Future Enhancement Ideas

- Additional weapon types and upgrades
- Progressive difficulty scaling
- Power-up system (shields, health restore, etc.)
- Particle effects improvements
- Sound effects and background music
- Leaderboard/high scores
- Multiple difficulty modes
- More elaborate boss patterns
- Procedural enemy generation

## Git Branches

- **claude/inspiring-planck-tealag**: Current development branch
- **main/master**: Release branch

## Attribution

Game implementation and maintenance by Claude Code agent.
