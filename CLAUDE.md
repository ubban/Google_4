# Sky Fighter - Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game built with vanilla HTML5 Canvas and JavaScript. Players control a fighter jet, switch weapons, drop smart bombs, and battle stage bosses. The game features responsive touch and keyboard controls, dynamic audio with synthesis, and progressive difficulty scaling.

**Repository**: ubban/Google_4  
**Language**: JavaScript (vanilla, no frameworks)  
**Entry Point**: `index.html` (1669 lines, self-contained)  
**Browser Target**: Modern browsers with Canvas 2D, Web Audio API, and touch support

---

## Architecture & Key Components

### Game Loop & State Management

- **Main game loop** (IIFE starting at line 423): Runs at 60 FPS via `requestAnimationFrame`
- **Global state variables**: 
  - Player position, health, weapon, bomb count, score, combo counter, stage
  - Enemy list, bullet list, projectile lists (player, boss, power-ups)
  - Canvas, audio context, control states (keyboard, joystick, touch)
  - Game flags: paused, game over, stage clear, muted, game running

### Core Game Objects

**Player (Plane)**
- Position and health (3 max)
- Weapons: Spread Gun, Burst, Laser, Heavy Gun (cycle via spacebar)
- Bombs: smart bombs (3 max), spawned via Ctrl, drop on all enemies
- Movement: keyboard (WASD/arrows) or joystick/touch controls
- Rendered via `drawPlane(x, y, w, h, color, darkColor, flame)` with flame animation

**Enemies**
- Wave-based spawning: `spawnEnemy()`
- Variety: Fast, normal, heavy, armored types with health/speed/drop rates
- Bullet patterns: single fire, spread, arcs, or no fire
- Collision detection: with player bullets, bombs, and player

**Bosses** (`spawnBoss()`)
- One per stage, spawns at stage progression thresholds
- Custom patterns: spinning projectiles, targeted shots, bullet sprays
- Boss-specific rendering: `drawBossShip(b)` 

**Power-ups**
- Spawned from defeated enemies via `maybeSpawnPowerUp(x, y)`
- Types: health, ammo (bomb)
- Drawn as rotatinggoldstars

**Projectiles**
- Player bullets: fast, vary by weapon (spread, burst, laser lines, heavy shells)
- Enemy bullets: slower, various patterns
- Boss projectiles: high-velocity, some homing
- Collision via `rectsOverlap(a, b)`

### Audio System

- **Web Audio API synthesis**: No external audio files
- **Beep function** (`beep(freq, dur, type, vol, opts)`): Sine, square, sawtooth waves
- **Effects**:
  - Weapon fire (frequency varies by type)
  - Boss hit, game over, combo pop, stage clear, power-up pickup
  - Noise bursts for explosions
- **Panning**: `panFor(x)` pans left/right based on canvas position
- **Mute toggle**: UI button at top-right, toggles with M key

### Rendering & Graphics

- **Canvas 2D context**: Fixed to game dimensions (1280×720 on desktop, responsive on mobile)
- **Color scheme**: Dark blue gradient sky, white/cyan UI, orange/red explosions
- **Key draw functions**:
  - `drawPlane()`: Player jet with flame effect
  - `drawBossShip()`: Boss with rotating turrets and detailed sprite
  - Enemies: Simple rectangles with color coding (speed, health, type)
  - Bullets: Small circles
  - HUD: Hearts, bombs, score, combo, stage banner
- **Effects**: Screen shake (`screenShake(mag, dur)`), flash hits, combo popups

### Input Handling

**Keyboard**
- WASD / Arrow keys: Movement
- Spacebar: Switch weapon
- Ctrl: Drop bomb
- M: Toggle mute
- P: Pause (desktop only)
- R: Restart game

**Touch & Joystick** (for mobile)
- Virtual joystick: `joystickStart()`, `joystickMove()`, `joystickEnd()`
- Weapon switch/bomb buttons on mobile UI
- Responsive radius, visual feedback

**Resize Handling**
- `resize()` adjusts canvas and viewport on window resize
- Maintains aspect ratio, scales UI elements with viewport

---

## Game Flow & Difficulty

### Stages & Progression
- 5 stages with escalating enemy waves and boss encounters
- Boss triggers at specific kill thresholds per stage
- Stage clear on boss defeat; next stage automatically starts

### Difficulty Levels
- Easy, Normal, Hard difficulty presets
- Difficulty affects:
  - Enemy spawn rate
  - Projectile speed & density
  - Enemy health/behavior
  - Scoring multiplier
