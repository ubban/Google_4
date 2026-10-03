# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5, Canvas, and JavaScript. Players control an aircraft, switch between weapons, deploy smart bombs, and battle stage bosses.

- **Type**: Single-page web game
- **Tech Stack**: HTML5 Canvas, Vanilla JavaScript (no frameworks)
- **Platform**: Browser-based (desktop and mobile)
- **Game Genre**: Vertical scrolling shooter (Bullet Hell style)

## Architecture Overview

The entire game is implemented as a single `index.html` file (~1669 lines) containing:
- HTML structure and UI elements
- Embedded CSS styling (dark theme, responsive, touch-optimized)
- Game logic implemented in a single IIFE (Immediately Invoked Function Expression)

### High-Level Design

The game uses a **single game loop** with these core systems:

1. **Input Handling**: Keyboard and touch controls
2. **Game State Management**: Menu, playing, pause, game over states
3. **Entity Management**: Player, enemies, bullets, effects
4. **Physics & Collision**: Simple bounding-box collision detection
5. **Rendering**: Canvas-based 2D drawing
6. **Audio**: Web Audio API for sound effects
7. **Persistence**: LocalStorage for best score tracking

## File Structure

```
Google_4/
├── index.html          # Main game file (all game code)
├── README.md          # Basic project information
└── .git/              # Git repository
```

## Core Game Systems

### 1. Player Ship
- **Health**: 3 lives (displayed as hearts)
- **Weapons**: Multiple weapon types with upgrade levels
  - Cannon (Basic)
  - Laser (Purple, rapid fire)
  - Spread (Green, multi-directional)
  - Missiles (Red, homing)
- **Smart Bombs**: Limited-use area-of-effect attacks (displayed as bomb indicators)
- **Multiplier**: Score multiplier based on consecutive hits
- **Movement**: Smooth motion with velocity-based physics (arrow keys or touch)

### 2. Enemies
- **Wave-based spawning**: Enemies appear in coordinated waves
- **Types**: Various enemy patterns with different behaviors
  - Standard enemies (single patterns)
  - Boss enemies (larger, multi-phase, telegraph attacks)
- **Drops**: Enemy defeats may drop weapon upgrades or score bonuses
- **Collision**: Simple rectangular collision with player

### 3. Bullets & Projectiles
- **Player bullets**: Limited spread, configurable fire rate
- **Enemy bullets**: Various patterns (straight, spray, tracking)
- **Visual effects**: Trails, impacts, explosions
- **Cleanup**: Off-screen bullets are removed for performance

### 4. Stage System
- **Progressive difficulty**: Stages increase in complexity
- **Stage banner**: Visual indicator when entering new stage
- **Boss encounters**: Special boss patterns at stage end
- **Wave progression**: Each stage has multiple waves

### 5. UI & HUD
Located in fixed position elements:
- **Score display**: Current score and personal best
- **Health**: Heart display (❤️)
- **Bombs**: Bomb counter (💣)
- **Weapon info**: Current weapon type and level
- **Multiplier**: Hit combo counter
- **Stage banner**: Stage number announcement
- **Combo pop**: Floating text for score events

### 6. Audio System
- **Web Audio API**: Context-based audio
- **Mute toggle**: Player can toggle sound
- **Sound effects**: Shooting, hits, explosions, boss patterns
- **Music**: Background ambience

### 7. Mobile/Touch Support
- **Touch controls**: On-screen joystick + button zones
- **Responsive layout**: Scales to viewport
- **Motion controls**: Optional device motion input
- **Gesture support**: Tap for fire, drag to move

## Key Variables & State

```javascript
// Game state
gameState       // 'menu', 'playing', 'paused', 'gameOver'
waves           // Current wave number
stage           // Current stage number
score           // Player's current score
bestScore       // Best score from localStorage

// Player state
player.x, player.y      // Position
player.health           // Lives (0-3)
player.bombs            // Smart bomb count
player.weapon           // Current weapon index
player.weaponLevels     // Array of upgrade levels per weapon
player.multiplier       // Hit combo multiplier

// Collections
bullets         // Array of player projectiles
enemies         // Array of active enemies
particles       // Array of visual effects
enemyBullets    // Array of enemy projectiles
```

## Development Guidelines

