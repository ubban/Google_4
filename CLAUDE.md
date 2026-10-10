# Sky Fighter Codebase

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire game is implemented as a single-file HTML5 application with embedded CSS and JavaScript.

- **Type**: Browser-based game
- **Language**: HTML5 + Canvas API + Vanilla JavaScript
- **File Structure**: Single `index.html` file (1669 lines)
- **Framework**: None (pure vanilla implementation)
- **Architecture**: Single-file monolithic application with procedural game loop

## Game Mechanics

### Core Features
- **Vertical Scrolling**: Player controls a ship in a top-down vertical scrolling shooter
- **Weapon System**: Multiple weapon types that can be switched during gameplay
- **Smart Bombs**: Special destructive items that clear large areas
- **Stage System**: Multiple progressively harder stages with increasing difficulty
- **Boss Encounters**: Stage bosses with unique attack patterns and health bars
- **Combo System**: Score multiplier based on consecutive hits without taking damage
- **Health System**: Player has multiple lives (hearts) with visual feedback

### UI Elements
- **HUD**: Top-center display showing:
  - Current health (hearts with visual indicator)
  - Bomb count
  - Current weapon
  - Score multiplier
- **Boss Bar**: Shows boss name and health when fighting bosses
- **Stage Banner**: Displays stage name on level transitions
- **Combo Pop**: Large text animation on screen center for combo notifications
- **Vignette Effect**: Red screen edge flash when taking damage
- **Flash Overlay**: White flash effect on bomb detonation and heavy hits
- **Top Bar**: Icon buttons for menu and settings (right side)

### Visual Style
- **Color Scheme**: Dark space theme (deep blue background with gradient sky)
- **Aesthetic**: Modern UI with glassmorphism (frosted glass effect on UI elements)
- **Animations**: Smooth transitions with cubic-bezier easing for visual polish
- **Responsive Design**: Scales to different screen sizes using `clamp()` for fluid typography

## Code Structure

The HTML file contains three main sections:

### 1. Styling (`<style>` tag)
- **Color Variables**: Dark space theme with blue and gold accents
- **Layout**: Flexbox-based, responsive design
- **Animations**: CSS keyframe animations for:
  - Heart beating (critical health indicator)
  - Screen flashing (damage/bomb effects)
  - Combo text pop
  - Stage banner entry
  - UI card entrance
  - Icon bobbing

### 2. HTML Structure (`<body>`)
- `#container`: Main game wrapper
- `#gameCanvas`: Canvas element where the game renders
- `#hud`: Heads-up display with stats
- `#overlay`: Modal overlay for menus (title screen, game over, etc.)
- `#overlayCard`: Card component inside overlay
- `#topBar`: Top-right icon buttons
- Effect overlays: `#vignette`, `#flash`, `#comboPop`, `#stageBanner`, `#bossBarWrap`

### 3. JavaScript Game Logic (embedded script)

The JavaScript section (lines ~450-1669) contains the game engine with:

#### Key Objects
- **Player**: Ship position, health, weapons, bombs
- **Bullets**: Player projectiles with position and velocity
- **Enemies**: Generic enemies spawned in waves
- **Bosses**: Boss entities with unique behaviors and attack patterns
- **Particles**: Visual effects (explosions, impacts)
- **Items**: Collectibles (weapon pickups, bombs, health)

#### Main Game Loop
- Input handling (keyboard and touch/mouse)
- Entity updates (position, collision, behavior)
- Rendering (canvas drawing)
- Particle and effect updates
- Scoring and multiplier system

#### Game States
- Title Screen
- Playing
- Boss Battle
- Stage Complete
- Game Over
- Victory

#### Control Scheme
- **Keyboard**: Arrow keys or WASD for movement, Space for fire, B for bomb
- **Touch**: Tap to set target position, game auto-aims
- **Mouse**: Click to set target position

## Development Notes

### Important Constants & Mechanics
- **Canvas Dimensions**: Responsive (fills container, see CSS)
- **Game Speed**: Frame-based with requestAnimationFrame
- **Difficulty Scaling**: Enemies and bosses get harder each stage
- **Scoring**: Base score from kills × current multiplier
- **Multiplier**: Increases with consecutive hits, resets on damage
- **Health System**: Player starts with 3-5 hearts, collectible hearts restore 1 health

### Performance Considerations
- Single canvas element renders all game elements
- Particle effects use simple rectangle/circle rendering
- No external dependencies or asset files
- Runs entirely in browser memory

### Browser Compatibility
- Requires HTML5 Canvas support
- Uses modern CSS features (clamp, backdrop-filter, custom properties)
- Touch-friendly with viewport settings optimized for mobile
- Dark color-scheme preference detection

## Code Modification Guidelines

### Adding New Features
1. **New Weapon Type**: Add to weapon data structure, add drawing logic to render function, add firing pattern to weapon handler
2. **New Enemy Type**: Create enemy class, add spawn logic, add collision detection
3. **New Boss**: Create boss class with health and attack patterns, add state machine for boss logic
4. **New Stage**: Increase difficulty multipliers, add new enemy waves in level progression

### Styling Updates
- Use CSS custom properties where possible (colors are root-level variables)
- Maintain responsive design using `clamp()` for font sizes and spacing
- Test on multiple screen sizes and aspect ratios (including mobile)
- Follow dark theme conventions (avoid bright colors without proper contrast)

### Performance Tips
- Minimize particle spawning in tight loops
- Cache frequently accessed DOM elements (they're referenced in JavaScript)
- Use requestAnimationFrame for smooth 60fps rendering
- Consider pooling game entities instead of creating/destroying frequently

## Game Assets

The game is entirely procedural - all graphics are drawn using Canvas API:
- Ships: Simple triangular/arrow shapes
- Bullets: Circles or rectangles
- Enemies: Various geometric shapes
- Bosses: Custom drawn patterns
- Explosions: Particle systems with simple shapes
- No external image, audio, or font files

## Testing & Debugging

The game includes:
- Console-based state logging possible through JavaScript console
- Visual feedback for all game events (flashes, color changes, animations)
- Responsive error handling to prevent game crashes
- Mobile/touch device compatibility built-in

## Future Enhancement Ideas

- Sound effects and music
- Persistent high score storage (localStorage)
- Leaderboard/online scores
- Additional difficulty modes
- Customizable key bindings
- Graphics settings (particle density, animation quality)
- Mobile app wrapper
- Accessibility improvements (screen reader support)
