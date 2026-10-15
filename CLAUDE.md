# Sky Fighter - CLAUDE.md

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, Canvas, and JavaScript. The game features weapon switching, smart bombs, stage bosses, and classic arcade-style gameplay.

## Technology Stack

- **Frontend**: HTML5 Canvas API
- **Language**: Vanilla JavaScript (no frameworks)
- **Assets**: Inline SVG for UI elements, procedurally generated game graphics
- **Styling**: CSS-in-JS with viewport-relative sizing (clamp, vw units)

## Project Structure

```
Google_4/
├── CLAUDE.md              # This file - codebase documentation
├── README.md              # Project title and metadata
├── index.html             # Single-file game implementation
└── .git/                  # Git repository
```

## Key Architecture

### Single-File Design
The entire game is contained in `index.html`, combining:
- HTML markup with a game canvas and HUD container
- CSS styles (dark theme, responsive layout, safe-area support for mobile)
- JavaScript game engine and logic

### Game Structure

#### Canvas & Rendering
- **Element**: `<canvas id="gameCanvas">` - main game viewport
- **Responsive**: Scales to fit viewport with max constraints
- **Background**: Gradient from dark blue to light cyan sky effect
- **HUD**: Fixed overlay showing stats (health, weapons, stage, score, etc.)

#### Game Loop
- Uses `requestAnimationFrame` for smooth 60fps rendering
- Handles input (keyboard, mouse, touch)
- Updates game state (entities, collisions, scoring)
- Renders all visual elements

#### Input Handling
- **Keyboard**: Arrow keys or WASD for movement, Space for shooting
- **Mouse**: Click/tap to aim and shoot
- **Touch**: Multi-touch support with viewport-relative positioning
- Respects `touch-action: none` for game control

#### Game Entities
- **Player**: Controllable aircraft with position, health, weapons
- **Enemies**: Multiple types with pathfinding and attack patterns
- **Weapons**: Different firing patterns (bullets, sprays, homing)
- **Smart Bombs**: Area-of-effect clearing mechanic
- **Boss Encounters**: Stage-specific boss fights with unique attack patterns
- **Effects**: Visual feedback (explosions, screen shake, particles)

#### Gameplay Features
- **Stage Progression**: Multiple levels with increasing difficulty
- **Weapon System**: Switch between different weapon types during gameplay
- **Health/Shield**: Player damage tracking and lives system
- **Scoring**: Points from enemies, combos, boss defeats
- **Collision Detection**: Pixel-perfect or bounding-box based collisions

#### Styling Highlights
- **Dark Theme**: `color-scheme: dark` with `#05122b` background
- **Responsive**: Uses `clamp()` for fluid scaling across device sizes
- **Safe Area Support**: `env(safe-area-inset-*)` for notch/cutout handling
- **Performance**: Hardware acceleration via canvas, `transform` properties
- **Mobile Optimized**: Removes tap highlights, disables text selection, custom cursor

## Development Guidelines

### Code Conventions
- Vanilla JavaScript - no external dependencies
- Single file keeps deployment simple; modularize if game logic grows substantially
- Use `const/let` for variable declarations
- Prefer functional/pure patterns where possible

### Performance Considerations
- Canvas rendering is the main bottleneck; batch draw calls where possible
- Use `requestAnimationFrame` instead of `setTimeout` for game loop
- Avoid DOM manipulation in the game loop (use canvas only)
- Profile with browser DevTools to identify hot paths

### Responsive Design
- Use viewport-relative units (`vw`, `clamp()`) for responsive sizing
- Test on mobile (portrait/landscape) and desktop viewports
- Ensure touch targets are adequate (min 44x44px for mobile)
- Verify safe-area insets work on devices with notches

### Browser Compatibility
- Modern browsers (Chrome, Firefox, Safari, Edge)
- HTML5 Canvas API support required
- ES6+ JavaScript features are used (arrow functions, destructuring, etc.)

## How to Run

1. **Open in Browser**: Simply open `index.html` in a web browser (file:// protocol works)
2. **Local Server** (optional): `python -m http.server 8000` then visit `http://localhost:8000`
3. **Deployment**: Serve as static HTML file - no server-side code required

## Testing

- **Manual Testing**: Play the game, verify all game modes work
- **Input Testing**: Test keyboard, mouse, and touch controls across devices
- **Performance**: Monitor FPS with browser DevTools, check CPU usage
- **Responsive**: Test on multiple screen sizes (mobile, tablet, desktop)

## Key Files to Modify

- **Game Logic**: All in `<script>` section of `index.html`
  - Entity management
  - Collision detection
  - Input handlers
  - Game state machine (menus, gameplay, game-over)
  
- **Styling**: CSS in `<style>` section
  - HUD layout
  - Canvas sizing
  - Responsive breakpoints
  
- **HTML**: Markup in `<body>` section
  - Canvas element
  - HUD containers
  - Safe-area support

## Common Tasks

### Adding a New Enemy Type
1. Define enemy class/object in game logic
2. Implement `update()` and `render()` methods
3. Add spawn logic in stage/wave definition
4. Test collision behavior with player bullets

### Adding a New Weapon
1. Define weapon properties (fire rate, damage, spread pattern)
2. Implement firing logic in player weapon system
3. Add UI indicator in HUD
4. Balance damage vs weapon availability

### Adjusting Difficulty
1. Modify enemy spawn rates and patterns
2. Adjust enemy health/damage values
3. Change weapon availability and power-ups
4. Tune boss encounter parameters

### Fixing Performance Issues
1. Check enemy/bullet count in DevTools Console
2. Profile with Performance tab (flame graphs)
3. Reduce draw call complexity if needed
4. Use object pooling for frequently created entities

## Notes for Claude Code

- This is a single-file HTML5 game - keep it self-contained unless substantial growth occurs
- Test game functionality manually in a browser (Canvas rendering requires visual verification)
- Responsive design is critical - changes should be tested on mobile devices
- Performance matters - avoid unnecessary DOM operations during gameplay
- Comments in game code should explain non-obvious logic (game physics, collision math, etc.)
