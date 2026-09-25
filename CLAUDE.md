# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 Canvas with vanilla JavaScript. The game features:
- Player-controlled fighter jet that scrolls vertically
- Multiple weapon types with cycling mechanics
- Smart bombs for clearing the screen
- Enemy formations and stage bosses
- Score multiplier system and combo tracking
- Mobile and desktop support with responsive controls

The entire game is contained in a single `index.html` file (~2000 lines) with embedded CSS and JavaScript.

## Architecture & Code Organization

### File Structure
```
/
├── index.html          # Complete game (HTML + CSS + JavaScript)
├── README.md           # Project overview
└── CLAUDE.md          # This documentation
```

### Code Sections in index.html

The file is organized as:

1. **HTML Head** (lines 1-208)
   - Meta tags, title, favicon, and theme configuration

2. **Styles** (lines 10-207)
   - CSS for game canvas, HUD, overlays, and animations
   - Responsive font sizing using `clamp()`
   - Dark theme with glowing effects

3. **HTML Body** (lines 209-217)
   - Game container with canvas
   - HUD elements for stats (health, score, weapon, multiplier, bombs)
   - Boss health bar
   - Visual effects (vignette, flash overlays, combo pop)

4. **JavaScript** (lines 218+)
   - Game state and configuration
   - Player, enemy, bullet, and effect classes
   - Game loop and event handlers
   - Rendering and physics systems

## Key Game Mechanics

### Player Controls
- **Arrow Keys** or **WASD**: Move player
- **Mouse/Touch**: Aim and shoot
- **Space**: Drop smart bomb (limited uses)
- **Weapon Cycle**: Automatically rotates between weapon types on each shot

### Game Systems

#### Player State
- **Health**: 3 hit points (displayed as hearts)
- **Weapons**: 4 types cycling in order (shot pattern/speed varies)
- **Bombs**: Limited smart bomb drops (clears screen and damages bosses)
- **Score Multiplier**: Increases on consecutive hits, resets on damage

#### Enemy Types
- **Fighters**: Standard enemies with simple bullet patterns
- **Scrappers**: Faster enemies with spread shots
- **Turrets**: Stationary turrets that fire at player
- **Bosses**: 3 stage bosses with larger health pools and complex patterns

#### Weapons
1. **Laser**: Fast, concentrated fire
2. **Plasma**: Wider spread, medium fire rate
3. **Missiles**: Slower, homing projectiles
4. **Cannon**: Large, powerful shots with slower rate

### Stage System
- 3 stages with increasing difficulty
- Each stage culminates in a boss battle
- Visual stage banners on transition
- Boss health bar appears during boss encounters

## Technical Details

### Key Variables & Objects

**GameState** (main game object):
- `player`: Player instance
- `enemies`, `bullets`, `effects`: Arrays for game objects
- `stage`, `score`, `kills`: Game progress tracking
- `gameOver`, `stageClear`: State flags
- `rng`: Random number generator (seeded for consistency)

**Player Class**:
- `x, y`: Position on canvas
- `health`: Current hit points
- `shield`: Temporary invulnerability after damage
- `weapon`: Current weapon type (0-3)
- `bombs`: Remaining smart bomb uses
- `mult`: Combo multiplier for scoring

**Enemy Classes** (Fighter, Scrapper, Turret, Boss):
- Inherited positioning and collision
- Wave pattern definitions
- Unique bullet patterns
- Health tracking (bosses have large pools)

### Canvas Rendering
- Fixed 600×720 canvas
- Scaled to fit viewport with maximum size constraints
- Centered on screen with hardware acceleration
- Particle effects for explosions and weapon feedback

### Performance Optimizations
- Object pooling for bullets and effects (reuse destroyed objects)
- Efficient spatial collision detection
- Particle effect culling (removed when off-screen)
- RequestAnimationFrame for smooth 60 FPS

## Running the Game

### Local Development
1. Open `index.html` in a web browser
2. Game starts immediately in a playable state
3. No server required; works offline

### Browser Compatibility
- Modern browsers with Canvas API support
- Mobile support: Touch controls for movement, tap-to-shoot
- Tested on Chrome, Firefox, Safari (desktop and iOS)

### Controls
- **Desktop**: Arrow keys or WASD to move, mouse to aim/shoot, spacebar for bomb
- **Mobile**: Touch controls (swipe to move, tap near player to shoot)

## Development Tips for Claude Code

### Adding New Features

#### New Enemy Type
1. Create a new class extending `GameState.Enemy`
2. Implement `update()` and `getDamagePattern()` methods
3. Define `waveDefinitions` for spawn patterns
4. Add to `spawnWave()` switch statement

#### New Weapon Type
1. Add weapon constant (e.g., `WEAPON_PLASMA = 2`)
2. Define bullet pattern in `getBulletPattern()` function
3. Adjust fire rate and damage values
4. Update weapon cycling logic in `onShoot()`

#### New Stage/Boss
1. Create boss class extending `Enemy`
2. Define boss health, movement, and attack patterns
3. Add stage definition to `STAGES` array
4. Create spawn logic in `nextStage()` or `spawnWave()`

### Debugging Tips
- Use `GameState.debugMode = true` to enable collision visualization (if implemented)
- Check browser console for errors
- Monitor FPS by watching `requestAnimationFrame` timing
- Test enemy spawn patterns with reduced enemy health (`Enemy.prototype.maxHealth = 1`)

### Code Style Notes
- Minimize external dependencies (pure vanilla JS)
- Use canvas transform for rotation/scale (avoid pre-rendering sprites)
- Leverage CSS animations for UI feedback
- Use GPU acceleration for smooth visual effects

## Game Difficulty & Balance

### Enemy Difficulty Scaling
- **Stage 1**: Basic fighters, slower bullets
- **Stage 2**: Mixed enemy types, increased bullet speed
- **Stage 3**: Turrets with spread patterns, boss has complex behavior

### Player Progression
- Score multiplier encourages aggressive play
- Weapon cycling adds strategy (choosing when to switch)
- Limited bombs force decision-making on bomb usage
- 3-hit health provides reasonable buffer for learning

### Scoring System
- Base score: 100 points per enemy
- Multiplier bonus: Base score × (1 + multiplier * 0.1)
- Multiplier resets on player damage, increases on consecutive hits
- Boss kills award 500+ base points

## Testing Checklist

- [ ] Game starts without errors
- [ ] Player movement works with all control methods
- [ ] All weapon types cycle and fire correctly
- [ ] Enemy spawning follows wave patterns
- [ ] Collision detection works for bullets/enemies/player
- [ ] Score calculation includes multiplier bonuses
- [ ] Bomb mechanic clears enemies and damages bosses
- [ ] Boss encounters trigger health bar display
- [ ] Stage progression works (3 stages + victory)
- [ ] Game over triggers on player health reaching 0
- [ ] Mobile touch controls respond correctly
- [ ] Canvas scales properly on different screen sizes

## Maintenance Notes

- Game data is not persisted (no save state)
- All game state is in-memory during play
- RNG is seeded for reproducible playtesting
- Visual effects are ephemeral and cleaned up automatically

---

Last Updated: 2026-09-25
