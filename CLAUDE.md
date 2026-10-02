# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 application. Players control a fighter aircraft, switch weapons, drop smart bombs, and defeat stage bosses in an arcade-style gameplay experience.

## Project Structure

```
Google_4/
├── CLAUDE.md              # This file - codebase documentation
├── README.md              # Project metadata
└── index.html             # Complete game implementation (single file)
```

## Technology Stack

- **Language**: HTML5 + JavaScript (ES6+)
- **Rendering**: HTML5 Canvas API
- **Architecture**: Single-file monolithic application with game loop and state management
- **Styling**: CSS3 (embedded in HTML)
- **Browser Compatibility**: Modern browsers with Canvas support

## Core Game Architecture

### Game Loop
- Request Animation Frame (RAF) based game loop for 60 FPS gameplay
- Delta time management for frame-rate independent physics
- Update → Render cycle pattern

### Main Systems

1. **Player Aircraft**
   - Controlled via keyboard input (arrow keys / WASD or touch/mouse)
   - Weapon system with switchable weapon types
   - Health/shield system
   - Smart bomb mechanic (limited charges)

2. **Enemy System**
   - Multiple enemy types with different behaviors
   - Projectile spawning and collision detection
   - Wave-based enemy spawning

3. **Boss Encounters**
   - Stage boss mechanics
   - Boss-specific attack patterns
   - Collision and damage systems

4. **Particle & Visual Effects**
   - Explosion particles
   - Enemy death animations
   - Visual feedback for damage and hits

5. **HUD (Heads-Up Display)**
   - Score tracking
   - Health/shield display
   - Weapon indicator
   - Smart bomb counter

### Input Handling
- Keyboard input: Arrow keys (movement), Space (shoot), Z (bomb)
- Touch/Mouse: Click or tap to move/aim
- Cross-browser touch event handling
- Safe area insets for notch support

## Key Features

- **Multiple Weapon Types**: Switch between different weapons with unique fire patterns
- **Power-ups**: Weapon upgrades and health restoration
- **Smart Bombs**: Powerful screen-clearing special attack (limited uses)
- **Progressive Difficulty**: Increasing enemy complexity through stages
- **Boss Battles**: Unique boss encounters with specific patterns
- **Score System**: Persistent score tracking during gameplay
- **Responsive Design**: Scales to different screen sizes and touch devices
- **Accessibility**: Dark theme, high contrast, keyboard/touch/mouse support

## Development Notes

### Canvas Rendering
- Game renders to a full-viewport canvas element
- Aspect ratio management for different devices
- Graphics are entirely code-generated (no asset files required)
- Coordinate system: (0,0) at top-left, x increases right, y increases down

### State Management
- Game state stored in JavaScript objects
- Player state, enemy array, projectile array, particle array
- Game phase tracking (menu, playing, paused, game over)

### Performance Considerations
- Spatial partitioning for collision detection (grid-based)
- Object pooling for frequently created/destroyed objects (projectiles, particles)
- Canvas context optimization (single context per frame)

### Collision Detection
- AABB (Axis-Aligned Bounding Box) collision system
- Pixel-perfect collision for high accuracy

## Running the Game

Simply open `index.html` in a modern web browser. No build process, server, or dependencies required.

### Browser Requirements
- ES6 JavaScript support
- HTML5 Canvas API
- Touch or pointer event support (optional, for mobile)

## Customization Points

### Easy Modifications
- Difficulty scaling: Enemy spawn rates and movement patterns
- Colors: CSS variables in `:root` and canvas drawing colors
- Game balance: Weapon damage, fire rates, enemy health
- Sound: Could be added via Web Audio API

### Expansion Areas
- Leaderboard system
- Additional weapon types
- New enemy and boss designs
- Sound effects and music
- Mobile app bundling (Cordova/Capacitor)

## Git Workflow

- **Main Branch**: `main` - stable, playable version
- **Feature Branches**: Topic-specific improvements (e.g., `feature/new-weapon`)
- **Commit Style**: Clear, atomic commits describing the change
- **Testing**: Manual testing in browser before commit

## File Size & Performance

- Single HTML file (~60KB uncompressed)
- No external dependencies
- Loads instantly in browser
- Target: 60 FPS on modern devices

## Known Limitations

- Single file design limits code organization
- No persistent data storage (scores reset on page reload)
- No sound implementation (silent gameplay)
- No network features (single-player only)

## Future Enhancement Ideas

- Score persistence (localStorage)
- Audio system integration
- Multiplayer (via WebSocket)
- Advanced boss patterns
- Additional game modes
- Mobile app release
- Difficulty settings menu
- Customizable controls UI
