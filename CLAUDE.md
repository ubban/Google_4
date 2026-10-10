# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single HTML5 file with inline CSS and JavaScript. The game features:
- Top-down shooting mechanics with player movement
- Multiple weapon types with switchable systems
- Smart bomb mechanics
- Stage-based progression with boss encounters
- Mobile and desktop support with responsive design

## Project Structure

```
Google_4/
├── index.html          # Main game file (1669 lines) - contains all HTML, CSS, and JS
├── README.md          # Project description
└── .git/              # Git repository
```

## File Organization

### index.html (1669 lines)
Single self-contained game file with:
- **HTML structure** (lines 1-50): Document setup, viewport configuration, canvas, HUD elements
- **CSS styles** (lines 10-~500): Dark theme styling, game canvas, HUD stats display, animations, responsive layouts
- **JavaScript** (lines ~500-end): Complete game engine implementation

## Key Game Systems

### HUD (Heads-Up Display)
- **Lives display**: Hearts showing player health (5 hearts = full health)
- **Bombs display**: Smart bomb indicators
- **Weapon stat**: Current weapon type indicator (green text)
- **Multiplier stat**: Score multiplier display (orange text)
- **Responsive design**: Uses CSS `clamp()` for mobile/desktop scaling

### Canvas & Rendering
- **Canvas ID**: `#gameCanvas`
- **Background**: Gradient from dark blue to light blue (`#0d2f5e` to `#7fc1ff`)
- **Resolution**: Responsive, scales to fit viewport while maintaining aspect ratio
- **Cursor**: Crosshair for aiming

### Visual Effects
- **Vignette effect**: Red pulsing border during critical health (activated via `.on` class)
- **Flash effects**: Screen flashes on hit (`#flash.hit`) or bomb detonation (`#flash.bomb`)
- **Heart beat animation**: Critical health pulsing animation

## Game Mechanics

### Player
- Spawn position: Center bottom of screen
- Movement: Responsive to keyboard/mouse/touch input
- Health: 5 hits to defeat (can be damaged by enemy fire)
- Weapons: Switchable between multiple types

### Weapons
- Basic laser (fast, narrow spread)
- Multiple weapon types available
- Weapon switching indicated in HUD
- Multiplier increases with consecutive hits

### Enemies
- Spawn in waves
- Various movement patterns
- Drop items on defeat (weapon upgrades, bombs, score)

### Bombs
- Limited quantity (shown in HUD)
- Clear large screen areas of enemies and bullets
- Visual feedback with flash effect

## Development Guidelines

### Browser Compatibility
- Target: Modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile support: iOS Safari, Android Chrome
- Features used: Canvas API, Flexbox, CSS Grid, Touch Events, Pointer Events

### Responsive Design
- Viewport meta tag with safe-area-inset for notched devices
- CSS `clamp()` for responsive font sizing
- `max-width/max-height` for canvas to prevent distortion
- Touch action disabled (`touch-action: none`) for game input

### Color Scheme
- **Primary background**: Dark blue (`#05122b`)
- **Canvas gradient**: `#0d2f5e` to `#7fc1ff`
- **Accent colors**: 
  - Yellow/gold for HUD values (`#ffd54f`)
  - Green for weapon indicator (`#7CFC00`)
  - Orange for multiplier (`#ff8a65`)
  - Red for critical vignette effect

### Coding Conventions
- Inline styles in `<style>` tag within HTML
- CSS custom properties: `:root` defines `color-scheme: dark`
- No external dependencies - pure vanilla JavaScript
- Game state managed through JavaScript objects/classes
- Canvas-based rendering with requestAnimationFrame
- Event listeners for keyboard, mouse, and touch input

### Animation Standards
- Smooth transitions using `transition: opacity 0.3s ease`
- Keyframe animations for effects (heartBeat, vignettePulse, flashHit, flashBomb)
- Duration: 0.25s for hit flash, 0.5s for bomb flash, 0.8s for heartbeat

## Performance Considerations

- Single file reduces HTTP requests
- Inline CSS/JS eliminates parse overhead
- Canvas 2D rendering for 2D sprite-based graphics
- Efficient collision detection system
- Requestanimationframe for smooth 60fps gameplay

## Deployment

- **Method**: Static file serving
- **Requirements**: Web server (any HTTP server works)
- **No build step**: File is ready to serve as-is
- **No external resources**: All assets are embedded or generated via Canvas

## Future Enhancement Points

- Multiple difficulty levels
- Score/leaderboard persistence
- Sound effects and music
- Additional enemy patterns and bosses
- Power-up system expansion
- Replay/recording functionality
- Local storage for high scores

## Testing Considerations

- Test on multiple browsers (desktop and mobile)
- Verify touch controls work on mobile devices
- Check responsive scaling on different viewport sizes
- Validate game balance (difficulty progression)
- Performance testing on lower-end devices

---
Last updated: 2026-10-10
