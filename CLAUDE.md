# Sky Fighter — Codebase Guide

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas. Players control a fighter jet, switch weapons, deploy smart bombs, and face stage bosses in a fast-paced arcade experience.

**Repository**: ubban/google_4  
**Language**: JavaScript (vanilla, no frameworks)  
**Deployment**: Single HTML file (`index.html`) — can be served statically or embedded

## Architecture

### Single-File Structure
- **`index.html`** — Complete game implementation: HTML5 structure, embedded CSS, and all JavaScript game logic
- **`README.md`** — Project metadata (timestamp)

### Game Engine Architecture
The game uses a canvas-based render loop with these core subsystems:

#### 1. **Core Systems**
- **Game State**: Tracks game mode (menu, playing, paused, game-over), score, level progression
- **Input Handler**: Listens to keyboard, mouse, and touch events for player movement and weapon/bomb selection
- **Render Loop**: RequestAnimationFrame-driven update and draw cycle
- **Physics**: Simple position/velocity updates for entities

#### 2. **Entity System**
- **Player Ship**: Positioned at screen bottom, moves horizontally, fires bullets and drops bombs
- **Enemies**: Fixed formations or wave-based spawn patterns, move in defined patterns, drop loot
- **Bullets**: Player projectiles with collision detection
- **Enemy Bullets**: Incoming fire that damages the player
- **Explosions/Effects**: Visual feedback for hits and destruction
- **Loot/Power-ups**: Dropped by destroyed enemies (weapon upgrades, bomb ammo, shields)

#### 3. **Combat System**
- **Weapon Types**: Multiple weapon slots (default gun, spread shot, laser, etc.); switch via keys/UI
- **Smart Bombs**: Limited-count AOE clearing mechanic
- **Damage/Collision**: Pixel-perfect or bounding-box collision detection
- **Health System**: Player has limited health; enemies have varying HP

#### 4. **Stage/Boss System**
- **Wave Spawner**: Procedural or pre-defined enemy patterns per stage
- **Boss Encounters**: Unique boss entities with complex movement and attack patterns
- **Progression**: Clear stage → advance to next with increased difficulty

#### 5. **UI/HUD**
- **Score Display**: Current score, high score
- **Weapon/Bomb Status**: Current weapon and bomb ammo count
- **Health Bar**: Player ship remaining health
- **Game Menu**: Start, resume, settings screens

## Key Code Patterns

### Game Loop
```javascript
function gameLoop() {
  update();  // Update all entity states
  draw();    // Render to canvas
  requestAnimationFrame(gameLoop);
}
```

### Entity Updates
Entities (player, enemies, bullets) follow this pattern:
- Store position (`x`, `y`), velocity (`vx`, `vy`), and state (active, hp, etc.)
- Update position: `x += vx; y += vy`
- Check boundaries and collision
- Mark as inactive if destroyed or out-of-bounds

### Collision Detection
- Typically uses axis-aligned bounding boxes (AABB) or circle collision
- Called during update phase; applies damage/effects when collisions detected

### Rendering
- Clear canvas each frame
- Draw background/parallax layers
- Draw entities (enemies, bullets, player) with sprites or shapes
- Render HUD/UI overlay

## Development Guidelines for Claude

### Code Style
- **No external dependencies** — keep vanilla JS; maintain single-file structure
- **Modular functions** — break game logic into focused functions (spawn enemies, update bullets, render HUD, etc.)
- **Clear variable names** — avoid single letters except loop counters
- **Comments** — explain complex collision logic, spawn patterns, and game state transitions

### When Adding Features
1. **Keep the single-file structure** unless there's a compelling reason to split
2. **Don't add build tools or transpilers** — stay compatible with direct HTML5 Canvas APIs
3. **Test in browser** — always verify changes work visually (collision, movement, rendering)
4. **Responsive design** — maintain mobile touch support and adapt to screen sizes
5. **Performance** — keep collision checks and draw calls optimized; profile with DevTools if adding many entities

### Testing Checklist
- [ ] Game starts and menu displays correctly
- [ ] Player can move left/right and stay in bounds
- [ ] Bullets fire and move toward top of screen
- [ ] Enemy waves spawn and follow defined patterns
- [ ] Collisions register (enemy-bullet, player-enemy, etc.)
- [ ] Score increases on enemy destruction
- [ ] Weapons/bombs switch and function correctly
- [ ] Game over/win conditions trigger as expected
- [ ] Works on both desktop (keyboard) and mobile (touch)

## Common Workflows

### Running the Game
Simply open `index.html` in a web browser (desktop or mobile). The canvas will fill the viewport and the game starts immediately or shows a menu.

### Debugging
- Use browser DevTools (F12) → Console for logs
- Check Network tab for any async loads (if added)
- Use Canvas debugging: draw bounding boxes temporarily, log entity positions

### Adding a New Weapon
1. Define weapon properties (projectile speed, spread, damage)
2. Add weapon type to the weapon switch logic
3. Implement spawn logic in the fire function
4. Test trajectory and collision with enemies

### Adding a New Enemy Type
1. Define enemy class/object with `x`, `y`, `vx`, `vy`, `hp`, `sprite/shape`, etc.
2. Add to enemy spawn patterns in wave logic
3. Implement movement update (linear, circular, sine wave, etc.)
4. Implement collision damage and destruction
5. Add loot drop on death

### Adding a Boss
1. Create boss entity with large health pool and complex movement
2. Define attack patterns (bullet spreads, tracking, etc.)
3. Trigger boss battle at stage milestone
4. Design unique visual/audio cues (if sound is added)

## Git Workflow
- **Branch**: `claude/inspiring-planck-ybk7ka` — development branch for Claude
- **Commits**: Use clear, descriptive messages (e.g., "Add spread-shot weapon", "Fix enemy collision bounds")
- **Push**: `git push -u origin claude/inspiring-planck-ybk7ka` when ready

## Performance Considerations
- **Entity Pooling** — Reuse bullet/enemy objects instead of creating new ones to reduce GC pressure
- **Collision Optimization** — Only check collisions between active entities; use spatial partitioning for many entities
- **Canvas Rendering** — Minimize canvas state changes; batch similar draws
- **Touch Optimization** — Debounce touch events; use `touch-action: none` in CSS (already set)

## Future Enhancement Ideas
- Sound effects and background music
- Power-ups system (shields, rapid-fire, bomb refills)
- Leaderboard storage (localStorage or remote)
- Difficulty selector
- Mobile app packaging (Cordova, Capacitor)
- Particle effects for explosions
- Boss patterns as data-driven configurations
- Tutorial/controls overlay

## Related Files
- **index.html** — Full game source (HTML + CSS + JS)
- **.git/** — Git repository metadata
- **README.md** — Project description

---

**Last Updated**: 2026-10-16  
**Maintainer**: Claude Code Agent
