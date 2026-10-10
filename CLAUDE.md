# Sky Fighter – Codebase Guide for Claude

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 application with embedded JavaScript and Canvas rendering. The game features dynamic difficulty scaling, weapon switching, smart bombs, stage progression, and boss encounters.

## Architecture & Structure

### Single-File Design
- **index.html** (1,669 lines): Complete game implementation
  - Embedded `<style>` section: CSS for HUD, animations, responsive layout
  - Embedded `<script>` section: All game logic, rendering, input handling, audio

### Canvas-Based Rendering
- **Canvas ID**: `#gameCanvas`
- **Resolution**: Dynamic, scales to fit device (max aspect ~16:9)
- **Refresh**: ~60 FPS (requestAnimationFrame loop)
- **Coordinate System**: (0,0) at top-left, positive X right, positive Y down

## Game Systems

### 1. **Player Character**
- Single player-controlled aircraft (variable position/rotation)
- Collision detection with enemies, bullets, powerups
- Health system: 3 hit points by default
- Weapon selection: cycles through current arsenal
- Smart bomb: area-effect screen clear (limited uses)
- Audio cues for gameplay events

### 2. **Enemy System**
- **Regular Enemies**: Spawn in waves, follow attack patterns, drop powerups
- **Boss Enemies**: Larger health pools, complex patterns, screen shake on defeat
- **Bullet Spawning**: Enemies fire in patterns toward player
- **Wave Management**: Difficulty increases with stage number

### 3. **Weapon System**
- **Types**: Standard, spread shot, laser, homing
- **Cycling**: Player can switch between owned weapons
- **Powerups**: Drop from defeated enemies to add new weapons or ammo
- **Ammo Tracking**: Some weapons track ammo, others unlimited

### 4. **UI/HUD System**
- **Stats Display**: Health hearts, bombs, weapon type, multiplier
- **Stage Banner**: Shows stage number and difficulty
- **Combo Counter**: Displayed for kill streaks
- **Responsive**: Adapts to screen size with `clamp()`

### 5. **Audio System**
- **Web Audio API** based with fallback graceful degradation
- **Functions**:
  - `beep(freq, dur, type, vol, opts)`: Synthesized tones
  - `noiseBurst(dur, vol, opts)`: Noise effects (explosions)
  - `panFor(x)`: Stereo panning based on screen position
- **Lazy Loading**: Audio context created on first interaction
- **Mute Toggle**: Button to silence audio

### 6. **Input Handling**
- **Keyboard**: WASD or Arrow Keys for movement, Space for shoot, X for bomb, M for mute, Q for fullscreen
- **Touch/Mobile**: Virtual joystick with thumb-stick emulation
- **Motion Sensors**: Optional tilt control (if supported)

### 7. **Difficulty & Progression**
- **Stages**: Game progresses through numbered stages (1, 2, 3, ...)
- **Difficulty Levels**: Easy, Normal, Hard, Nightmare (selectable)
- **Wave Scaling**: Enemy count and pattern complexity increases per stage
- **Boss Triggers**: Boss spawn at stage milestones (every 3–5 waves typically)

## Key Functions

### Core Game Loop
- **`main(dt)`**: Primary update loop, manages enemies, bullets, collisions
- **`draw()`**: Canvas rendering pass for all sprites
- **`requestAnimationFrame` callback**: Handles timing and FPS management

### Enemy & Boss Management
- **`spawnEnemy()`**: Creates regular enemy with random pattern
- **`spawnBoss()`**: Creates stage boss with unique behavior
- **`maybeSpawnPowerUp(x, y)`**: Random powerup drop on enemy death
- **`updateMotionBtn()`**: Checks device orientation sensor

### Rendering
- **`drawPlane(x, y, w, h, color, darkColor, flame)`**: Renders player ship
- **`drawBossShip(b)`**: Renders boss sprite with damage state coloring
- **`renderHearts(count, max)`**: HUD health indicator
- **`renderBombs(count, max)`**: HUD bomb counter
- **`flashHit()` / `flashBomb()`**: Screen flash animations

