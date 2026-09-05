# Sky Fighter Codebase

## Project Overview

Sky Fighter is a Raiden-style vertical shoot-'em-up browser game built entirely in vanilla HTML5 Canvas and JavaScript. The game runs in a single HTML file with no external dependencies, making it highly portable and easy to deploy.

**Key Features:**
- Vertical scrolling shooter gameplay
- Multiple weapons and power-ups
- Boss battles with stage progression
- Responsive design supporting desktop and mobile
- Touch controls for mobile devices
- Pixel-perfect collision detection

## Architecture

The game is implemented as a single-file HTML application (`index.html`) with three distinct layers:

### 1. **Markup & Styling** (HTML/CSS)
- Responsive canvas setup with safe area handling for mobile notch
- CSS Grid/Flex for HUD layout
- Dark space-themed color scheme
- Touch optimization (no tap highlight, crosshair cursor)

### 2. **Game Loop & Rendering** (Canvas API)
- RequestAnimationFrame-driven game loop
- Canvas rendering at native device resolution
- Layered rendering: background gradient → entities → effects → HUD

### 3. **Game Logic** (Vanilla JavaScript)
- Entity system (Player, Enemies, Projectiles, Effects)
- Collision detection between hitboxes
- Input handling (keyboard + touch/mouse)
- State management (menu, gameplay, boss, game over)
- Audio effects (Web Audio API)

## Key Game Systems

### Player Entity
- Single player controlled via arrow keys or mouse/touch movement
- Weapon system with cycling (default beam, spread, heavy)
- Limited smart bombs (screen-clear ability)
- Health/shield system with visual feedback
- Death animation and respawn logic

### Enemy System
- Spawned in waves from various entry points
- Different enemy types with distinct behaviors:
  - Basic formation flyers
  - Spinning attackers
  - Boss enemies with multiple phases
- Projectile fire and collision response

### Collision Detection
- Circular and rectangular hitbox collision testing
- Separate hitboxes for gameplay entities
- Damage application on collision
- Projectile/enemy destruction on impact

### HUD (Heads-Up Display)
- Score tracking and display
- Lives counter
- Current weapon indicator
- Smart bomb count
- Health/shield status
- Game state overlays (menu, game over, stage clear)

### Audio System
- Web Audio API for sound effects
- Weapon fire sounds
- Enemy destruction sounds
- Boss music/ambient audio
- Volume control

## File Structure

```
Google_4/
├── index.html          # Complete game implementation
├── Claude.md           # This documentation
└── README.md           # Basic project info
```

## How to Play

1. **Movement**: Use arrow keys or move mouse/touch pointer
2. **Fire**: Automatic continuous fire
3. **Weapon Switch**: Z key to cycle weapons
4. **Smart Bomb**: X key (limited uses)
5. **Mobile**: Touch controls map to pointer position

## Development Notes

### Adding New Enemy Types
1. Define enemy pattern in spawner logic
2. Set movement speed and firing rate
3. Add collision and animation handling
4. Update spawn frequency in wave system

### Adding New Weapons
1. Create weapon object with fire rate and projectile properties
2. Implement firing pattern in player fire() method
3. Add weapon indicator to HUD
4. Balance damage and fire rate

### Debugging Tips
- Browser DevTools console logs game state
- Pause at breakpoints to inspect entity positions
- Use canvas methods to draw hitboxes for collision debugging
- Monitor FPS in performance tab

### Browser Compatibility
- Requires HTML5 Canvas support
- Web Audio API for sound
- CSS Grid/Flex for responsive layout
- Touch Events API for mobile
- Tested on: Chrome, Firefox, Safari, Edge (desktop and mobile)

## Performance Optimization

- Canvas rendering at native resolution
- Object pooling for projectiles to reduce GC pressure
- Culling of off-screen entities
- Single animation frame loop
- No external library overhead

## Testing & QA

### Play Testing Focus Areas
- Wave difficulty progression
- Boss attack pattern fairness
- Touch control responsiveness
- Sound timing and sync
- Memory usage on extended play sessions
- Mobile device performance (esp. older hardware)

### Known Limitations
- No persistent high score storage (localStorage could be added)
- No pause menu (could be added)
- Mobile performance depends on device GPU
- Sound may not work in some browsers with restrictive autoplay policies

## Future Enhancement Ideas

- Save/load game progress
- Achievements system
- Additional stage/boss content
- Power-up variety (shields, speed boost, etc.)
- Leaderboard
- Settings menu (difficulty, sound, graphics quality)
- Controller support (gamepad API)

## Building & Deployment

No build process required. The game runs directly from `index.html`:

1. **Local testing**: Open `index.html` in a browser
2. **Live server**: Serve via any HTTP server (required for audio on some browsers)
3. **Deployment**: Copy `index.html` to any web hosting

## Code Quality

- Vanilla JavaScript (no frameworks or build tools)
- Inline styles for portability
- Modular game loop structure
- Clear variable naming and function organization
- Comments for complex algorithm sections
