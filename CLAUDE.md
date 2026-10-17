# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 Canvas with vanilla JavaScript. The game features weapon switching, smart bombs, stage bosses, and a complete arcade-style gameplay loop.

### Key Information
- **Type**: Web Game (HTML5 Canvas)
- **Language**: HTML, CSS, JavaScript (vanilla, no frameworks)
- **Platform**: Browser-based
- **Entry Point**: `index.html`
- **Architecture**: Single-file monolithic structure with inline styles and scripts

## File Structure

```
Google_4/
├── index.html          # Main game file (HTML, CSS, and JS combined)
├── README.md           # Basic readme
└── .git/               # Git repository
```

## Architecture & Components

The entire game is contained in `index.html` with three main sections:

### 1. **HTML Structure**
- Container div for game layout
- Canvas element (`#gameCanvas`) for rendering gameplay
- HUD elements for displaying player stats
- Visual effect layers (vignette, flash effects)

### 2. **CSS Styling**
The stylesheet handles:
- **Layout**: Fixed full-screen game container with centered canvas
- **HUD Display**: Flex-based stats display showing:
  - Hearts (player health)
  - Bombs (smart bomb count)
  - Weapon type and level
  - Score multiplier
  - Icons and visual indicators
- **Animations**: 
  - `heartBeat` - Critical health warning animation
  - `vignettePulse` - Red vignette effect when taking damage
  - `flashHit` - Quick white flash on impact
  - `flashBomb` - Longer flash effect for bomb explosions
- **Visual Effects**: Gradients, shadows, blur effects, responsive font sizing with `clamp()`

### 3. **Game Engine (JavaScript)**

#### Core Systems
1. **Rendering System**: Canvas 2D context for all game graphics
2. **Update Loop**: Likely uses `requestAnimationFrame` for smooth 60 FPS gameplay
3. **Input Handling**: Mouse/touch controls for aiming and firing
4. **Collision Detection**: Hit detection between player, enemies, bullets, and bombs
5. **Entity Management**: Player, enemies, projectiles, effects

#### Game Mechanics

**Player Character**
- Health system (multiple hearts)
- Weapon switching capability
- Smart bomb mechanic (limited charges)
- Movement and aiming
- Collision detection with enemy fire

**Weapons**
- Multiple weapon types available
- Weapon levels/power progression
- Different fire patterns and damage values

**Enemies**
- Multiple enemy types
- Projectile attacks
- Boss encounters at stage end
- Spawn patterns and waves

**Visual Feedback**
- HUD updates for all game state changes
- Flash effects on damage/bomb use
- Vignette effect during critical health
- Real-time stat display

## Key CSS Classes and IDs

- `#container` - Full-screen game wrapper
- `#gameCanvas` - Main drawing surface
- `#hud` - Head-up display container
- `.stat` - Individual HUD stat containers
- `#hearts` - Player health display
- `#bombs` - Bomb count display
- `#weaponStat` - Current weapon info
- `#multStat` - Score multiplier display
- `#vignette` - Damage effect overlay
- `#flash` - Flash effect overlay

## Development Conventions

### Canvas Rendering
- Uses 2D context (`getContext('2d')`)
- Coordinate system: origin at top-left
- Color scheme: Dark blue background with bright UI elements
- Asset-light approach: Uses shapes, text, and CSS gradients

### Responsive Design
- Uses viewport meta tags for mobile support
- Touch-action handling disabled on canvas
- Safe-area insets for notched devices
- `clamp()` for responsive font sizing
- Max-width/height constraints on canvas

### UI/UX Considerations
- Dark theme for arcade aesthetic (`#05122b` background)
- High-contrast text with shadows for readability
- Responsive HUD that scales with viewport
- Feedback for all player actions (hit effects, bomb animation)
- Color-coded stats (yellow for score, green for weapons, orange for multiplier)

## How to Work With This Codebase

### Making Changes
1. All game code is in `index.html` - edit directly
2. Changes take effect on page reload
3. Use browser DevTools console for debugging

### Common Tasks

**Adding/Modifying Features**
- Game mechanics: Find and edit the JavaScript game engine code
- UI Changes: Modify the CSS or HTML structure
- Visual Effects: Adjust keyframes, gradients, or animation parameters

**Testing**
- Open `index.html` in a web browser
- Use browser DevTools (F12) for console errors
- Test on mobile devices for touch responsiveness

**Performance Considerations**
- Canvas rendering is the main performance factor
- Watch for excessive redraws or entity counts
- Use `requestAnimationFrame` for smooth updates
- Consider particle effect impact on frame rate

## Code Organization Tips

The JavaScript section likely contains:
- Game state variables
- Main game loop function
- Entity classes (Player, Enemy, Projectile, etc.)
- Input handling listeners
- Collision detection functions
- Rendering functions
- Animation/effects code

When adding code, maintain logical grouping and add comments for complex systems.

## Git Workflow

- Repository: `ubban/Google_4`
- Development Branch: `claude/inspiring-planck-tupzlt`
- Always commit changes with clear messages
- Push after completing features or fixes

## Future Enhancement Ideas

- Multiple difficulty levels
- Additional weapon types and power-ups
- Leaderboard/high score tracking
- Sound effects and background music
- Additional stage bosses
- Save/resume functionality
- Mobile-optimized touch controls
