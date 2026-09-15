# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas game. Players control an aircraft, dodge enemy fire, collect power-ups, and defeat stage bosses. The game features multiple difficulty levels, weapon switching, bomb attacks, combo multipliers, and mobile-friendly touch controls.

### Key Features
- **Vertical shoot-'em-up gameplay** with scrolling backgrounds
- **Multiple difficulty levels** (Easy, Normal, Hard, Extreme)
- **Weapon system**: Vulcan guns and laser attacks with upgrade levels
- **Smart bombs** for emergency situations
- **Combo system** with score multipliers for consecutive kills
- **Boss battles** at the end of each stage
- **Mobile support** with touch joystick and button controls
- **Responsive design** with adaptive UI sizing
- **Audio effects** with mute support
- **High score tracking** via localStorage

## File Structure

```
Google_4/
├── index.html          # Complete game implementation (1669 lines)
├── README.md           # Project metadata
└── Claude.md          # This file
```

### Single File Architecture
The entire game is contained in `index.html`, including:
- HTML structure (canvas and UI elements)
- CSS styling (game visuals and animations)
- JavaScript game logic (game loop, physics, AI, input handling)

## Game Architecture

### Core Game Systems

#### 1. **Rendering System**
- `drawFrame()`: Main render function called every animation frame
- Canvas-based 2D rendering for all game objects
- Particle system for visual effects (explosions, trails)
- Background parallax scrolling
- Post-processing effects (vignette, screen shake, flash)

#### 2. **Physics & Collision**
- Rectangle-based collision detection (`rectCollide()`)
- Projectile physics with velocity vectors
- Player movement clamping within canvas bounds
- Enemy pathfinding with waypoint-based movement

#### 3. **Game State Management**
- `resetGame()`: Initialize new game session
- Global state variables: player, bullets, enemies, particles, powerUps, boss
- Pause/resume functionality
- Game over state tracking
- Stage progression system

#### 4. **Input Handling**
Multiple input methods supported:
- **Keyboard**: Arrow keys for movement, Space/Z for fire, B for bombs
- **Mouse**: Click to fire, hover to aim player position
- **Touch**: Joystick for movement, buttons for fire/bombs
- **Joystick input**: Virtual on-screen joystick with configurable zone

#### 5. **Entity Systems**

**Player**
- Position, velocity, collision box (40x40)
- Weapon system with cooldown
- Health/lives system
- Invulnerability frames after hit
- Speed variable for difficulty tuning

**Enemies**
- Variety of enemy types with different attack patterns
- Waypoint-based movement
- Health and damage values
- Spawn rates tied to stage and difficulty
- Enemy bullets with unique firing patterns

**Boss**
- Large health pool (difficulty-dependent)
- Multi-phase attacks
- Health bar UI display
- Stage transition trigger

**Power-ups**
- Weapon upgrades (increases weapon level)
- Rapid-fire temporary boost
- Health recovery
- Smart bomb recharge

**Projectiles**
- Player bullets (vulcan or laser based on weapon)
- Enemy bullets
- Collision response and damage

**Particles**
- Explosion effects
- Hit animations
- Visual feedback for events

#### 6. **Audio System**
- Web Audio API context (`audioCtx`)
- Procedural sound generation (no audio files)
- Sound effects for:
  - Weapon firing (varies by weapon type)
  - Explosions
  - Power-up pickup
  - Combo announcements
  - Boss events
- Master gain control with mute support

#### 7. **UI & HUD**
- Real-time score, lives, bombs display
- Weapon name and level indicator
- Combo multiplier display
- Boss health bar (when fighting boss)
- Stage banner animations
- Combo pop animations
- Pause menu with difficulty selector

#### 8. **Difficulty System**
Four difficulty levels with tuned parameters:
- **Easy**: More lives, slower enemies, lower spawn rates
- **Normal**: Baseline difficulty
- **Hard**: Aggressive enemies, high damage
- **Extreme**: Maximum difficulty with intense patterns

