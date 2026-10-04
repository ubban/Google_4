# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game, built as a single-file HTML5 application. The game features an arcade-style space shooter with progressive difficulty, multiple weapon types, boss battles, and persistent score tracking.

- **Repository**: ubban/google_4
- **Main File**: `index.html` (1669 lines)
- **Type**: Web-based arcade game
- **Play**: Open `index.html` in any modern web browser

## Key Features

- **Progressive Stages**: 5 stages with increasing difficulty and unique boss battles
- **Weapon System**: Multiple weapon types that level up (basic, rapid, spread shot)
- **Smart Bombs**: Limited-use area-of-effect destruction
- **Score System**: Points, multiplier bonuses, combo tracking, and best-score persistence
- **Audio**: Sound effects and background music (toggleable)
- **Motion Controls**: Tilt-based aiming on mobile (can be disabled)
- **Pause System**: Game can be paused mid-play
- **Touch Controls**: Joystick-based movement for mobile play
- **Responsive Canvas**: Adapts to all screen sizes and resolutions

## Code Structure

The entire game is contained within a single IIFE (Immediately Invoked Function Expression) that initializes when the page loads. Key components:

### Canvas & Rendering
- Main game canvas (1200x800 base resolution, scales to fit)
- 2D context rendering with particle effects and visual overlays
- Vignette effect and flash overlays for visual feedback

### Game State Management
- `gameState`: Tracks current game phase (menu, playing, gameover)
- `score`, `best`, `health`, `bombs`, `multiplier`: Core gameplay stats
- `stage`, `stageEnemies`: Current stage and enemy tracking
- `playerWeapon`: Weapon type and level system

### Input Handling
- **Keyboard**: Arrow keys or WASD for movement, Space for shoot, B for bombs, P for pause
- **Touch**: Virtual joystick in bottom-left corner
- **Motion**: Accelerometer-based aiming (mobile devices)
- **Mouse**: Crosshair-following on desktop

### Core Game Objects
- **Player**: Position, weapon state, health, bomb count
- **Bullets**: Projectiles from player weapons
- **Enemies**: Various types with different behaviors and drop patterns
- **Bosses**: Stage-specific bosses with unique attack patterns
- **Particles**: Visual effects for explosions, hits, and impacts
- **UI Elements**: Score display, weapon info, health hearts, bomb counter

### Key Functions (Search for these in index.html)
- `update()`: Main game loop update logic
- `drawGame()`: Renders all game objects
- `handleInput()`: Processes keyboard/touch/motion inputs
- `checkCollisions()`: Collision detection for bullets, enemies, player
- `spawnEnemyWave()`: Creates enemy formation patterns
- `spawnBoss()`: Initiates boss battles
- `damage()`, `heal()`: Health modifications
- `addScore()`: Score and multiplier handling

### Audio System
- Uses Web Audio API with dynamically generated sounds
- Toggle with mute button (top-right corner)
- Sound effects for hits, weapon switches, boss actions

### Storage & Persistence
- Best score saved to `localStorage` key `'skyfighter_best'`
- Persists across browser sessions

## Making Changes

### Adding a New Enemy Type
1. Find the enemy spawning section (search for `spawnEnemyWave`)
2. Add a new enemy object with properties: `x`, `y`, `type`, `health`, `speed`
3. Define rendering and behavior in the main update loop
4. Add collision and behavior logic to `checkCollisions()` and `update()`

### Modifying Weapon Behavior
1. Search for `playerWeapon` object definition
2. Modify bullet speed, spread, fire rate, or damage values
3. Adjust `weaponLvl` multipliers for progression balance

### Adjusting Difficulty
- **Enemy spawn rate**: Modify enemy spawn intervals in stage definitions
- **Boss health**: Search for boss creation and change `health` value
- **Player health**: Modify initial `health` value in game setup
- **Score multipliers**: Adjust multiplier increment/decay rates

### Changing Visuals
- **Canvas size**: Look for `canvas.width = 1200; canvas.height = 800`
- **Colors**: Search for hex color values in draw functions
- **Particle effects**: Modify particle creation in explosion/hit handlers
- **Text rendering**: Search for `ctx.fillText` or `ctx.font` for UI text

### Audio Tweaks
- Sound generation uses frequency/duration parameters
- Modify Web Audio API calls in sound effect functions
- Adjust `audioContext.createOscillator()` frequency values

## Development Notes

### Testing Locally
1. Open `index.html` directly in a web browser (no server required)
2. Use browser DevTools (F12) to debug
3. Check console for any JavaScript errors
4. Test on multiple devices/screen sizes

### Browser Compatibility
- Requires modern browser with:
  - HTML5 Canvas
  - Web Audio API
  - LocalStorage
  - Touch events (for mobile)
  - Device motion API (for tilt controls)

### Mobile Considerations
- Game uses `touch-action: none` to prevent unwanted browser behaviors
- Virtual joystick appears on touch devices
- Screen orientation is portrait-friendly but adapts to any size
- Uses `safe-area-inset` for notched devices

### Performance Tips
- The game renders continuously via `requestAnimationFrame`
- Particle count is managed to maintain 60 FPS
- Enemies and bullets are checked for off-screen cleanup
- Canvas is sized efficiently relative to device DPI

## Important Code Locations

| Feature | Search Term |
|---------|-------------|
| Stage definitions | `const stages = [` |
| Boss spawn logic | `function spawnBoss` |
| Weapon switching | `playerWeapon.type` |
| Collision detection | `checkCollisions()` |
| Score/combo handling | `addScore(` |
| Player movement | `handleInput(` / keyboard check |
| Particle effects | `particles.push(` |
| Sound effects | `playSound(` / `audioContext` |

## Commit Guidelines

When making changes, use clear commit messages that describe what was modified:
- `Add new enemy type (Spinner)`
- `Balance boss health and spawn rate`
- `Fix collision detection edge case`
- `Improve mobile touch controls`

## Future Enhancement Ideas

- Additional weapon types or special abilities
- Power-up system with temporary boosts
- Leaderboard or online scoring
- New stage themes and boss designs
- Difficulty selector (Easy/Normal/Hard)
- Tutorial or story elements
- Controller support (gamepad API)
- Visual settings (brightness, particle density, etc.)
