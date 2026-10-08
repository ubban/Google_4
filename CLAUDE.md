# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It features:
- Canvas-based rendering with responsive design
- Multiple difficulty levels (Easy, Normal, Hard)
- Weapon upgrade system with progressive fire patterns
- Boss battles with distinct attack behaviors
- Smart bomb mechanic for clearing enemy bullets
- Synthesized audio effects (beeps, noise bursts for impacts/bombs)
- Touch and keyboard/mouse controls
- Responsive HUD showing score, lives, bombs, weapon level

## Architecture

The entire game is contained in a single `index.html` file (~1,670 lines) with embedded HTML, CSS, and JavaScript. The code uses a traditional game loop pattern with clear separation of concerns:

### Game State
- `difficulty`: Current difficulty setting (Easy/Normal/Hard) affecting enemy spawning and damage
- `score`, `mult`: Score and combo multiplier (multiplier resets on damage taken)
- `health`, `maxHealth`: Player lives (typically 3)
- `bombs`, `maxBombs`: Smart bomb count
- `weapon`: Player weapon type with level progression
- `gameActive`: Boolean controlling pause/play state

### Entity System
Entities are managed as arrays:
- **Player**: Single ship object with `x`, `y`, `w` (width), `h` (height), `vx`, `vy` (velocity)
- **Enemies**: Array of small enemy ships spawned in waves; each has position, velocity, health, and firing behavior
- **Boss**: Single object that spawns at stage end; has complex attack patterns (radial, spirals, homing, etc.)
- **Bullets**: Player shots (array) and enemy shots (separate array) with velocity and collision bounds
- **Explosions**: Temporary visual particles with color variations
- **Particle Effects**: Power-up collectibles, visual feedback

### Game Loop
1. **Update Phase** (`update(dt)`): 
   - Process input, move entities, handle collisions
   - Update entity lifetimes and remove dead entities
   - Manage game state transitions (damage, power-ups, stage progression)
   - Boss AI and attack pattern timing

2. **Draw Phase** (`draw()`):
   - Clear canvas with gradient background
   - Render cloud parallax
   - Draw all entities (player, enemies, boss, bullets, explosions)
   - Update HUD elements (score, weapon, lives)

### Weapon System
The `weapon` object tracks:
- `type`: Weapon name (VULCAN, BURST, SPREAD, SPIRAL, HOMING, CROSS)
- `level`: Upgrade level (1-4) affecting fire rate and bullet count
- `fireRate`: Milliseconds between shots

Fire patterns are implemented as separate functions (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, etc.) that populate the `playerBullets` array with specific trajectories.

### Boss Patterns
Each boss progresses through attack phases. `updateBoss(dt)` manages:
- Health tracking (displayed in health bar)
- Phase transitions at health thresholds
- Attack timing and variety
- Destruction and stage advancement

Common attacks include:
- Radial spreads (bullets in all directions)
- Homing projectiles tracking player
- Spiral patterns
- Aimed spreads following player position

### Collision Detection
`rectsOverlap(a, b)` handles AABB (axis-aligned bounding box) collisions for:
- Player bullets hitting enemies/boss
- Enemy bullets hitting player
- Power-ups hitting player

### Audio System
Synthesized sounds using Web Audio API:
- `beep(freq, dur, type, vol, opts)`: Tonal sounds for shooting, power-ups, UI
- `noiseBurst(dur, vol, opts)`: Noise-based sounds for explosions, impacts
- Frequency/volume adjustments based on game events

## Development Notes

### Making Changes
- **Edit gameplay balance**: Adjust `difficulty` object properties (enemySpawnRate, enemyHealth, etc.) or `damage` values in collision handling
- **Modify weapon fire patterns**: Edit the fire pattern functions (fireRadial, etc.) to change bullet count, spread angle, or speed
- **Add boss attacks**: Extend `updateBoss()` with new phase conditions and call new fire pattern functions
- **Adjust visuals**: Modify canvas drawing code or CSS animations (canvas applies live, CSS updates require reload)

### Testing Changes
Simply refresh the browser (`F5` or reload the page). The game initializes immediately:
1. Overlay shows start menu with difficulty selection
2. Click "Start" to begin a new game
3. Use arrow keys/WASD or mouse drag (on desktop) / touch drag (on mobile) to move
4. Z or left-click to shoot
5. X or right-click to use bomb

### Performance Considerations
- Canvas rendering is main bottleneck; batch draw calls when possible
- Enemy spawn rate and bullet counts scale with difficulty
- Particle effects (explosions) are capped to avoid frame drops
- Collision checking is O(n*m) between bullets and entities; consider spatial partitioning if scaling to many more entities

### Responsive Design
- Canvas scales to fit viewport while maintaining aspect ratio
- HUD uses `clamp()` for font scaling across device sizes
- Touch controls adapt to device orientation
- Safe area insets used for notched devices

## Common Commands

**Running the game:**
- Open `index.html` in a web browser (supports all modern browsers)
- No build process, HTTP server, or dependencies required
- Works offline after initial load

**Testing workflow:**
1. Make changes to `index.html`
2. Refresh browser
3. Test gameplay through menus and difficulty levels
4. Check console for any JavaScript errors (F12 Developer Tools)

## Key File Sections

Look for these patterns when modifying code:
- **Line ~425**: Game state initialization and DOM element caching
- **Line ~468**: Helper functions (announce, trackPerf, audio, visuals)
- **Line ~741**: Difficulty settings configuration
- **Line ~875**: Game reset logic and initial entity placement
- **Line ~1022**: Enemy spawning logic
- **Line ~1043**: Boss spawning and progression
- **Line ~1124**: Weapon firing and pattern selection
- **Line ~1148**: Main update loop with input, collision, and state management
- **Line ~1304**: Power-up application logic
- **Line ~1411**: Boss update and attack patterns
- **Line ~1520**: Main draw loop with rendering order
