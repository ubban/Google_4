# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file (`index.html`). The game is fully self-contained with embedded CSS, JavaScript, and Canvas-based rendering. It requires no build step, dependencies, or server—just open the HTML file in a modern browser to play.

### Key Features
- Classic vertical scrolling shooter gameplay with player aircraft
- Progressive difficulty with multiple stages and boss encounters
- Weapon system with upgradeable weapon types and levels
- Smart bomb (special weapon) mechanic with limited charges
- Score multiplier system based on consecutive enemy hits
- Multiple difficulty levels (Easy, Normal, Hard)
- Touch controls with virtual joystick for mobile/tablet gameplay
- Keyboard controls for desktop (arrow keys + spacebar)
- Persistent score tracking via localStorage
- Reduced motion support for accessibility
- Web Audio API sound effects and background music
- Screen reader support with ARIA announcements

## How to Run and Develop

### Running the Game
Simply open `index.html` in a modern web browser. No build process, bundler, or server required.

```bash
# Option 1: Open directly in browser
open index.html  # macOS
start index.html # Windows
xdg-open index.html # Linux

# Option 2: Use a simple HTTP server
python -m http.server 8000  # Python 3
npx http-server              # Node.js
```

### Development Workflow
1. Edit `index.html` directly—all code is in this single file
2. Test changes by refreshing the browser (Ctrl+R or Cmd+R)
3. Use browser DevTools (F12) to debug:
   - **Console**: Check for errors and use `console.log()` debugging
   - **Performance**: Use the `trackPerf()` function to monitor frame timing
   - **Canvas**: Inspect rendered elements with the Inspector
4. Test on mobile via browser DevTools device emulation or physical device
5. Commit changes with clear messages describing game mechanics or features added

### Testing Notes
- Test across different browser sizes (use DevTools responsive mode)
- Test on mobile devices—the game includes touch control code and should work on phones/tablets
- Test all difficulty levels and game states (main menu, active gameplay, pause, game over)
- Test audio-related features: mute button, sound effects at different game events

## Code Architecture and Structure

The entire game logic is contained in a single IIFE (Immediately Invoked Function Expression) starting at line 423. The structure follows this pattern:

### 1. **DOM Elements & Constants** (lines 426–466)
   - Canvas and UI element references
   - LocalStorage keys for persisting settings (best score, mute state, difficulty, reduced motion preference)

### 2. **Utility Functions** (lines 468–841)
   - **Audio**: `ensureAudio()`, `beep()`, `noiseBurst()`, `panFor()` - Web Audio API wrapper
   - **UI Updates**: `announce()`, `updateMotionBtn()`, `updateMuteBtn()`, `renderHearts()`, `renderBombs()`
   - **Visual Effects**: `flashHit()`, `flashBomb()`, `showComboPop()`, `showStageBanner()`, `screenShake()`
   - **Input Handling**: `joystickStart()`, `joystickMove()`, `joystickEnd()`, `canvasPointFromEvent()`
   - **Settings**: `setDifficulty()`, `resize()`, `togglePause()`

### 3. **Game State Variables** (throughout, initialized in `resetGame()`)
   - Player ship position, health, bombs, active weapon
   - Enemies array and boss object
   - Particles and explosions
   - Score, multiplier, stage counter
   - Game flags (paused, ended, boss stage, etc.)

### 4. **Core Game Systems**

#### **Player System**
- Drawn via `drawPlane()` function (lines 913–969)
- Controlled via keyboard (arrow keys) or touch joystick
- Has health (lives), bombs, and active weapon
- Firing handled in `fireWeapon()` (lines 1124–1147)

#### **Enemy System**
- Spawned via `spawnEnemy()` (lines 1022–1042) at intervals based on difficulty
- Move downward; destroyed on collision with player bullets
- Drop power-ups when killed via `maybeSpawnPowerUp()` (lines 1058–1068)
- Collision detection with player and bullets via `rectsOverlap()` (lines 1069–1072)

