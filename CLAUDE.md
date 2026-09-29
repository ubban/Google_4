# Claude.md – Google_4 Project Guide

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-em-up game. Players control a fighter aircraft to defeat enemies, switch weapons, deploy smart bombs, and defeat stage bosses.

- **Type**: Single-page HTML5 game
- **Technology**: HTML5 Canvas, vanilla JavaScript (no frameworks)
- **Entry Point**: `index.html` (self-contained file with embedded CSS and JavaScript)
- **Platform**: Web browser (desktop and mobile)

## Project Structure

```
Google_4/
├── index.html          # Main game file (all HTML, CSS, and JavaScript)
├── README.md           # Project timestamp/metadata
└── .git/               # Version control
```

## Running the Game

1. **Local Development**: Open `index.html` directly in a web browser
   - Use `file:///path/to/index.html` in the address bar, or
   - Serve with a local web server: `python -m http.server 8000` then visit `http://localhost:8000`

2. **Features**:
   - Canvas-based rendering (no external libraries)
   - Touch controls for mobile devices
   - Crosshair cursor
   - Dark theme with blue sky gradient background
   - HUD overlay for game stats

## Code Structure (Within index.html)

The single `index.html` file contains three main sections:

### 1. HTML Structure
- `<head>`: Metadata, title, favicon, and style definitions
- `<body>`: Container for game canvas and HUD overlay
- Game elements: `#container`, `#gameCanvas`, `#hud`

### 2. CSS Styling
- **Theming**: Dark color scheme (`#05122b`, `#0d2f5e`)
- **Canvas**: Gradient sky background (`#0d2f5e` → `#1e6fd8` → `#7fc1ff`)
- **HUD**: Positioned absolutely, with text shadow for readability
- **Responsive**: Uses `clamp()` and viewport-width units for scaling
- **Accessibility**: Safe area insets (`env(safe-area-inset-top)`) for notched devices
- **Mobile**: Touch-action settings, crosshair cursor, viewport constraints

### 3. JavaScript Game Logic
- **Canvas setup**: Dynamic sizing, high-DPI support
- **Game loop**: Using `requestAnimationFrame` for smooth 60fps rendering
- **Input handling**: Keyboard (arrow keys, spacebar) and mouse/touch controls
- **Game entities**: Player aircraft, enemies, bullets, effects, bosses
- **Collision detection**: AABB (axis-aligned bounding box) collision system
- **Rendering pipeline**: Background → entities → effects → HUD

## Development Guidelines

### For Claude Code Agents

**Permission Model**: This is a straightforward web game with no backend or external dependencies. Most changes involve:
- Modifying HTML structure (minimal)
- Updating CSS styles
- Editing JavaScript game logic

**Testing**: Always test changes by:
1. Opening `index.html` in a browser
2. Playing through a few levels or stages
3. Testing on both keyboard and touch inputs (if applicable)
4. Verifying HUD and canvas rendering

**Code Style**:
- Vanilla JavaScript (no dependencies)
- Comments: Minimal – only when the "why" is non-obvious
- Functions: Self-documenting names; no unnecessary abstractions
- Avoid premature optimization or feature flags

**Common Tasks**:
- **Balance changes**: Adjust game parameters (speed, damage, spawn rates)
- **Visual tweaks**: Modify colors, fonts, or canvas effects
- **Mechanic changes**: Update player/enemy behavior, collision rules
- **New features**: Add weapons, power-ups, enemies, or game modes

### Git Workflow

1. **Branch**: Always develop on `claude/inspiring-planck-gwetik`
2. **Commits**: Use clear, descriptive messages
   - Example: `Add smart bomb visual effect`
   - Example: `Balance: increase boss health by 20%`
3. **Push**: Use `git push -u origin claude/inspiring-planck-gwetik`
4. **PR**: Not required unless explicitly requested by user

## Performance Considerations

- **Canvas rendering**: Game loop uses `requestAnimationFrame` (browser-optimized)
- **No build step**: Pure HTML/JS/CSS – no transpilation or bundling needed
- **Mobile optimization**:
  - Touch-action constraints to prevent browser gestures
  - Viewport settings to prevent zoom/pan
  - Responsive font sizing
  - High-DPI canvas handling

## Common Patterns

### Adding Game Objects
1. Define object properties (position, velocity, size, health)
2. Update position in game loop
3. Draw to canvas context
4. Check collisions with other entities
5. Remove when off-screen or destroyed

### Event Handling
- Keyboard: `keydown`, `keyup` events (arrow keys, spacebar)
- Mouse: `mousemove`, `click` events
- Touch: `touchstart`, `touchmove`, `touchend` events
- Canvas focus management for mobile

### Canvas Operations
- `fillStyle`, `strokeStyle`: Set colors
- `fillRect()`, `strokeRect()`: Draw rectangles
- `beginPath()`, `arc()`, `fill()`: Draw circles
- `drawImage()`: Sprite rendering (if used)
- `globalAlpha`: Transparency effects

## Known Quirks / Technical Notes

1. **Single File Architecture**: All code is in one HTML file. Keep entity definitions organized by type (Player, Enemy, Boss, Bullet, Effect).

2. **Canvas Coordinate System**: Origin (0, 0) is top-left. Y increases downward.

3. **Collision Detection**: AABB system is simple but effective for rectangular objects. For circles, use distance-based detection.

4. **Mobile Considerations**:
   - Touch events may have latency
   - Ensure hit boxes are large enough for finger input
   - Test viewport constraints (`maximum-scale=1`)

5. **Browser Compatibility**:
   - Modern browsers only (ES6+ JavaScript assumed)
   - Canvas 2D context required
   - No vendor prefixes needed

## Useful References

- [Canvas API MDN Docs](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)
- [Game Development Best Practices](https://developer.mozilla.org/en-US/docs/Games)

## Contributing

When making changes:
1. Test the game in a browser after each logical change
2. Keep commits focused and descriptive
3. Avoid breaking existing features without explicit approval
4. Document any new game mechanics or balancing changes in commit messages

---

**Last Updated**: 2026-09-29  
**Branch**: claude/inspiring-planck-gwetik  
**Repository**: https://github.com/ubban/Google_4
