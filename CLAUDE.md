# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game. Players control a fighter aircraft to defeat enemies and stage bosses using different weapons and smart bombs.

- **Language**: HTML5, CSS3, JavaScript (ES6+)
- **Platform**: Web Browser (Modern browsers with Canvas support)
- **Architecture**: Single-page application using HTML Canvas for game rendering

## Project Structure

```
Google_4/
├── index.html          # Main game file (HTML, CSS, and JavaScript combined)
├── README.md           # Project readme
└── CLAUDE.md           # This file - Codebase documentation
```

## Key Features

- **Player-controlled fighter aircraft** with movement and shooting mechanics
- **Multiple weapons system** - Switch between different weapon types
- **Smart bombs** - Tactical bomb drops for major damage
- **Enemy waves** - Progressive enemy encounters
- **Stage bosses** - Challenging boss encounters at stage completion
- **Health/Lives system** - Player survivability mechanics
- **Score tracking** - Track player performance

## Technical Details

### File: index.html

This is a monolithic HTML file containing:

1. **HTML Structure**
   - Canvas element (#gameCanvas) for game rendering
   - HUD container (#hud) for displaying stats
   - Status indicators for health, ammo, score, etc.

2. **CSS Styling**
   - Dark space-themed color scheme (#05122b, #0b1d3a)
   - Responsive design with viewport scaling
   - HUD positioned absolutely with safe-area inset awareness (mobile support)
   - Animated gradient sky background (#0d2f5e to #7fc1ff)

3. **JavaScript Game Logic**
   - Game loop for rendering and game state updates
   - Player controller for movement and actions
   - Enemy spawning and AI systems
   - Collision detection
   - Particle effects and visual feedback
   - Score and lives management

### Game Rendering

- **Canvas API**: 2D rendering context
- **Resolution**: Dynamic based on viewport, constrained with max-width/max-height
- **Background**: Linear gradient sky effect
- **Crosshair cursor**: Visual feedback for targeting

### Input Methods

- **Keyboard Controls**: WASD or Arrow keys for movement, Space/Click for shoot, etc.
- **Touch Controls**: Support for mobile devices with touch-action: none

### Responsive Design

- Uses `clamp()` CSS function for font sizing
- Safe-area insets for notched devices
- Viewport-relative scaling with `vw` units
- Mobile-optimized with `user-scalable=no`

## Development Guidelines

### Code Style

- Use modern ES6+ features (arrow functions, const/let, destructuring)
- Keep game logic modular where possible in a single-file structure
- Use descriptive variable names for game entities (player, enemies, bullets, etc.)
- Comments for complex algorithms (collision detection, AI, particle physics)

### Game Loop Structure

Expected pattern:
```javascript
// Update game state
updatePlayerPosition()
updateEnemies()
updateBullets()
checkCollisions()
updateScore()

// Render frame
clearCanvas()
drawBackground()
drawPlayer()
drawEnemies()
drawBullets()
drawHUD()
```

### Color Scheme

- **Dark blue backgrounds**: #05122b, #0b1d3a, #0d2f5e
- **Sky blue**: #1e6fd8, #7fc1ff
- **Primary UI blue**: #1976d2
- **Text**: White (#fff) with text-shadow for readability

### Browser Compatibility

- Modern browsers (Chrome, Firefox, Safari, Edge)
- Canvas 2D API support required
- Touch Events API for mobile support
- CSS Grid/Flexbox for layout

## Common Tasks

### Running the Game

1. Open `index.html` in a web browser
2. Game starts automatically on page load
3. Use keyboard/mouse/touch controls to play

### Adding New Features

When adding features like:
- **New weapon types**: Add weapon enum/constants, update firing logic
- **New enemies**: Define enemy type, spawning pattern, AI behavior
- **Particle effects**: Add particle emitter and physics calculation
- **UI elements**: Update HUD styling and stat display

Keep all changes within the single index.html file unless modularization becomes critical.

### Performance Considerations

- Canvas drawing operations are synchronous
- Use requestAnimationFrame for smooth 60fps rendering
- Minimize DOM manipulation during game loop
- Reuse objects for particles/bullets when possible
- Optimize collision detection with spatial partitioning if needed

## Important Notes

- **Single File Architecture**: All code lives in index.html - this simplifies deployment but requires careful organization
- **Mobile Support**: The game includes mobile optimizations (safe-area insets, touch controls)
- **Accessibility**: Dark theme is enforced via `color-scheme: dark`
- **Storage**: If implementing save games, use localStorage for browser-based storage

## Future Enhancement Ideas

- Local storage for high scores
- Difficulty levels
- Power-up system
- Background music/sound effects
- Multiplayer/leaderboard (would require backend)
- More varied enemy types and boss patterns