Each difficulty has:
- `lives`: Starting health
- `bombs`: Number of smart bombs
- `enemySpeed`: Enemy movement speed multiplier
- `enemyHealth`: Enemy durability
- `fireRate`: Enemy attack frequency

#### 9. **Scoring & Progression**
- Points awarded for enemy kills (scaled by multiplier)
- Kills per stage triggers boss battle
- Stage counter increments after boss defeat
- Combo chain system for consecutive kills
- Combo multiplier (1x to 8x) decays over time
- Best score persistence in localStorage

#### 10. **Visual Effects**
- **Screen shake**: Triggered by impacts and explosions
- **Hit vignette**: Red edge flash on player damage
- **Screen flash**: White flash for explosions
- **Particle bursts**: Multi-directional particle spreads
- **Combo animations**: Centered text pop with scale/fade
- **Stage banners**: Sliding banner for stage transitions

### Game Loop

Main update function: `update()`
1. Parse input (keyboard, mouse, joystick, touch)
2. Update player position and state
3. Spawn enemies based on spawn timer
4. Update all bullets (player and enemy)
5. Update all enemies and their AI
6. Update boss (if active)
7. Handle all collisions (player-enemy, bullets-enemies, etc.)
8. Update particles and effects
9. Update UI and HUD
10. Render frame
11. Schedule next frame via `requestAnimationFrame`

Approximate frame rate: 60 FPS (limited by browser/device)

### Control Schemes

#### Desktop (Keyboard + Mouse)
- **Arrow Keys**: Move player
- **Left Mouse Button** or **Space**: Fire weapon
- **B** key: Use smart bomb
- **P** key: Pause
- **Mouse move**: Optional aiming assistance

#### Mobile (Touch)
- **Left zone**: Virtual joystick for movement
- **Fire button** (right): Hold to fire
- **Bomb button**: Tap to use bomb
- **Pause button**: Top right corner

#### Accessibility
- Reduced motion preference support (CSS `prefers-reduced-motion`)
- Configurable difficulty levels
- Touch-friendly UI sizing
- Safe area support for notched devices
- Colorblind-friendly UI (uses shapes and text labels)

## Key Game Mechanics

### Weapon System
- **Vulcan**: Spread gun with rapid fire, good damage output
- **Laser**: Focused beam with higher single-shot damage
- Upgradeable with pickups (levels 1-3)
- Upgrade resets when hit (intentional game mechanic)

### Bomb System
- Limited use (2-3 per life depending on difficulty)
- Clears screen of enemy bullets
- Damages/kills all on-screen enemies
- Recharges via power-up pickups
- Visual indicator shows bomb count

### Combo System
- Kill counter resets when taking damage
- Each consecutive kill without damage increases multiplier
- Multiplier caps at 8x
- Multiplier decays over time if no kills
- Visual feedback with combo pop announcements
- Score calculation: `points = basePoints * multiplier`

### Boss Battles
- Triggered after stage-specific kill count
- Boss appears at top of screen with health bar
- Multi-phase attack patterns
- Increased visual feedback and urgency
- Stage advance on boss defeat
- Difficulty affects boss health significantly

### Stage Progression
- 8 stages total (or continuous after stage 8)
- Kill count per stage increases: Stage 1 needs 15 kills, scaling up
- Stage banner displays stage number
- Enemy variety and difficulty increases with each stage
- Boss difficulty escalates per stage

## Data Persistence

### localStorage Keys
- `SKY_FIGHTER_BEST`: Best score ever achieved
- `SKY_FIGHTER_MUTE`: Audio mute preference (0/1)
- `SKY_FIGHTER_DIFF`: Last selected difficulty (easy/normal/hard/extreme)
- `SKY_FIGHTER_REDUCE`: Reduced motion preference marker

## Configuration

### Quality Settings
- **Auto-detection** based on device performance
- **0 (Full)**: All particles and effects enabled
- **1 (Medium)**: Reduced particle count
- **2 (Low)**: Minimal effects, optimized for low-end devices

