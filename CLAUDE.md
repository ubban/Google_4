# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 web game. It features:
- Player-controlled aircraft with weapon switching
- Progressive difficulty with stage bosses
- Smart bomb mechanic for clearing enemies
- Score multipliers and health system
- Canvas-based 2D graphics and real-time rendering

## Code Structure

The entire game is in `index.html` - a self-contained game with:
- **HTML Structure**: Minimal (game canvas, HUD elements, container)
- **CSS Styling**: Inline styles covering UI, animations, and visual effects
- **JavaScript**: All game logic, rendering, and controls in a single `<script>` block

### Main Components

#### Game Loop
- `requestAnimationFrame()` drives the main update/render cycle
- Frame-independent delta timing for smooth gameplay

#### Game Objects
- **Player**: Controlled via mouse/touch, switches between three weapon types
- **Enemies**: Spawn waves with increasing difficulty
- **Bosses**: Special stage-ending enemies with large health pools
- **Projectiles**: Player bullets and enemy fire
- **Bombs**: Smart bomb explosion clearing nearby enemies
- **Visual Effects**: Explosions, vignette flash on damage, screen flash

#### Game State Management
- Stage progression system
- Difficulty scaling (wave count, enemy health, spawn rate)
- Score, multiplier, and health tracking
- Pause/game-over states

## Key Gameplay Mechanics

### Weapon System
Three weapon types with distinct fire patterns:
1. **Standard Spray**: Wide, rapid-fire bullets
2. **Laser Beam**: Piercing beam attacking vertically
3. **Missile**: Homing projectiles targeting enemies

Players cycle through weapons by clicking/tapping the weapon display.

### Enemy AI
- Enemies move down-screen in formation
- Simple path-following and spiral patterns
- Boss enemies with larger health pools and varied attack patterns
- Enemies fire projectiles at the player

### Difficulty Progression
Increases per stage:
- More enemies per wave
- Increased enemy health and fire rate
- Faster enemy movement
- Larger boss health pools

### Score System
- Points for destroying enemies (varied by type)
- Score multiplier (increases with consecutive kills, resets after 2s idle)
- Bonus multiplier displayed during high-scoring runs

## Development Guidelines

### Running/Testing
1. Open `index.html` in a modern web browser
2. Game auto-starts; use mouse/touch to control player
3. Click weapon display to cycle weapons
4. Press Space to activate smart bomb

### Making Changes
- **Gameplay Balance**: Adjust constants in the game initialization (damage values, spawn rates, etc.)
- **Visual Effects**: Modify CSS animations and canvas drawing code
- **Game Mechanics**: Core logic is in the update loop and object methods
- **Performance**: Monitor for frame rate issues; canvas draws at 60 FPS

### Key Constants (Top of Script)
- `CANVAS_WIDTH`, `CANVAS_HEIGHT`: Game dimensions
- `PLAYER_SPEED`: Player movement speed
- `WEAPON_*`: Damage and fire rate per weapon
- `BASE_SPAWN_RATE`: Initial enemy spawn frequency
- `STAGE_*`: Difficulty scaling parameters

### Important Patterns
- **Canvas Drawing**: All visuals via `ctx.fillRect()`, `ctx.fillCircle()`, etc.
- **Collision Detection**: Circle-to-circle distance-based checks
- **Event Handling**: MouseMove/Touch events for player control, Space key for bomb
- **Animation**: CSS keyframes for UI elements; JavaScript for game object animation

## Testing

### Manual Testing Checklist
- Player movement and boundaries
- All three weapons fire and hit correctly
- Enemy spawning and wave progression
- Bomb clears nearby enemies
- Score updates and multiplier works
- Health system and game-over state
- Boss appearance and defeat progression
- Mobile touch controls

### Performance Notes
- Game targets 60 FPS on desktop and mobile
- Canvas size is responsive to window dimensions
- Monitor for memory leaks in enemy/projectile pools

## Deployment

The game is a static HTML file - simply host `index.html` on any web server or serve locally. No build step required.

### Browser Compatibility
- Chrome, Firefox, Safari, Edge (modern versions)
- Mobile browsers (iOS Safari, Chrome Mobile)
- Requires HTML5 Canvas support

## Common Tasks

### Adjusting Difficulty
Modify `STAGE_*` constants and spawn rate multipliers to change progression speed.

### Adding New Enemy Types
1. Create new enemy constructor function
2. Add spawn logic to enemy wave system
3. Implement unique AI and attack pattern
4. Add health/damage values

### New Weapon Types
1. Add weapon definition to weapons array
2. Implement fire logic in player update
3. Adjust weapon cycling UI

### Visual Tweaks
Modify canvas drawing code or CSS for animations, colors, and effects.

---

**Last Updated**: 2026-09-24  
Maintained by Claude Code