### Audio & Feedback
- **`beep(freq, dur, type, vol, opts)`**: Synthesized sound effect
- **`noiseBurst(dur, vol, opts)`**: Explosion/impact noise
- **`announce(text)`**: Screen text popup (stage, combo, etc.)

### Input
- **`joystickStart/Move/End()`**: Touch joystick tracking
- **`canvasPointFromEvent(e)`**: Converts browser coordinates to canvas space
- **`weaponLabel()`**: Returns current weapon name

### Collision & Physics
- Distance-based collision detection (circles/rectangles)
- Bullet-enemy collision spawns explosions and powerups
- Enemy-player collision triggers damage and knockback

## Important Data Structures

### Game State Object
```javascript
// Main state variables (global scope)
let gameState = {
  width: canvas.width,
  height: canvas.height,
  player: { x, y, w, h, vx, vy, health, maxHealth, bombs, currentWeapon, ammo... },
  enemies: [ {type, x, y, vx, vy, health, pattern, fireRate...} ],
  bullets: [ {type, x, y, vx, vy, owner, damage...} ],
  explosions: [ {x, y, radius, life} ],
  powerups: [ {type, x, y, vx, vy} ],
  wave: number,
  kills: number,
  score: number,
  difficulty: string,
  isPaused: boolean,
  ...
}
```

### Enemy Patterns
- **Pattern IDs**: Numbered attack patterns (e.g., "straight", "spread", "sine", "aimed")
- **Fire Rates**: Per-enemy configurable bullet frequency
- **Movement**: Velocity-based, with optional waypoints

### Weapons
- **Bullet Type**: Determines damage, speed, visual
- **Fire Pattern**: Single, spread, homing, etc.
- **Ammo**: Limited or unlimited per weapon type

## Styling & Responsive Design

### Key CSS Features
- **Dark Theme**: `color-scheme: dark`, dark blue backgrounds
- **Animations**: Heart beat pulse on critical health, flash effects
- **Responsive**: Uses `clamp()` for adaptive font sizes
- **Safe Area**: `env(safe-area-inset-*)` for notched devices
- **Vignette Effect**: Red screen damage indicator (canvas-based pseudo-effect)

## Development Notes

### Adding New Features

1. **New Enemy Type**:
   - Define pattern in `spawnEnemy()` or `spawnBoss()`
   - Add rendering logic in `draw()` loop
   - Update collision detection if needed

2. **New Weapon**:
   - Create bullet pattern in main loop
   - Add weapon label in `weaponLabel()`
   - Handle powerup drop logic in `maybeSpawnPowerUp()`

3. **New Stage/Boss**:
   - Adjust `spawnBoss()` spawn conditions
   - Add unique rendering in `drawBossShip()`
   - Tune health/patterns in boss object

4. **Audio Effects**:
   - Use `beep()` for tones, `noiseBurst()` for noise
   - Set panning with `panFor(x)` for stereo positioning

### Debugging Tips

- **Canvas Coordinates**: Use `canvasPointFromEvent()` to verify input mapping
- **Collision Bounds**: Add debug draw of hitboxes (circle/rect) if needed
- **Performance**: Check `trackPerf(dt)` for FPS monitoring
- **Audio Issues**: Ensure `ensureAudio()` called before first sound

### Known Constraints

- **Single-File Design**: All code in one HTML file limits modularity; refactoring into separate modules would improve maintainability
- **Global Scope**: Heavy use of global variables; consider encapsulation in future versions
- **Canvas Size**: Responsive sizing can cause artifacts on resize; cache canvas dimensions
- **Mobile Performance**: Complex wave patterns may drop frames on older devices

## Testing & Validation

- **Manual Testing**: Play through all stages, test weapon switching, bomb usage
- **Mobile**: Verify touch joystick and tilt controls on various devices
- **Audio**: Check mute button, audio panning, sound timing
- **Difficulty Levels**: Confirm scaling from Easy to Nightmare
- **Responsive**: Test on phone, tablet, desktop at various resolutions

## File Checklist

- ✅ **index.html**: Single source of truth
- ✅ **README.md**: Basic project title
- ✅ **CLAUDE.md**: This guide (added Oct 10, 2026)

---

**Last Updated**: October 10, 2026  
**Maintained By**: Claude Haiku 4.5
