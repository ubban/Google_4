# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and JavaScript. The entire game is implemented in a single `index.html` file (~1669 lines) with no external dependencies.

**Core mechanics:**
- Player controls a fighter jet using mouse/keyboard to navigate and shoot
- Multiple weapon types with progression system
- Enemy waves with boss encounters
- Smart bomb mechanic for clearing the screen
- Health system with hearts display
- Score and weapon multiplier tracking

## Architecture

The game uses a **single-file architecture** with inline HTML, CSS, and JavaScript. The code structure within `index.html`:

### Layout Sections
1. **HTML Head** (lines ~1-45)
   - Meta tags for viewport, theme color, description
   - SVG favicon (blue fighter jet on dark background)
   
2. **CSS Styles** (lines ~46-150)
   - Dark blue theme (primary: `#0b1d3a`, background: `#05122b`)
   - Game canvas responsive sizing with aspect ratio
   - HUD styling: stats, hearts, bombs display
   - Animation keyframes: heartBeat, vignettePulse, flashHit/flashBomb
   - Safe area insets for mobile notches
   
3. **HTML Body** (lines ~150-200)
   - Single canvas element for game rendering
   - HUD container with stat displays (Score, Weapon, Multiplier, Hearts, Bombs)
   - Overlay effects: vignette (red pulse on hit), flash (white flash)

4. **JavaScript Game Logic** (lines ~200-1669)

## Key Game Systems

### Canvas & Rendering
- Fixed 400px×600px game viewport (16:10 aspect ratio)
- Responsive scaling to fill window while maintaining aspect ratio
- Gradient background: `#0d2f5e` → `#1e6fd8` → `#7fc1ff`

### Entity System
Core entities are managed as arrays:
- **Player**: Single object with position, health, fire rate tracking
- **Bullets**: Array of {x, y, vx, vy} projectiles
- **Enemies**: Array of {x, y, health, type, ...} for normal and boss waves
- **Bombs**: Array of particle effects when smart bomb detonates
- **Explosions**: Visual particles on destruction

### Player Controller
- **Position**: Center of screen (x=200), bottom area (y varies 400-580)
- **Movement**: Constrained to canvas bounds
- **Weapons**: Cycles through 5 weapon types
  - Weapon 1 (green): Basic spread shot (1x multiplier)
  - Weapon 2: Medium spread (1x)
  - Weapon 3: Wide spread (1.5x multiplier)
  - Weapon 4: Dense patterns (2x multiplier)
  - Weapon 5: Very dense spread (2x multiplier)
- **Firing**: Automatic when mouse/finger over game, fire button when not
- **Smart Bombs**: 3 available, clears most entities with screen flash

### Enemy System
- **Wave-based spawning**: Enemies spawn in coordinated waves
- **Enemy types**:
  - Small fighters: Fast, weak (1 HP)
  - Medium fighters: Moderate speed and health
  - Tough enemies: Slow, heavily armored (5+ HP)
  - Bosses: Large, distinct patterns, appear at stage end
- **Behaviors**: Patrol, dive, attack patterns vary by type
- **Drops**: Weapon pickups when destroyed

### Collision Detection
- Circle-based hitboxes for simplicity
- Player-bullet collision: Damage to enemies
- Enemy-player collision: Damage to player
- Bullet-enemy collision: Damage to enemy
- Pickup-player collision: Weapon change

### UI/HUD System
- **Real-time stats**: Score, current weapon, multiplier
- **Health hearts**: Visual representation (❤️/🩶)
- **Bombs**: Visual counter (💣)
- **Damage effect**: Red vignette pulse when hit
- **Hit flash**: White screen flash on impact

## Development Guide

### Modifying Game Balance

#### Enemy Difficulty
Located in wave spawning section - adjust:
- `enemySpawnInterval`: Time between enemy spawns (ms)
- Enemy health values: `health: 1|2|5` fields
- Enemy speed: `vy`, `vx` velocity values

#### Weapon Damage/Spread
In weapon fire code:
- Bullet damage: `damage: 1` field (default)
- Spread pattern: Adjust angle calculations in weapon type branches
- Fire rate: `fireDelay` and `fireTimer` tracking

#### Player Stats
- Starting health: `playerHealth = 4` (in player init)
- Smart bomb count: `bombCount = 3`
- Movement speed: Scale factor in mouse move handler

### Adding New Features

#### New Enemy Type
1. Add spawn branch in wave spawning code
2. Define `type`, `health`, `speed`, `color` properties
3. Add rendering in enemy draw loop: `if (e.type === 'newType') { ... }`
4. Add AI behavior in update loop

#### New Weapon
1. Add case in `currentWeapon` switch statement in fire handler
2. Define bullet spread pattern (angles, positions)
3. Set `damage` and visual appearance (color)
4. Adjust `fireDelay` for balance

#### New Visual Effect
1. Add particle array (similar to `bombs`)
2. Create particle struct: `{x, y, vx, vy, age, maxAge, ...}`
3. Add update logic (movement, fade)
4. Add draw logic in main render loop

### Code Style & Conventions

- **Variable names**: camelCase for game state (playerX, enemyWave)
- **Colors**: Hex format (e.g., `#1976d2`, `#ff4444`)
- **Physics**: Simple Euler integration (position += velocity)
- **Coordinates**: Canvas origin (0,0) at top-left, +Y downward
- **Comments**: Minimal; code is self-documenting where possible

## Deployment

**Current deployment:**
- Single HTML file served directly
- No build process required
- Responsive to all screen sizes (mobile/desktop)
- Works offline after first load (no external dependencies)

**To deploy:**
1. Commit changes to git
2. Push to remote branch
3. Host index.html on any static server
4. Browser renders at full window size

## Testing & Debugging

### Browser DevTools
- **Console**: No errors should appear during normal gameplay
- **Network**: Zero external requests (all inline)
- **Performance**: Target 60fps; monitor Canvas rendering time

### Manual Testing Checklist
- [ ] Player movement smooth and responsive
- [ ] All 5 weapons fire and cycle correctly
- [ ] Enemy waves spawn and behave as expected
- [ ] Collisions register accurately
- [ ] Smart bomb clears screen and damages all enemies
- [ ] Score increments on enemy kills
- [ ] Weapon multiplier displays correctly
- [ ] Game handles window resize smoothly
- [ ] Mobile touch controls work (tap = fire, drag = move)

## Performance Considerations

- **Canvas rendering**: Direct 2D API (no layers or intermediate buffers)
- **Entity updates**: Simple O(n) iteration per frame
- **Collision checks**: O(n²) broad-phase, acceptable for ~100 entities max
- **Memory**: All state in RAM; no external data loading

Target: 60 FPS on modern devices, 30+ FPS on older hardware.

## Known Limitations

1. **Single HTML file**: Harder to maintain as feature count grows
2. **No persistence**: Game state lost on page reload
3. **Browser-dependent**: Console audio/vibration APIs may vary
4. **Touch controls**: Simplified for mobile (no multi-touch gestures)
5. **Accessibility**: No screen reader support yet

## Future Enhancement Ideas

- Levels/stages with progressive difficulty
- Sound effects and music (Web Audio API)
- Leaderboard/high score storage (IndexedDB)
- Power-ups: shields, rapid fire, spread enhancers
- Boss character cutscenes
- Mobile app wrapper (Electron, Cordova)
- Refactor to modular JS with classes/modules if size grows
