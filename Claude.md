# Sky Fighter Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file browser-based game where players control a fighter aircraft, switch between different weapons, use smart bombs to clear the screen, and battle through stages with bosses.

- **Type**: Browser-based game (HTML5/Canvas)
- **Language**: JavaScript (inline in HTML)
- **Main File**: `index.html` (~1,669 lines)
- **Entry Point**: Open `index.html` in a web browser

## Project Structure

```
Google_4/
├── index.html          # Complete game implementation (HTML, CSS, JS)
├── README.md           # Basic project info
└── .git/               # Git repository
```

## Game Architecture

The game is implemented entirely within `index.html` with three main sections:

### 1. HTML Structure
- Single container div with canvas for rendering
- HUD (Heads-Up Display) elements for stats (score, health, ammo, weapon type)
- Fullscreen and volume controls

### 2. CSS Styling
- Dark space theme with gradient background (`#05122b` to `#1e6fd8` to `#7fc1ff`)
- Responsive design with viewport units (`clamp()` for scaling)
- Safe area insets for mobile notches
- Touch-friendly UI with removed tap highlights

### 3. JavaScript Game Engine
The core game logic includes:

#### Key Systems:
- **Canvas Rendering**: 2D canvas-based rendering at adaptive resolution
- **Sprite System**: Player, enemies, bosses, bullets, explosions
- **Collision Detection**: Rectangular bounding box detection
- **Audio System**: Background music, sound effects with volume control
- **Input Handling**: Keyboard (arrow keys, space, Q, E) and touch/mouse controls
- **Particle Effects**: Explosions, visual feedback

#### Game Objects:
- **Player**: Controllable aircraft with health, position, selected weapon
- **Weapons**: Multiple weapon types (e.g., bullets, missiles) with ammo counts
- **Enemies**: Spawned in waves with AI behavior
- **Bosses**: Stage bosses with unique patterns
- **Projectiles**: Player bullets and enemy fire
- **Particles**: Explosion and visual effects

#### Game States:
- Playing/Running
- Paused (P key)
- Game Over
- Boss encounters

#### Controls:
- **Arrow Keys**: Move player
- **Space**: Fire weapon
- **Q**: Cycle weapon backwards
- **E**: Cycle weapon forwards
- **Z**: Drop smart bomb (limited uses)
- **P**: Pause/unpause
- **M**: Mute/unmute audio

## Development Guidelines

### Code Style
- Functional, procedural style with event listeners
- Direct canvas manipulation for rendering
- Frame-based game loop using `requestAnimationFrame`

### Adding Features
1. **New Weapons**: Add to weapon definitions array, implement firing logic
2. **New Enemies**: Create in enemy spawning system with AI patterns
3. **New Bosses**: Define boss objects with health, attack patterns, reward
4. **Visual Effects**: Add to particle system or use existing explosion effects

### Performance Considerations
- Avoid creating objects unnecessarily in the game loop
- Use object pooling for bullets and particles if needed
- Keep collision detection efficient with spatial partitioning if complexity grows

### Testing
- Open `index.html` directly in a browser
- Test on multiple devices (desktop, mobile, tablet)
- Verify controls respond correctly
- Check audio plays as expected
- Validate collision detection accuracy

## Technologies Used

- **HTML5**: Semantic markup, canvas element
- **CSS3**: Flexbox, gradients, viewport units, safe area support
- **JavaScript (ES5/ES6)**: Game loop, event handling, canvas API
- **Web Audio API**: Background music and sound effects

## Common Tasks

### To Run the Game
```bash
# Open directly in a browser
open index.html  # macOS
# or
firefox index.html  # Linux
# or
google-chrome index.html  # Windows/Linux
```

### To Deploy
- Upload `index.html` to any web server
- No build step required
- Works completely standalone

### To Modify Game Balance
- Adjust enemy spawn rates in enemy spawning logic
- Modify player health, weapon damage in their definitions
- Change projectile speed in movement code
- Tune boss health and attack frequency

## Notes for Claude Code

- The entire codebase is in one file, making it easy to navigate
- Game state is managed through global variables and game objects
- No external dependencies or build tools required
- All assets are embedded (SVG icons, inline styles)
- Audio is likely handled through the Web Audio API or HTML5 audio elements
- The game adapts to different screen sizes using viewport units and canvas scaling

## File Size
- Total: ~1,669 lines (HTML, CSS, and JavaScript combined)
- Single file keeps code simple and self-contained
