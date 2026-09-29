# Claude.md - Sky Fighter Game Codebase

## Project Overview
**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML5 application. The entire game logic, rendering, and gameplay is implemented in one `index.html` file using vanilla JavaScript and HTML5 Canvas.

## Project Structure
```
Google_4/
├── index.html       # Complete game implementation (single-file architecture)
├── README.md        # Project documentation
└── Claude.md        # This file - agent guidance
```

## Technical Details

### Architecture
- **Single-file implementation**: All game code is in `index.html`
- **Technology stack**: HTML5, CSS3, Vanilla JavaScript
- **Rendering**: HTML5 Canvas with WebGL (if available)
- **No external dependencies**: Fully self-contained

### Game Features
- Vertical scrolling shoot-'em-up gameplay (Raiden-style)
- Player-controlled aircraft with keyboard/mouse controls
- Weapon switching system
- Smart bomb capability
- Stage bosses
- HUD display for game stats
- Responsive design with safe area support for mobile/notched devices

### Key Game Elements
The index.html contains:
- **Styling**: Complete dark-themed CSS with responsive scaling
- **Game Canvas**: 800x600 main game viewport with gradient background
- **HUD System**: Stats display (Score, Lives, Shield, Weapon level)
- **Game Logic**: 
  - Player movement and shooting
  - Enemy spawning and AI
  - Collision detection
  - Weapon and bomb systems
  - Boss encounters

## Running the Game
1. Open `index.html` in any modern web browser
2. Use keyboard or mouse to control the player aircraft
3. Game loads immediately with no build or installation required

## Development Guidelines for Claude Agents

### Code Style
- Keep the single-file architecture intact for deployment simplicity
- Use vanilla JavaScript (no external libraries)
- Maintain responsive CSS with proper mobile viewport handling
- Use CSS custom properties (`:root`) for theming and colors

### When Making Changes
1. **Preserve compatibility**: Ensure the game runs identically when opened in a browser
2. **Performance**: Canvas games are performance-sensitive; avoid unnecessary loops or DOM manipulation
3. **Testing**: Always verify gameplay feels responsive and collisions work correctly
4. **Mobile support**: Test with touch controls and handle safe-area insets (notched devices)

### Common Tasks

#### Adding New Features
- New weapons: Modify the weapon switching logic in the game loop
- New enemies: Add enemy classes/logic to the game update cycle
- Visual effects: Use Canvas drawing methods for efficiency

#### Bug Fixes
- Collision detection issues often require tweaking hitbox coordinates
- Movement/responsiveness: Check input handling and game tick rate
- Visual glitches: Verify canvas clearing and redraw order

#### Performance Optimizations
- Batch canvas operations where possible
- Minimize DOM queries (use cached references)
- Profile using browser DevTools Performance tab

## Game Controls
- **Arrow keys** or **WASD**: Move player aircraft
- **Space** or **Click**: Shoot
- **E** or **Shift**: Switch weapons
- **Q**: Deploy smart bomb

## Browser Compatibility
- Chrome/Chromium: Full support
- Firefox: Full support
- Safari: Full support (test safe-area viewport behavior)
- Edge: Full support

## Future Enhancement Ideas
- Sound effects and background music
- Multiple difficulty levels
- High score persistence (localStorage)
- Particle effects
- Additional stage bosses
- Power-up systems

## Notes for Claude Agents
- This is a self-contained game with no server/backend requirements
- The entire game state is in-memory (no persistence by default)
- Be cautious with changes that affect the game loop timing
- Always test gameplay changes in a browser before committing
- Canvas rendering is case-sensitive for drawing methods
