# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 Canvas application. Players control a fighter plane, dodge enemy fire, switch between weapons, deploy smart bombs, and defeat stage bosses. The game features three difficulty levels, a score multiplier chain system, weapon upgrades, touch controls with a virtual joystick, and procedural audio generation using the Web Audio API.

## Quick Start

### Running the Game
1. Open `index.html` directly in a modern web browser (Chrome, Firefox, Safari, Edge)
2. Alternatively, serve with a local HTTP server to avoid CORS issues:
   ```bash
   # Python 3
   python3 -m http.server 8000
   # Then visit http://localhost:8000
   ```

### Testing Changes
After modifying the code:
1. Save `index.html`
2. Refresh the browser (Ctrl+R or Cmd+R)
3. The game will immediately reflect changes

## Codebase Architecture

### Single-File Structure
The entire game exists in `index.html` (~1669 lines) with these major sections:

#### 1. **HTML Head (lines 1-380)**
- Meta tags and responsive viewport configuration
- CSS styling for the game canvas, HUD, UI overlays, and touch controls
- Accessibility features (screen reader support with `#srLive` element)
- SVG favicon definition

#### 2. **HTML Body (lines 380-420)**
- `#gameCanvas`: The main 2D canvas for game rendering
- `#container`: Full-screen wrapper
- `#hud`: Heads-up display (score, multiplier, weapon level, hearts, bombs)
- `#overlay`: Start screen and game over screen
- `#touchControls`: Virtual joystick and action buttons for mobile
- Visual effects containers: `#flash` (hit flash), `#comboPop`, `#vignetteEl`, `#stageBanner`

#### 3. **Game Script (lines 423-1666)**

**Constants & Configuration (lines 423-825):**
- `DIFFICULTIES`: Three difficulty presets (easy, normal, hard) with spawn rates, lives, bombs
- `ENEMY_TYPES`: Enemy definitions (grunt, interceptor, tank) with health, speed, damage
- `BOSSES`: Boss ship definitions for each stage
- `THEMES`: Stage background color themes
- `KILLS_PER_STAGE`: Enemy quota per stage (18 kills)
- Audio context initialization and compression setup

**Input Handling (lines 826-862):**
- Keyboard: WASD/Arrow keys for movement, Space/E for firing, Q for bombs
- Mouse: Click to set firing target
- Touch: Virtual joystick in `#moveZone`, fire and bomb buttons
- Global state: `firing`, `mouseActive`, `mouseTarget`, `keys` object

**Game State (lines 863-910):**
Main game variables initialized in `resetGame()`:
- `player`: Position (x, y), dimensions, speed, weapon, level, invulnerability timer
- `bullets`: Player projectiles array
- `enemies`: Enemy ships array
- `enemyBullets`: Enemy projectiles array
- `particles`: Visual effects (explosions, impacts)
- `powerUps`: Weapon and health drops
- `boss`: Current boss object or null
- Scoring system: `score`, `chain` (combo), `multiplier`, `kills`, `stage`
- UI state: `lives`, `bombCount`, `paused`, `running`, `gameOver`

**Core Game Systems:**

1. **Rendering (lines 913-1639):**
   - `draw()`: Main render function called every frame
   - `drawPlane()`: Player and enemy ship renderer with gradients and dynamic flames
   - `drawBossShip()`: Boss-specific rendering with boss bar UI
   - `drawClouds()`: Scrolling background parallax effect
   - Particle rendering with fade-out alpha blending

2. **Game Loop (lines 1640-1653):**
   - `loop(now)`: RequestAnimationFrame callback
   - Delta time calculation with 50ms cap for stability
   - Conditional update/draw when `running && !paused`
   - Performance tracking via `trackPerf(dt)`
   - Screen shake effect countdown

3. **Update Logic (lines 1000+):**
   - `update(dt)`: Main game update function
   - Player movement with boundary clamping
   - Weapon firing cooldown and rapid-fire cooldown
   - Enemy spawning based on stage difficulty
   - Collision detection between player/bullets and enemies/bullets
   - Enemy and bullet movement
   - Power-up collection
   - Boss progression and stage advancement
   - Combo chain timeout

4. **Audio System (lines 545-677):**
   - Web Audio API context with dynamics compressor
   - `beep(freq, dur, type, vol, opts)`: Oscillator-based sounds with biquad filter and panning
   - `noiseBurst(dur, vol, opts)`: White noise for explosions
   - `sfx` object: Pre-configured sound effect definitions (fire, hit, powerUp, explosion variants by enemy type)
   - `panFor(x)`: Stereo panning based on screen position
   - Audio cue mapping via `EXPLODE_SFX` object