### Code Style
- **No external dependencies**: Keep it vanilla JavaScript
- **Namespace**: All code wrapped in IIFE to avoid global pollution
- **Naming**: Camelcase for variables/functions, descriptive names
- **Comments**: Add for complex logic, state transitions, or non-obvious math

### Game Loop
```javascript
function gameLoop() {
  // 1. Handle input
  // 2. Update game state
  // 3. Update entity positions
  // 4. Check collisions
  // 5. Clean up dead entities
  // 6. Render everything
  // 7. Schedule next frame
}
requestAnimationFrame(gameLoop);
```

### Adding Features
1. **New weapon type**: Add to weapon config, define bullet behavior
2. **New enemy pattern**: Create wave data in `waves` array, define `update()` and `draw()` methods
3. **New game state**: Add case to state machine in input/update logic
4. **New UI element**: Add to HTML, reference in IIFE, update in render phase

### Performance Considerations
- **Entity pooling**: Reuse bullet/particle objects where possible
- **Collision optimization**: Use spatial partitioning for large enemy counts
- **Canvas rendering**: Batch draw calls, minimize context state changes
- **Event delegation**: Use single canvas click handler vs. multiple buttons

### Browser Compatibility
- **Modern browsers**: Chrome, Firefox, Safari, Edge (2020+)
- **Mobile**: iOS Safari 13+, Chrome Android
- **Fallbacks**: No Web Audio → mute by default
- **Polyfills**: None required for target browsers

## How to Run

### Local Development
1. Open `index.html` directly in a browser
2. Game loads immediately with start menu
3. Use arrow keys or touch controls to play

### Testing
- **Desktop**: Arrow keys for movement, Space for weapon select, B for bomb
- **Mobile**: On-screen joystick for movement, buttons for actions
- **Performance**: Open DevTools (F12), check FPS/memory in Performance tab
- **Audio**: Check browser console for Web Audio errors

## Git Workflow

### Branch: `claude/inspiring-planck-5fiv6x`
This is the active development branch for Claude-assisted changes.

### Commit Guidelines
- Clear, descriptive commit messages
- Include what changed and why (game balance, bug fix, feature, etc.)
- Reference specific systems affected (e.g., "Adjust boss bullet patterns", "Fix collision detection")

### Making Changes
1. Edit `index.html` directly (it's the only source file)
2. Test changes in browser
3. Commit with meaningful message
4. Push to designated branch

## Score Progression

**Scoring System:**
- Base points per enemy type
- Multiplier bonus based on consecutive hits without taking damage
- Weapon upgrade pickups award bonus points
- Boss defeat awards significant point bonus
- Perfect stage (no damage) awards stage clear bonus

**Best Score:**
- Saved to browser localStorage
- Persists across sessions
- Displayed on game over screen

## Known Considerations

### Technical Debt
- Single file (~1669 lines) could be modularized
- No build step (consider for future optimization/minification)
- No automated tests (manual testing required)

### Performance Notes
- Canvas rendering scales linearly with entity count
- Mobile: Reduce particle effects for 60 FPS maintenance
- Large enemy waves: Monitor memory usage for garbage collection spikes

### Future Enhancement Ideas
- Additional weapon types
- Level progression system
- Leaderboard/achievements
- Sound/music additions
- Animation polish
- Enemy AI improvements
- Particle system enhancements

## Useful References

### DOM Elements (in IIFE scope)
- `canvas`, `ctx` - Main rendering target
- `overlay`, `startBtn` - Menu/game-over screen
- `scoreEl`, `bestEl` - Score displays
- `heartsEl`, `bombsEl` - Resource indicators
- `touchControls` - Mobile input system
- `paused` - Game pause state flag

### Key Functions
- `gameLoop()` - Main update/render loop
- `handleInput()` - Process keyboard/touch
- `updateEntities()` - Physics and position updates
- `checkCollisions()` - Hit detection
- `drawGame()` - Render all entities
- `playSound()` - Audio playback

## Notes for Claude Code Assistant

When working on this codebase:
1. All changes are to `index.html` only
2. Test changes immediately in browser
3. Keep game loop performance in mind
4. Maintain responsive design for mobile
5. Document complex algorithms or game balance changes
6. Use localStorage carefully (avoid data loss)
7. Audio context may need user gesture to initialize
