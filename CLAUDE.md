# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. Players control a fighter aircraft to defeat enemies and stage bosses, with dynamic weapon switching, smart bombs, and progressive difficulty.

## Project Structure

```
Google_4/
├── index.html      # Single-file game (HTML + CSS + JavaScript)
├── README.md       # Project description
└── CLAUDE.md       # This file
```

### File Descriptions

- **index.html** (1669 lines): Complete game implementation
  - HTML structure with canvas container
  - CSS styling for responsive game layout and UI
  - JavaScript game engine (see Game Architecture below)

## Game Architecture

### Core Systems

1. **Canvas & Rendering**
   - Main game canvas element with responsive sizing
   - Dark blue gradient background (space theme)
   - Layer system: vignette (overlay), flash (damage feedback), HUD

2. **Game Loop**
   - Update phase: player input, entity movement, collision detection
   - Render phase: draw all sprites and effects
   - Uses RequestAnimationFrame for smooth 60fps gameplay

3. **Player Control**
   - Touch/mouse crosshair aiming
   - WASD/Arrow keys for movement
   - Spacebar/Touch for shooting
   - Q/1 for weapon switching
   - B/2 for smart bombs

4. **Entity System**
   - Player (health, position, weapons, bombs)
   - Enemies (multiple types, spawn patterns)
   - Bullets (player and enemy)
   - Effects (explosions, hits)

5. **Weapon System**
   - Multiple weapon types (default shot, spread, rapid fire, etc.)
   - Weapon switching with visual/audio feedback
   - Ammo/cooldown management

6. **UI Elements**
   - HUD display: health (hearts), bombs, weapon type, score multiplier
   - Dynamic updates in real-time
   - Responsive text scaling (clamp/vw units)

## Key Game Features

- **Wave-based enemy spawning** with progressive difficulty
- **Boss battles** at stage completion
- **Dynamic score multiplier** for consecutive hits
- **Screen-shake effects** for impact feedback
- **Full touch support** for mobile/tablet play
- **Responsive design** scaling from mobile to desktop

## How to Run

1. Open `index.html` in any modern web browser
2. Use mouse/touch to aim and WASD/Arrow keys to move
3. Press Spacebar/Tap to shoot, Q/1 to switch weapons, B/2 for bombs

## Development Guidelines

### Code Style

- Single-file architecture for simplicity and distribution
- Vanilla JavaScript (no frameworks or build tools required)
- CSS Grid/Flexbox for responsive layout
- Clear naming conventions for game entities and systems

### Adding New Features

1. **New Enemy Types**: Add to enemy spawning logic with custom AI behavior
2. **New Weapons**: Extend weapon system with additional shot patterns
3. **New Levels/Bosses**: Modify wave spawning and boss encounter logic
4. **Visual Effects**: Add new animation keyframes and render calls

### Browser Compatibility

- Modern browsers (Chrome, Firefox, Safari, Edge)
- Requires Canvas 2D and ES6 JavaScript support
- Touch support via PointerEvents API
- CSS Grid/Flexbox for responsive UI

## Important Patterns

### Entity Management
- Update and render methods follow a consistent pattern
- Collision detection uses bounding boxes
- Dead entities are removed from active lists

### Game State
- Single source of truth for game state variables
- Score, health, and progression tracked throughout game loop
- Win/lose conditions checked each frame

### Responsive Design
- Uses CSS `clamp()` for fluid scaling
- Safe area insets for notched devices
- Touch-action CSS properties for optimal touch handling

## Common Tasks

- **Adjust difficulty**: Modify enemy spawn rates and wave timing
- **Balance weapons**: Adjust weapon damage, cooldown, and accuracy
- **Tune physics**: Modify player speed, bullet velocity, collision sizes
- **Add animations**: Create new CSS @keyframes for visual effects

## Performance Notes

- RequestAnimationFrame ensures smooth 60fps gameplay
- Canvas rendering is optimized for 2D sprite drawing
- Collision detection uses simple bounding box checks
- Effect pooling would improve performance at scale

## Testing Checklist

Before deploying changes:
- [ ] Game starts and initializes correctly
- [ ] Player movement and aiming responsive
- [ ] Weapons fire and cycle properly
- [ ] Enemies spawn and behave correctly
- [ ] Collisions detect accurately
- [ ] UI updates display correct information
- [ ] Mobile/touch controls work
- [ ] Boss encounters trigger at correct points
- [ ] Game ends/wins properly
- [ ] No console errors

## Future Enhancements

- Sound effects and background music
- Progressive difficulty scaling
- Leaderboard system
- Additional level themes
- Power-up system
- Advanced AI patterns for bosses