5. **UI & Persistence (lines 1655+):**
   - LocalStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
   - HUD updates: `scoreEl`, `multEl`, `weaponNameEl`, `weaponLvlEl`, `heartsEl`, `bombsEl`
   - Boss bar visibility and fill animation
   - Overlay transitions for start/end screens
   - Screen reader announcements via `announce()`

### Key Game Objects

**Player Object:**
```javascript
{
  x, y,           // Canvas coordinates
  w: 40, h: 40,   // Dimensions
  speed: 260,     // Pixels per second
  cooldown: 0,    // Weapon fire timer
  rapid: 0,       // Rapid-fire bonus timer
  weapon: 'vulcan' | 'laser',  // Current weapon
  level: 1-3,     // Weapon upgrade level
  invuln: 0       // Invulnerability timer (post-hit)
}
```

**Enemy Object:**
```javascript
{
  x, y,              // Position
  type: string,      // 'grunt' | 'interceptor' | 'tank'
  health, maxHealth, // Hit points
  speed,             // Movement speed
  theta: 0,          // Angle to player (computed each frame)
  angle,             // Visual rotation
  shootCooldown: 0   // Firing timer
}
```

**Bullet Object:**
```javascript
{
  x, y,              // Position
  vx, vy,            // Velocity
  damage,            // Damage per hit
  maxRange,          // Distance before despawn
  fired: 0,          // Distance traveled
  isPlayerBullet     // Owner (boolean)
}
```

**Boss Object:**
Similar to enemy, with additional properties:
- `shield`: Remaining health
- `attacks`: Attack pattern array
- `attackTimer`: Current attack phase timer
- Boss phases trigger after specific kill thresholds per stage

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant with properties: `health`, `speed`, `damage`, `points`, `fireRate`, `fireSpeed`
2. Add explosion sound mapping to `EXPLODE_SFX` if needed
3. Update spawning logic in `update()` to include new type
4. Enemy rendering is handled generically by `drawPlane()`; customize in `drawPlane()` if needed
5. Collision and movement logic is type-agnostic; test via browser

### Adjusting Game Balance
- Difficulty: Modify `DIFFICULTIES` object (spawn interval, enemy health scaling, player lives, bombs)
- Enemy spawning: Change `spawnTimer` increment in `update()` or adjust `spawnChance` calculation
- Player health: Modify `lives` or `maxLives` in `resetGame()`
- Weapon balance: Adjust `player.cooldown` rates or `bullet` damage values in firing code

### Adding Visual Polish
- Screen shake: Call `screenShake(magnitude, duration)` from collision/explosion events
- Particle effects: Push new particle objects to `particles[]` array with `x, y, vx, vy, life, color`
- Color themes: Add new entry to `THEMES` array and reference by stage index
- Flash effects: `flashHit()` for player hit, `flashBomb()` for bomb detonation

### Audio Tweaking
- Modify frequencies in `sfx` definitions (e.g., fire frequency for pitch)
- Adjust volume (`vol` parameter, range 0-1)
- Change `type` for oscillator waveform: `'sine'` (smooth), `'square'` (harsh), `'sawtooth'` (bright)
- Duration values are in seconds (0.1 = 100ms)

### Browser Testing Checklist
- Desktop: Chrome, Firefox, Safari (test mouse + keyboard)
- Mobile: Touch controls, joystick response, screen orientation
- Accessibility: Mute button persists state, reduced-motion preference respected, screen reader announcements clear
- Performance: Game runs smoothly at 60 FPS (check console for frame time tracking)

## Performance Considerations

- **Canvas size**: Auto-scales to window with `canvas.width/height` set on resize
- **Delta time capping**: 50ms max delta prevents physics instability on frame drops
- **Particle pooling**: Particles are not recycled; high spawn rates may impact performance
- **Collision detection**: O(n²) with enemy bullets; optimize if enemy bullet count exceeds ~100
- **Audio**: Web Audio context shared; ensure audio is muted on tab blur to avoid CPU overhead

## Debugging Notes

- Performance data available via `frameTimes` array (updated each frame)
- Screen reader live region (`#srLive`) shows game events for accessibility testing
- LocalStorage values can be inspected in DevTools > Application > Local Storage
- Game state (player, enemies, boss, etc.) is available in global scope within IIFE for console inspection

## Future Enhancement Ideas

- Boss attack pattern variations based on difficulty
- Progressive difficulty scaling within a single game session
- Weapon-specific special abilities (laser beam size, vulcan spread)
- Enemy formation patterns or coordinated attacks
- Leaderboard storage with player names
- Tutorial or on-screen control hints for new players
