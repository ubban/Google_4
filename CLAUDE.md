# Sky Fighter - Codebase Documentation

## Project Overview
**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML/JavaScript application. Players control a spacecraft, switch weapons, drop smart bombs, and defeat stage bosses. The game runs entirely in the browser with no external dependencies.

### Game Mechanics
- **Player Control**: Move ship with WASD or arrow keys, aim with mouse/touch
- **Weapons**: Switch between rapid-fire lasers and spread shots
- **Smart Bombs**: Drop screen-clearing bombs (limited supply)
- **Health System**: Lives-based gameplay with visual feedback
- **Scoring**: Earn points from defeating enemies and bosses
- **Difficulty**: Progressive stages with increasingly challenging enemies and bosses

## Architecture

### Core Files
- **index.html** (~1669 lines): Complete single-file game implementation containing:
  - HTML structure (canvas container, HUD elements)
  - CSS styling (dark space theme, animations, responsive layout)
  - JavaScript game engine with ES6 classes

### Key Game Classes (in index.html)
- `Vector2`: 2D vector math utilities (position, velocity, collision)
- `Entity`: Base class for all game objects (position, velocity, rendering)
- `Player`: Player-controlled spacecraft with weapon/bomb systems
- `Projectile`: Bullets and lasers fired by player or enemies
- `Enemy`: Basic enemy AI with movement and shooting patterns
- `Boss`: Boss entities with multi-phase battles and complex attacks
- `Particle`: Visual effects (explosions, smoke, impacts)
- `Game`: Main game engine managing state, physics, rendering, and game loop

### Core Systems
- **Rendering**: 2D canvas with particle effects, screen shake, vignette effects
- **Physics**: Simple collision detection, velocity-based movement
- **Input Handling**: Keyboard (WASD/arrows) and mouse/touch aiming
- **Game State**: Stage progression, score tracking, lives/health
- **Audio/VFX**: Screen shake, flash effects, damage indicators

## Development Guidelines

### Running the Game
```bash
# Open in browser (development)
open index.html

# Or serve locally for best experience
python -m http.server 8000
# Then navigate to http://localhost:8000
```

### Code Style
- **Language**: Modern JavaScript (ES6+) with classes and arrow functions
- **No Bundling**: Single-file approach for simplicity and portability
- **Responsive**: Uses CSS clamp() and viewport units for mobile/desktop compatibility
- **Performance**: Canvas rendering with efficient collision detection

### Making Changes
When modifying the game:

1. **Game Logic Changes**: Locate the relevant class in index.html (Player, Enemy, Boss, etc.)
2. **Visual Tweaks**: Modify CSS styles or particle/animation parameters
3. **Adding Features**: Extend Entity base class or create new game systems
4. **Testing**: Open index.html in browser, test game progression and mechanics

### Important Variables and Constants
- **Canvas Size**: Dynamic based on viewport, managed in game loop
- **Game States**: PLAYING, STAGE_CLEAR, GAME_OVER, BOSS_BATTLE
- **Collision Groups**: Player, Enemies, Projectiles, Particles
- **Physics**: Acceleration, drag, max velocity for all moving objects

### Browser APIs Used
- `<canvas>` 2D rendering context for game graphics
- `requestAnimationFrame` for game loop (~60fps)
- Mouse/Touch events for player input
- Window resize events for responsive scaling

## Deployment
The game is a single HTML file and requires no build process:
1. Copy index.html to any web server
2. Access via HTTP (some features may require HTTPS for certain browsers)
3. No additional assets or dependencies needed

## Known Limitations
- Single-file structure makes code navigation challenging for large changes
- No persistent storage (progress/high scores lost on page refresh)
- No audio (visual feedback only)
- No network/multiplayer features

## Future Enhancement Opportunities
- Level editor or procedural level generation
- Sound effects and background music
- Persistent high scores (localStorage)
- Multiple difficulty levels
- Mobile app wrapper (Cordova/Electron)
- Modular refactoring (separate .js files)