#### **Boss System**
- Spawned via `spawnBoss()` (lines 1043–1057) after defeating enemy waves
- Drawn via `drawBossShip()` (lines 970–1021)
- Updated in `updateBoss()` (lines 1411–1440)
- Uses various attack patterns: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireDoubleRing()`, `fireHomingSeeded()`
- Health bar rendered at top of screen
- Killing boss triggers wave progression and stage transition via `killBoss()` (lines 1441–1455)

#### **Weapon System**
- Five weapon types: VULCAN, LASER, SPREAD, HOMING, BOMB
- Each weapon has a level (1–5) affecting damage and spread
- Weapon upgrade pickups increase weapon level
- Current weapon and level displayed in HUD
- Weapon label accessible via `weaponLabel()` (lines 871–874)

#### **Scoring System**
- Base score for destroying enemies: 100 points
- Multiplier increases with consecutive hits; resets on missed (player hit)
- Multiplier displayed in HUD and increased in `addScore()` (lines 1087–1095)
- Combo resets via `breakChain()` (lines 1096–1102)
- Best score persisted to localStorage

#### **Power-up System**
- Dropped randomly from destroyed enemies
- Types: HEALTH (red), BOMB (blue), WEAPON (green), MULT (gold)
- Applied in `applyPowerUp()` (lines 1304–1323)
- Animate downward and disappear if not collected

### 5. **Main Game Loop** (lines 1640–1663)
- `loop(now)` called via `requestAnimationFrame` for 60 FPS gameplay
- Calls `update(dt)` for game logic (lines 1148–1303)
- Calls `draw()` for rendering (lines 1520–1639)
- Manages delta time and performance tracking

## Key Customization Points for Future Development

### Difficulty Tuning
- **Enemy spawn rates**: Adjust in `spawnEnemy()` based on `difficulty` variable
- **Boss patterns**: Modify boss attack functions (`fireRadial()`, `fireAimedSpread()`, etc.)
- **Player health/bombs**: Modify `resetGame()` initial values
- **Stage progression**: Adjust enemy wave counts and boss trigger points in `update()`

### Weapon Balance
- **Weapon damage**: Modify bullet radius in `fireWeapon()` for each weapon type
- **Fire rate**: Adjust `playerCooldown` and weapon-specific delays
- **Weapon spread patterns**: Modify bullet count and angle in weapon firing functions

### Visual & Audio
- **Colors**: Canvas drawing uses hardcoded RGB values—search for `#` or `rgb()` for color constants
- **Sound effects**: Modify `beep()` frequency/duration and `noiseBurst()` parameters
- **Animations**: Screen shake duration/magnitude in `screenShake()`, flash effects in `flashHit()` and `flashBomb()`
- **Background**: Gradient defined in CSS at line 27

### Mobile/Touch
- **Touch joystick size**: Adjust `JOYSTICK_RADIUS` (current value embedded in code around line 785)
- **Button positions**: CSS classes `#fireBtn` and `#bombBtn` positioned absolutely at bottom of screen
- **Responsive scaling**: `resize()` function handles canvas scaling to window size

## Performance Considerations

### Frame Timing
- Game targets 60 FPS but adapts to lower framerates
- `trackPerf(dt)` monitors frame timing; logs warnings if frame drops below threshold
- Delta time (`dt`) passed through update loop to scale movement frame-independently

### Particle Effects
- Explosions created via `spawnExplosion()` with configurable particle count
- Particles stored in arrays and cleaned up when off-screen
- Consider limiting particle count on low-end devices if performance issues arise

### Canvas Rendering
- Single canvas context for all rendering
- Cloud background rendered at lines 1507–1519 (parallax scrolling effect)
- Enemy/boss/bullet drawing happens in `draw()` function in sequence
- No off-screen canvas optimization currently—full redraw each frame

## Browser Compatibility

- **Supported**: Chrome, Firefox, Safari, Edge (all modern versions)
- **Required features**:
  - Canvas 2D context
  - Web Audio API (for sound; gracefully degraded if unavailable)
  - LocalStorage (for persistent settings)
  - Touch events (for mobile)
- **Responsive**: Adapts to all screen sizes via CSS viewport meta tag and `resize()` function
- **Accessibility**: Screen reader support via ARIA live regions (`srLive` element)

## LocalStorage Keys

The game persists the following user data:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty level
- `skyfighter.reducedMotion`: Reduced motion preference

## Common Tasks

### Add a New Enemy Type
1. Add spawn logic in `spawnEnemy()`
2. Add drawing code in `draw()` loop where enemies are rendered
3. Add collision/update logic in `update()` enemy loop
4. Adjust `rectsOverlap()` bounding box if needed

### Add a New Boss Attack Pattern
1. Create a new function like `fireNewPattern(b, params)` following the existing pattern
2. Call it from `updateBoss()` at appropriate times
3. Test hit detection and visual balance

### Adjust Game Difficulty
1. Modify `setDifficulty()` to adjust difficulty variables (enemy spawning, health, etc.)
2. Test on all three difficulty levels
3. Ensure boss difficulty scales appropriately

### Add Sound Effects
1. Create a new function using `beep()` or `noiseBurst()`
2. Call it at game events (e.g., `fireWeapon()`, `applyPowerUp()`, `killBoss()`)
3. Use volume parameter to balance with other sounds