- Selected via UI or by the `setDifficulty(name)` function

### Scoring & Combo System
- Kill streaks increase score multiplier (combo meter)
- Hits reset combo to 1× 
- Visual feedback: `showComboPop(text)` for combo milestones
- Power-ups influence score bonus

---

## Key Functions Reference

### Game State & Control
- `resetGame()`: Clears all entities, resets player, initializes stage
- `setDifficulty(name)`: Sets game difficulty preset
- `updateMotionBtn()`: Shows/hides motion control on mobile
- `ensureAudio()`: Initializes Web Audio context on first user input
- `updateMuteBtn()`: Updates mute icon state

### Rendering & Effects
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Renders player jet
- `drawBossShip(b)`: Renders boss with turrets
- `screenShake(mag, dur)`: Camera shake on impacts
- `flashHit()`, `flashBomb()`: Brief screen flash for events
- `showComboPop(text)`: Displays combo notification
- `showStageBanner(text)`: Shows "Stage X" splash
- `renderHearts(count, max)`, `renderBombs(count, max)`: HUD elements

### Spawning
- `spawnEnemy()`: Creates wave enemies based on stage/difficulty
- `spawnBoss()`: Spawns boss for current stage
- `maybeSpawnPowerUp(x, y)`: Random power-up drop on enemy kill

### Collision & Physics
- `rectsOverlap(a, b)`: AABB collision test
- `canvasPointFromEvent(e)`: Maps input coordinates to canvas
- `joystickStart/Move/End()`: Virtual joystick input

### Audio
- `beep(freq, dur, type, vol, opts)`: Synthesize tones
- `noiseBurst(dur, vol, opts)`: Create noise effects
- `panFor(x)`: Pan audio left/right by position

### Utilities
- `weaponLabel()`: Returns current weapon name
- `announce(text)`: Logs to browser console for debug
- `trackPerf(dt)`: Measures frame time performance

---

## Data Structures

### Enemy Object
```javascript
{
  x, y,           // position
  w, h,           // dimensions
  type,           // 'fast', 'normal', 'heavy', 'armored'
  health,         // current HP
  health_max,     // max HP
  color,          // display color
  speed,          // movement speed
  fire_rate,      // shots per frame
  fire_timer,     // countdown to next shot
  pattern,        // bullet pattern function
  drop_rate       // power-up spawn chance
}
```

### Boss Object
```javascript
{
  x, y,           // position
  w, h,           // dimensions
  health,         // current HP
  health_max,     // max HP
  color,          // display color
  turrets,        // array of turret positions
  phase,          // behavior phase (0, 1, 2...)
  timer,          // phase timer
  pattern,        // custom bullet pattern
  dialog          // optional dialog text
}
```

### Bullet Object
```javascript
{
  x, y,           // position
  vx, vy,         // velocity
  r,              // radius
  color,          // display color
  owner           // 'player', 'enemy', 'boss'
}
```

---

## Browser Compatibility & Features Required

- **Canvas 2D**: Drawing and rendering
- **Web Audio API**: Sound synthesis
- **Touch Events**: Mobile input
- **Pointer Events**: Cross-platform input
- **requestAnimationFrame**: 60 FPS game loop
- **LocalStorage** (optional): Could store settings/high scores (not yet implemented)

---

## Performance Notes

- **60 FPS target**: Measured via `trackPerf(dt)`
- **Collision checks**: O(n²) worst-case between entities; acceptable for game scale
- **Canvas resize**: Triggers on `orientationchange` and `resize` events
- **Audio**: Synthesized on-the-fly; no decompression overhead

---

## Known Limitations & Future Improvements

1. **No persistent storage**: Scores/settings reset on page reload
2. **Single difficulty persistence**: Could save player's preferred difficulty
3. **Limited boss patterns**: Could add more varied boss behaviors per stage
4. **No combo multiplier visualization**: Could show 2×, 3×, etc. on screen
5. **Mobile touch responsiveness**: Could optimize joystick dead zone
6. **Audio volume normalization**: Synthesis can sound unpredictable across devices

---

## Running & Development

1. Open `index.html` in a modern browser
2. Use keyboard (WASD/arrows + spacebar + Ctrl) or touch controls
3. Toggle mute with M or UI button
4. Adjust difficulty before starting
5. No build step required; code is self-contained

**Testing Tips**:
- Open browser DevTools (F12) for console logs and performance metrics
- Test on mobile device for touch responsiveness
- Check canvas scaling on various screen sizes

---

**Last Updated**: October 3, 2026  
**Maintained By**: Claude Code
