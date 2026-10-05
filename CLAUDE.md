# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript, HTML5 Canvas, and CSS. Players navigate a fighter aircraft through vertically-scrolling levels, switching between weapons, dropping smart bombs, and defeating stage bosses.

The game runs entirely in-browser with no external dependencies. It features:
- Responsive canvas rendering optimized for desktop and mobile
- Multiple weapon types with cooldown systems
- Smart bomb mechanics with screen-clearing effects
- Enemy waves, patterns, and boss encounters
- Score tracking and multiplier system
- Health/lives system with visual feedback (hearts and vignette effects)

## Architecture

The entire game is implemented in a single `index.html` file containing:

### Structure
- **HTML5 Canvas (`#gameCanvas`)**: Primary rendering surface for all game graphics
- **Game HUD (`#hud`)**: Display of stats (health, score, weapon, multiplier, bombs)
- **Visual Effects Layers**:
  - `#vignette`: Red pulsing overlay for damage/critical state
  - `#flash`: White flash effect for hits and bombs

### Game Loop
- Runs at 60 FPS using `requestAnimationFrame`
- Updates all entity positions, velocities, and state
- Handles collision detection
- Renders all visuals each frame

### Key Game Systems

**Player (Fighter Aircraft)**
- Position: center-bottom of screen
- Health: 3 lives (displayed as hearts ❤️)
- Controls: Mouse/touch for aiming, click to shoot
- Weapons: Cycles through available weapon types (visual feedback via HUD)
- Bombs: Limited smart bombs with cooldown and visual availability indicator

**Enemies**
- Spawned in waves with varying patterns
- Each has position, velocity, health, and collision box
- Drops items (score bonuses, weapon pickups, bomb refills)
- Visual feedback: damage flash, destruction animation

**Weapons System**
- Default weapon: Standard bullets
- Alternate weapons: Spread shots, lasers, missiles (example structure ready for expansion)
- Cooldown-based firing to prevent spam
- Weapon type rotates on pickup or keyboard input
- HUD shows current weapon name and cooldown percentage

**Bombs**
- Smart bomb mechanic: clears screen, damages/destroys nearby enemies
- Limited supply (display shows available bombs)
- Cooldown between uses
- Visual effect: bright white flash across entire screen

**Scoring**
- Points awarded for destroying enemies
- Multiplier system: increases with consecutive kills
- Multiplier decays if player takes damage or pauses
- Score displayed in top HUD

**Collision Detection**
- Rectangle-based bounding boxes for player, enemies, projectiles
- Pixel-perfect collision not implemented (AABB sufficient for gameplay feel)

## Running the Game

### Development
Simply open `index.html` in a modern web browser:
```bash
# Using a local server (recommended for better compatibility)
npx http-server .
# Then navigate to http://localhost:8080
```

Or drag-and-drop `index.html` into your browser.

### Testing
- **Desktop**: Use mouse to aim, click to fire, keyboard shortcuts for weapon/bomb selection
- **Mobile/Touch**: Touch to aim, tap to fire
- **Game Balance**: Lives, weapon balance, enemy difficulty can be tuned via constants at top of `<script>` section

## Code Guidelines

### When Making Changes

1. **Keep the single-file structure**: All code stays in `index.html`. Avoid extracting to separate files unless the file grows beyond 2000 lines.

2. **Naming conventions**:
   - Game entities (Player, Enemy, Projectile): `PascalCase` for class-like structures
   - Variables/functions: `camelCase`
   - Constants: `UPPER_SNAKE_CASE`
   - DOM elements: Prefix with `#` when referring to IDs in comments

3. **Comments**: Only document WHY, not WHAT. Well-named variables already show WHAT they do.
   - Exception: Complex collision logic, game balance constants, or non-obvious timing tricks merit a one-line explanation.

4. **No external dependencies**: Keep all code vanilla JavaScript. No jQuery, frameworks, or libraries.

5. **Browser compatibility**: Target modern browsers (ES6+, CSS3). Mobile-first responsive design for canvas scaling.

### Modifying Game Balance

Look for these constants near the top of the script:
- `PLAYER_HEALTH`: Starting lives
- `PLAYER_SPEED`: Movement speed
- `WEAPON_COOLDOWNS`: Fire rate per weapon
- `BOMB_COOLDOWN`: Time between bomb uses
- Enemy spawn rates, patterns, and health values

### Adding Features

**New Weapons**:
- Add weapon object to `WEAPONS` array with name, cooldown, fire pattern
- Implement firing logic in the weapon cycling section
- Update HUD to display new weapon name

**New Enemy Types**:
- Define enemy spawn pattern
- Implement movement and firing logic in enemy update loop
- Test collision and health handling

**Visual Polish**:
- Add particle effects using canvas drawing operations
- Modify color schemes in CSS
- Adjust animation timings for feel

## Performance Notes

- Canvas rendering is optimized for 60 FPS on modern devices
- No memory leaks expected with proper cleanup of destroyed entities
- Tested responsive scaling from mobile (320px) to 4K displays
- Safe area insets (`env(safe-area-inset-*)`) handle notches on mobile devices

## Known Limitations

- No persistent high score storage (can add localStorage if needed)
- No sound effects (audio could be added with Web Audio API)
- No pause menu (could implement with overlay and game state flag)
- Mobile performance on older devices not optimized

## Future Enhancement Ideas

- Progressive difficulty scaling across levels
- Leaderboard system with local storage
- Multiple difficulty modes
- Power-up system beyond weapons
- Particle effects library
- Sound/music integration
- Controller support (gamepad API)
