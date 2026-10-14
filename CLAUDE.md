# Sky Fighter - Codebase Documentation

## Project Overview
Sky Fighter is a **Raiden-style vertical shoot-'em-up game** built with vanilla JavaScript and HTML5 Canvas. It's a single-file web application that runs entirely in the browser with no external dependencies.

**Repository:** `ubban/Google_4`  
**Main File:** `index.html` (1669 lines)  
**Technology Stack:** HTML5, CSS3, Vanilla JavaScript (Canvas API)

## Game Features

### Core Gameplay
- **Vertical scrolling shooter** - Navigate a starfighter through wave-based combat
- **Dual weapon system** - Switch between Laser and Missile armaments
- **Smart bombs** - Deploy heavy ordnance to clear the screen
- **Wave-based progression** - Each stage introduces progressively harder enemies
- **Boss encounters** - End-of-stage boss battles with distinct patterns
- **Score system** - Track player performance with persistent high score

### Controls
- **Mouse/Touch Movement** - Move the player craft to avoid projectiles and position for attack
- **Click/Tap to Shoot** - Hold to maintain fire on current weapon
- **W Key** - Switch to Laser weapon
- **M Key** - Switch to Missile weapon  
- **Space Bar** - Deploy smart bomb (limited charges per stage)
- **R Key** - Restart game after game over

