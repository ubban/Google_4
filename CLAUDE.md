# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (~1,669 lines).

## Architecture

### Single-File Structure
- **index.html**: Contains all HTML markup, embedded CSS styles, and JavaScript game code within a single `<script>` tag
- All code runs in an IIFE (Immediately Invoked Function Expression) for scope isolation
- No build step, bundler, or external dependencies required

### Game Systems

#### Core Loop (line 1640)
- **Main game loop**: Uses `requestAnimationFrame(loop)` for 60fps target
- **update(dt)** (line 1148): Handles game logic each frame
  - Player movement and shooting
  - Enemy spawning and AI
  - Collision detection
  - Score/life/bomb management
- **draw()** (line 1520): Renders all game elements and UI/HUD
- **updateBoss(dt)** (line 1411): Separate boss-specific logic

#### Entity Systems
- **Player**: Single controllable plane with position, velocity, weapons, lives
- **Enemies**: Multiple enemy types defined in `ENEMY_TYPES` (line 502)
  - Grunt: Basic enemy
  - Interceptor: Fast, evasive
  - Tank: High HP, slow
- **Bosses**: Six pre-designed boss encounters defined in `BOSSES` (line 508)
  - Each boss has unique appearance, HP multiplier, behavior
  - Boss spawns after `KILLS_PER_STAGE` (18) enemy kills
- **Projectiles**: Player bullets and enemy fire tracked in arrays
- **Particles**: Explosions and visual effects using `spawnExplosion()` (line 1073)

#### Game State (line 865-867)
- `score`, `lives`, `maxLives`, `running`, `paused`, `gameOver`
- `kills`, `stage`, `stageKills`, `bossWarned`
- `difficulty` (easy/normal/hard with different parameters in `DIFFICULTIES`)
- Weapon tracking: name, level, ammo pools
- Combo/multiplier system for consecutive hits

#### UI/HUD
- Health hearts display (line 680)
- Bomb counter (line 690)
- Score, best score, weapon status
- Boss health bar when boss is active
- Game over/start overlay with difficulty selector
- Touch controls for mobile (on-screen joystick and fire/bomb buttons)

#### Controls
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, E for bomb
- **Touch/Mobile**: Virtual joystick on left, fire/bomb buttons on right
- Motion sensors supported for tilt controls (line 529)
- Pause button available during gameplay

#### Visual Systems
- **Themes**: Six different sky/cloud color schemes defined in `THEMES` (line 493)
- **Drawing functions**:
  - `drawPlane()` (line 913): Renders player and enemy planes
  - `drawBossShip()` (line 970): Renders boss with unique visual
  - `drawClouds()` (line 1507): Scrolling parallax background
- **Adaptive quality** (line 474-483): Tracks frame times and downgrades rendering quality if FPS drops below 40

#### Audio
- Mutable sound effects system controlled via mute button
- localStorage persists mute state

#### Data Persistence
- Uses localStorage for:
  - Best score (`skyfighter.bestScore`)
  - Muted state (`skyfighter.muted`)
  - Difficulty setting (`skyfighter.difficulty`)
  - Reduced motion preference (`skyfighter.reducedMotion`)

#### Difficulty Tuning
The `DIFFICULTIES` object (line 485) controls enemy spawn rate, speed, AI aggression, lives, bombs, and boss health for each difficulty level.

## Development

### Testing the Game
Since there's no build step, simply open `index.html` in a web browser:
- Desktop: Open the file directly in Chrome, Firefox, Safari, or Edge
- Mobile: Serve the file via HTTP and access from a mobile device to test touch controls

### Browser DevTools
- **Console**: Check for any runtime errors
- **Network tab**: Verify no external resources are failing (game uses no external dependencies)
- **Performance profiler**: Profile the update/draw loops to identify bottlenecks
- **Debugger**: Set breakpoints in the JavaScript code

### Common Editing Tasks

#### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (around line 502)
2. Update spawn logic in `spawnEnemy()` (line 1022) to include conditions for spawning this type
3. Ensure `drawPlane()` renders it correctly
4. Update minStage if it shouldn't appear until later

#### Modifying Difficulty Tuning
Edit the `DIFFICULTIES` object (line 485) to adjust:
- `spawnBase` / `spawnMin`: Enemy spawn rates
- `enemySpeed`: Multiplier for enemy movement speed
- `enemyShootChance`: Probability enemies fire each frame
- `lives` / `bombs`: Starting resources
- `bossHpMul`: Boss health multiplier

#### Adding Visual Themes
Add an object to the `THEMES` array (line 493) with:
- `sky`: Array of three gradient colors
- `cloud`: RGB values for cloud color
- `sun`: RGB values for sun/glow color
- `name`: Theme display name

#### Tweaking Game Balance
Key parameters scattered throughout:
- `KILLS_PER_STAGE` (line 491): Enemies needed to trigger boss spawn
- Player movement speed and bullet speed in `update()` function
- Boss attack patterns in `updateBoss()` function
- Weapon upgrade progression logic

### Code Style
- Uses `const` and `let` (no `var`)
- Consistent 2-space indentation
- Comments are minimal; code is self-documenting
- DOM elements cached at top for performance
- Game logic separated from rendering

### Performance Considerations
- Frame time tracking enables adaptive quality reduction
- Particle and projectile arrays reuse objects where possible
- Canvas 2D API used directly (no rendering library)
- No external dependencies to load

## Deployment

The game is ready to deploy as-is:
- Single HTML file with no build artifacts
- Works on desktop and mobile browsers
- Responsive design using CSS viewport units
- Uses localStorage (not IndexedDB or server APIs)

Simply copy `index.html` to a web server or CDN.
