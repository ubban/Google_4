# Sky Fighter - Claude.md

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and vanilla JavaScript. The entire game is contained in a single `index.html` file with embedded CSS and JavaScript.

### Key Features
- **Player-controlled aircraft**: Navigate with mouse/touch, shoot with click
- **Enemy AI**: Multiple enemy types including fighters and tanks
- **Weapon system**: Switch between standard bullets and laser weapons
- **Power-ups**: Collect items to boost firepower and health
- **Boss battles**: Stage-based progression with boss encounters
- **Score tracking**: Persistent HUD showing score, health, ammo, and combo multipliers
- **Responsive design**: Works on desktop and mobile (touch controls)
- **Visual effects**: Glowing effects, shadow rendering based on quality settings

## Architecture

### Single File Structure
The entire game is in `index.html` (~1669 lines) containing:
1. **HTML Structure** (lines 1-100): Canvas container, HUD elements
2. **CSS Styling** (lines 101-400): Game canvas, HUD layouts, responsive design
3. **JavaScript Game Engine** (lines 401-1669): All game logic

### Key Game Systems

#### 1. **Canvas & Rendering**
- Main canvas element (`#gameCanvas`) for game graphics
- HUD overlay for score, health, ammo display
- Quality-based rendering (glow effects when quality === 0)
- Double-buffering via canvas context

#### 2. **Entity System**
Game entities are stored as arrays and objects:
- `player`: Player aircraft object with position, velocity, health
- `bullets`: Array of player projectiles (standard + laser)
- `enemies`: Array of enemy aircraft
- `enemyBullets`: Array of enemy projectiles
- `powerUps`: Array of collectible items
- `boss`: Boss object (null when inactive)

#### 3. **Player Controls**
- **Mouse/Touch**: Track pointer position (`mouseTarget`)
- **Click/Tap**: Fire weapon
- **Keyboard** (implied): Movement keys (check for arrow keys or WASD)
- **Weapon Switch**: Cycle between weapons (check for key binding)

#### 4. **Enemy AI**
- `ENEMY_TYPES` object defines:
  - Fighter: Fast, agile, low health
  - Tank: Slow, heavily armored, high health
  - Boss: Special entity with unique attack patterns
- Enemies spawn based on stage progression
- AI behaviors: movement patterns, firing intervals, health-based rendering

#### 5. **Physics & Collision**
- Simple AABB (Axis-Aligned Bounding Box) collision detection
- Projectile-enemy collision handling
- Enemy-player collision damage
- Power-up collection (proximity-based)

#### 6. **Game Loop**
Standard game loop pattern:
1. Update entity positions
2. Check collisions
3. Update game state (score, health, etc.)
4. Render all entities
5. Draw HUD

#### 7. **Stage & Progression**
- Stages increase in difficulty
- Enemy spawn patterns change per stage
- Boss appears at stage intervals
- Score multiplier/combo system

## Development Guidelines

### Adding Features
1. **New Entity Types**: Add to entity arrays and update collision detection
2. **New Weapons**: Define in weapon system, add switching logic
3. **New Enemies**: Add type to `ENEMY_TYPES`, implement AI behavior
4. **Visual Changes**: Modify canvas drawing code (search for `ctx.` calls)
5. **Game Balance**: Adjust constants in `ENEMY_TYPES`, damage values, spawn rates

### Code Locations (By Feature)
- **Rendering**: Search for `ctx.fillRect`, `ctx.fillStyle`, `drawPlane`, `drawBossShip`
- **Collision**: Look for distance calculations and overlap checks
- **Enemy AI**: `ENEMY_TYPES` object, enemy update loop
- **Player Input**: Search for `mouseTarget`, click/touch event listeners
- **Score System**: HUD drawing code, score increment logic
- **Power-ups**: Power-up spawn, collision, and effect application

### Common Tasks

#### Fix a Bug
1. Identify which system is affected (rendering, collision, AI, etc.)
2. Find the relevant section (use Ctrl+F with a descriptive search)
3. Review the logic carefully (game code often has subtle timing issues)
4. Test thoroughly with multiple playthroughs

#### Adjust Game Balance
- **Enemy difficulty**: Modify `ENEMY_TYPES` properties (hp, fireRate, damage)
- **Weapon power**: Adjust bullet damage values
- **Spawn rates**: Change enemy spawn frequency
- **Player health**: Modify max health constant

#### Optimize Performance
- Quality setting affects shadow/glow rendering
- Consider batch rendering for many entities
- Profile with browser DevTools (look for CPU bottlenecks in render loop)

## File Structure
```
Google_4/
├── index.html          # Complete game (HTML + CSS + JS)
├── README.md           # Project info
└── CLAUDE.md           # This file
```

## Testing & Running
1. Open `index.html` in a modern web browser
2. Game starts immediately
3. Use mouse/touch to aim and click/tap to shoot
4. Check browser console for any errors

## Dependencies
- **None**: Pure vanilla JavaScript with HTML5 Canvas API
- **Browser APIs**: Canvas 2D Context, Touch Events, Mouse Events
- **Compatibility**: Modern browsers (Chrome, Firefox, Safari, Edge)

## Notes for Claude Code
- The entire game logic is in one file; changes have broad impact
- Test thoroughly after any modifications to collision or rendering logic
- Performance matters for smooth gameplay (60 FPS target)
- Touch controls require special handling for mobile
- Visual feedback is important (glow effects, particle effects enhance gameplay feel)
- The game uses a simple state machine; be careful with timing-dependent logic

## Future Enhancement Ideas
- Particle effects for explosions
- Sound effects and music
- Leaderboard/high scores (localStorage)
- Multiple difficulty modes
- Additional stage themes
- Achievements/unlock system
- Multiplayer modes (if scope permits)