### Game Constants (in DIFFICULTIES object)
```javascript
easy:    { lives: 5, bombs: 3, enemySpeed: 0.7, enemyHealth: 0.8, fireRate: 1.2 }
normal:  { lives: 3, bombs: 2, enemySpeed: 1.0, enemyHealth: 1.0, fireRate: 1.0 }
hard:    { lives: 2, bombs: 2, enemySpeed: 1.3, enemyHealth: 1.2, fireRate: 0.8 }
extreme: { lives: 2, bombs: 2, enemySpeed: 1.5, enemyHealth: 1.4, fireRate: 0.6 }
```

### Canvas Size
- Responsive to viewport size
- Maintains 1:1 aspect ratio (square)
- Scales to fit within viewport while maintaining square shape

## Running the Game

1. **Open in browser**: Simply open `index.html` in any modern web browser
2. **Local server** (optional):
   ```bash
   python3 -m http.server 8000
   # Then visit http://localhost:8000
   ```
3. **No build process required**: Fully self-contained HTML file

### Browser Compatibility
- Chrome/Edge: Full support
- Firefox: Full support
- Safari: Full support
- Mobile browsers: Full support with touch controls

### Performance Notes
- Targets 60 FPS on modern devices
- Auto-scales quality down on lower-end devices
- Reasonable performance on 2-3 year old mobile devices
- Game loop uses `requestAnimationFrame` for smooth updates

## Development Notes

### Code Organization
The entire codebase is in a single file to avoid build requirements and simplify distribution. Major sections:

1. **Lines 1-200**: HTML structure and meta tags
2. **Lines 200-750**: CSS styling (game canvas, UI, animations)
3. **Lines 750-900**: Audio setup and initialization
4. **Lines 900-950**: Game state initialization
5. **Lines 950-1100**: Rendering functions (player, enemies, particles)
6. **Lines 1100-1300**: Enemy AI and spawning logic
7. **Lines 1300-1500**: Collision detection and physics
8. **Lines 1500-1669**: Main game loop and input handling

### Adding New Features
1. **New enemy type**: Add pattern to `spawnEnemies()`, define AI in `updateEnemies()`
2. **New power-up**: Add to `powerUpTypes`, handle in collision detection
3. **New visual effect**: Create in particle system or add CSS animation
4. **New difficulty preset**: Modify `DIFFICULTIES` object

### Debugging
- Open browser DevTools console to see any JavaScript errors
- Check localStorage values: `localStorage.getItem('SKY_FIGHTER_*')`
- Performance profiling via Chrome DevTools Performance tab
- Touch issues can be tested via Chrome DevTools device emulation

### Audio Context Initialization
- Web Audio API context created on first user interaction
- Essential for playing sounds on mobile (autoplay restrictions)
- `ensureAudio()` function safely initializes context

## Common Tasks

### Change Difficulty Balance
Modify values in the `DIFFICULTIES` object:
```javascript
DIFFICULTIES = {
  easy: { lives: 5, bombs: 3, enemySpeed: 0.7, ... },
  // etc.
}
```

### Adjust Spawn Rate
Modify spawn logic in `spawnEnemies()` function - look for `spawnTimer` increment conditions

### Add New Enemy Type
1. Create drawing function `drawEnemyType()`
2. Add spawn case in `spawnEnemies()`
3. Add update AI in `updateEnemies()`
4. Define health and speed in enemy object

### Tweak Score Multipliers
Look for `pointValue` calculations and `multiplier` application in collision handling

### Change Color Scheme
Modify hex colors in CSS section or update `fillStyle` values in drawing functions

## Performance Optimization Tips

- Monitor quality setting auto-detection (may need tweaking for specific devices)
- Reduce particle system load for very low-end devices
- Consider disabling parallax background on mobile if needed
- Profile with Chrome DevTools to identify bottlenecks

## Future Enhancement Ideas

- Multiple stages with varying backgrounds
- New weapon types (missiles, homing shots)
- Special attack system beyond bombs
- Boss patterns with more complexity
- Leaderboard integration
- Achievement system
- Tutorial mode for new players
- Mobile orientation lock
- Cross-device save sync (with backend)

---

**Last Updated**: 2026-09-15
**Version**: 1.0 (Initial Release)
**Author**: Claude Code Agent