### Visual Design
- **Deep space aesthetic** - Dark blue gradient background (#05122b to #0d2f5e)
- **Responsive layout** - Adapts to any screen size with CSS clamp() for scaling
- **Safe area support** - Respects device notches and safe zones (env(safe-area-inset-*))
- **Crosshair cursor** - Visual feedback indicating shoot readiness
- **HUD displays** - Real-time stats (health, weapons, score, stage) with text shadow effects

## Architecture

### Code Organization
The entire application is contained in `index.html` as a self-contained module. The structure follows this pattern:

```
HTML Document
├── Head Section
│   ├── Meta tags (viewport, description, theme)
│   ├── Favicon (SVG embed)
│   └── CSS Styling (all inline)
├── Body
│   ├── Game Container (#container)
│   ├── Canvas Element (#gameCanvas)
│   ├── HUD Panel (#hud)
│   └── Game Over Screen (#gameOverScreen)
└── Script Section
    └── JavaScript Game Engine (inline)
```

### Game Engine Components

#### 1. **Game State Management**
- `gameState`: Main game object tracking current stage, score, lives, weapons
- `player`: Spaceship state (position, health, weapon config)
- `enemies`: Array of active enemy entities
- `particles`: Visual effects (explosions, impacts)
- `waves`: Pre-defined enemy formations and timings

#### 2. **Rendering System**
- Canvas-based 2D rendering using `ctx.fillRect()`, `ctx.drawImage()`, custom sprite drawing
- Layered drawing (background, players, enemies, projectiles, effects)
- Frame-rate independent animation using `requestAnimationFrame`

#### 3. **Entity System**
- **Player spacecraft** - Tracks position, health, firepower, bomb supply
- **Enemy ships** - Multiple types with unique behaviors (basic, heavy, sniper, boss)
- **Projectiles** - Player bullets and enemy fire
- **Particles** - Explosion effects and visual feedback

#### 4. **Collision Detection**
- Distance-based collision for bullets and enemies
- Rectangle-based collision for player damage
- Boundary checking for viewport constraints

#### 5. **Input Handling**
- Mouse move tracking for cursor-based player movement
- Touch support with normalized coordinates
- Keyboard event listeners (WASD, Space for abilities, R for restart)

#### 6. **Wave System**
- Pre-scripted enemy waves with timing and patterns
- Automatic difficulty scaling through stage progression
- Boss encounters with distinct movement and attack patterns

## Key Implementation Details

### Responsive Design Approach
- Uses CSS `clamp()` for adaptive font sizing: `clamp(12px, 2.8vw, 16px)`
- Viewport-relative measurements: `50vw`, `96vw`, `clamp(6px, 2vw, 12px)`
- Touch-optimized with `touch-action: none` and tap highlight removal
- Safe area compensation for mobile notches

### Weapon System
- **Laser**: Rapid-fire continuous beam (high rate, low damage per hit)
- **Missile**: Slower, larger projectiles (medium rate, higher damage per hit)
- Weapon switching with dedicated keys (W/M)
- Smart bomb: Screen-clear ability with limited charges

### Mobile Optimization
- Maximum viewport scaling disabled (`maximum-scale=1`)
- User scaling disabled (`user-scalable=no`)
- Viewport fit for notched devices (`viewport-fit=cover`)
- Touch action control to prevent browser touch behaviors

### Performance Considerations
- Single canvas for all rendering
- Efficient sprite drawing with minimal fillRect calls
- Particle pooling (explosion effects cleaned up after display)
- RequestAnimationFrame for 60fps target

## Styling Highlights

### Color Scheme
- **Primary Background**: `#05122b` (very dark blue)
- **Sky Gradient**: `linear-gradient(#0d2f5e, #1e6fd8 45%, #7fc1ff)` (dark to light blue)
- **Text**: White (`#fff`) with text-shadow for readability
- **Accents**: Blues and teals matching the space theme

### CSS Features Used
- CSS variables (`:root` color scheme)
- Viewport units (vw, vh, max/min combinations)
- Safe area insets (`env(safe-area-inset-*)`)
- CSS calc and clamp functions
- Flex layout for HUD arrangement
- Box-shadow for depth effects
- Linear gradients for visual polish

## Development Guidelines

### When Modifying Game Logic
1. **Keep responsive calculations in CSS** - Use clamp() and viewport units rather than hard-coded pixels
2. **Test on multiple screen sizes** - The game scales from mobile to desktop
3. **Maintain safe area awareness** - Preserve `env(safe-area-inset-*)` usage
4. **Update the README.md timestamp** if making significant feature changes
5. **Commit messages should reference the game feature** being added/fixed

### Adding New Features
- **New enemy types**: Add to wave definitions and implement movement/attack patterns
- **New weapons**: Extend weapon system with new firing mechanisms
- **New UI elements**: Add to #hud section and style with responsive units
- **Game balance**: Adjust enemy health, player damage, spawn rates in game state

### Testing Considerations
- Test mouse movement and click accuracy
- Verify touch controls on mobile devices
- Check responsive scaling (use browser devtools)
- Confirm safe area rendering on notched devices
- Validate collision detection edge cases

## Deployment

The project is deployment-ready as a static single-file application:
- **No build step required** - Open `index.html` directly in a browser
- **No dependencies** - Pure HTML5/CSS3/JavaScript
- **No server required** - Works entirely client-side
- **Cross-browser compatible** - Uses standard Canvas API

### To Deploy
1. Copy `index.html` to web server
2. Serve with standard MIME type (`text/html`)
3. Optionally configure cache headers for the canvas assets

## File Structure
```
/home/user/Google_4/
├── CLAUDE.md              (This file - codebase documentation)
├── README.md              (Project timestamp)
├── index.html             (Complete game - 1669 lines)
└── .git/                  (Version control)
```

## Recent Changes
- **Latest commit**: "Add Sky Fighter: Raiden-style vertical shoot-em-up game"
- Complete game implementation in single HTML file
- Responsive design supporting mobile and desktop
- Full feature set including weapons, bombs, boss battles

## Known Characteristics
- Single-file architecture keeps all code together for easy deployment
- No external asset files (all colors, shapes rendered via Canvas)
- Inline styles for self-contained deployment
- Frame-rate dependent on browser performance
- Touch controls normalized for multi-device support

## Future Expansion Possibilities
- **Additional enemy patterns** - More variety in wave formations
- **Weapon upgrades** - Power-up system during gameplay
- **Leaderboard persistence** - LocalStorage for high score tracking
- **Sound effects** - Web Audio API integration
- **Mobile app wrapper** - Cordova/Electron packaging
- **Difficulty modes** - Easy/Normal/Hard progression
