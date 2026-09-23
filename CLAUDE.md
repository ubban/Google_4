# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5, CSS3, and JavaScript. It's a single-file implementation (~1669 lines) with no external dependencies. The game features multiple difficulty levels, procedurally-varied boss encounters, dynamic audio synthesis, adaptive performance, and full touch/mouse control support.

## Architecture Overview

The entire game is encapsulated in an **IIFE (Immediately Invoked Function Expression)** starting at line 423, creating a private scope for all game logic. This structure:
- Prevents global namespace pollution
- Maintains state isolation for multiple game instances (if needed)
- Groups related functionality naturally by system

### Major Systems

**1. Configuration & Constants (lines 463-523)**
- `STORAGE_*`: LocalStorage keys for persistence (best score, mute state, difficulty, motion preferences)
- `DIFFICULTIES`: Three difficulty tiers (easy/normal/hard) controlling spawn rates, enemy behavior, lives, bombs, and boss HP scaling
- `KILLS_PER_STAGE`: Enemies defeated per stage = 18
- `THEMES`: Six visual themes with gradient colors, affecting sky appearance and lighting tone
- `ENEMY_TYPES`: Three enemy variants (grunt, interceptor, tank) with speed, HP, color, score, and minimum stage
- `BOSSES`: Six boss types with unique color schemes and HP multipliers

**2. Audio System (lines 542-678)**
- Web Audio API synthesis (no MP3/external files)
- `ensureAudio()`: Lazy-initializes audio context with compressor for consistent levels
- `beep()`: Generates tones with frequency sweeps, optional filter, and spatial panning
- `noiseBurst()`: Creates noise effects (explosions)
- Sound effects are triggered inline as needed; all synthesis happens on-demand
- Audio is muted if `muted` flag is true (respects user preference)

**3. UI/Visual Feedback (lines 680-751)**
- `renderHearts()` / `renderBombs()`: HUD element rendering (DOM manipulation for icons)
- `flashHit()` / `flashBomb()`: Screen flash effects on damage/bomb use (CSS animation triggers)
- `showComboPop()` / `showStageBanner()`: Animated overlay text for scoring/progression milestones
- `screenShake()`: Applies camera jitter via canvas transform, duration-based
- `updateMuteBtn()` / `updateMotionBtn()`: Toggles and persists audio/motion preferences
- `setDifficulty()`: Updates difficulty setting and announces change to screen readers

**4. Input Handling (lines 767-862)**
- Touch joystick emulation with separate zones for movement and fire/bomb buttons
- Keyboard support (arrows for movement, space/Z for fire, X/C for bomb)
- `joystickStart/Move/End()`: Virtual joystick tracking with position smoothing
- `canvasPointFromEvent()`: Converts screen/touch coordinates to game canvas space (handles DPI scaling)
- Multi-touch support prevents accidental button presses during joystick movement

**5. Game State & Update Loop (lines 863-1507)**
- `resetGame()`: Initializes player, enemies, bullets, powerups, and score (NOT called on new game, use separately for restart)
- `update(dt)`: Main game loop (called every frame with delta time)
  - Updates player position/weapon/bombs
  - Advances all entities (enemies, bullets, explosions, particles)
  - Handles collision detection (player bullets ↔ enemies, enemies ↔ player)
  - Manages powerup spawning and pickup
  - Tracks enemy kill count for stage progression and boss spawning
- `applyPowerUp()`: Handles weapon upgrade acquisition logic

**6. Rendering Pipeline (lines 1507-1639)**
- `drawClouds()`: Parallax scrolling background clouds with depth separation
- `drawPlane()`: Renders player ship with animated engine flame
- `drawBossShip()`: Boss rendering with animated core pulse and health state visualization
- `draw()`: Main render function (called every frame)
  - Clears canvas with sky gradient
  - Renders clouds (both front/back layers)
  - Draws all game entities (enemies, boss, bullets, explosions, powerups, player)
  - Updates HUD (score, multiplier, health, bombs, boss health bar)
  - Handles game-over overlay rendering

**7. Game Loop & Timing (lines 1640-1669)**
- `loop(now)`: requestAnimationFrame callback
  - Calculates delta time clamped to 1/30 (prevents spiral-of-death on frame drops)
  - Tracks frame timing for adaptive quality adjustment
  - Calls update() then draw()
  - Scales canvas and applies camera shake transforms

## Key Data Structures

**Player Object**
```
{
  x, y,                    // Position (center)
  w: 32, h: 40,           // Dimensions
  speed: 0,               // Current movement speed (0-1)
  weaponLevel: 0,         // Upgrade tier (0-5, max)
  bombs: [count],         // Bomb remaining count
  lives: [count],         // Health remaining
  shootTimer: [ms],       // Fire rate limiter
  targetX: [x],           // Lerp target for smooth movement
}
```

**Enemy Object**
```
{
  type: 'grunt'|'interceptor'|'tank',  // Enemy variant
  x, y,                   // Position
  w, h,                   // Dimensions (from ENEMY_TYPES)
  hp: [count],           // Health remaining
  shootTimer: [ms],      // Cooldown before next shot
  angle: [radians],      // Facing direction (for weaving interceptors)
}
```

**Bullet Object**
```
{
  x, y,                   // Position
  vx, vy,                 // Velocity components
  r: [radius],           // Collision radius
  type: 'player'|'enemy', // Source
  homing: [target],      // Target reference (if homing bullet)
}
```

**Boss Object**
```
{
  x, y, w, h,            // Position and dimensions
  hp, maxHp,             // Health state
  shootTimer,            // Cooldown
  phase: 0-2,            // Behavior phase (transitions at HP milestones)
  shakeAmount: [mag],    // Animation state
  coreGlowPhase: [0-1],  // Glow pulsing animation
}
```

## Common Modifications

**Adjusting Difficulty Balance**
- Modify `DIFFICULTIES` object (line 485): `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- `spawnBase`: Base frames between spawns (lower = more enemies)
- `spawnMin`: Minimum frames between spawns (floor under difficulty scaling)

**Adding New Enemy Types**
1. Add entry to `ENEMY_TYPES` (line 502) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` (line 1022) to include new type in spawn pool
3. Add render code in `draw()` (line 1520) for new enemy visual
4. Update `EXPLODE_SFX` (line 678) if custom explosion sound needed

**Changing Boss Behavior**
- Boss AI patterns are hardcoded in `updateBoss()` (line 1411)
- Each phase has its own firing pattern (`fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireColumns`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing`)
- Boss spawn/progression logic is in `spawnBoss()` (line 1043) and `killBoss()` (line 1441)

**Modifying Game Visuals**
- **Sky/Theme**: Add new theme to `THEMES` (line 493) with gradient colors and cloud tint
- **Player Ship**: Edit `drawPlane()` (line 913) for new hull shape or flame animation
- **Boss Ship**: Edit `drawBossShip()` (line 970) for different color scheme or geometry
- **Enemies**: Modify color/shape in `draw()` (line 1520) enemy rendering section

**Weapon Upgrade Levels**
- `fireWeapon()` (line 1124) handles player firing, dispatched based on `weaponLevel`
- Fire patterns (lines 1324-1367): each level maps to a unique firing function (`fireRadial`, `fireAimedSpread`, etc.)
- Add new pattern as a new function, then add case in `fireWeapon()` to call it at new level

**Audio Adjustments**
- Frequency values in `beep()` calls: standard audio frequencies (440=A4, lower=deeper)
- Duration in milliseconds: `beep(freq, durMs, 'sine'|'square'|'triangle', volume, opts)`
- Modify master gain (line 556) to change overall volume

## Development Workflow

**To Test Locally**
1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. No build step required; all changes take effect on refresh

**To Debug**
- Open DevTools Console (F12)
- Most state is in closure scope; use `loop()` for frame stepping
- Canvas rendering happens in `draw()` (line 1520); add console logs before/after draw calls
- Player input logged via `canvasPointFromEvent()` (line 840)

**To Profile Performance**
- The adaptive quality system (lines 473-483) tracks frame times and downgrades rendering automatically
- Current quality level: `quality` variable (0=full, 1=medium, 2=low) in `trackPerf()`
- Check DevTools Perf tab for jank; look for long tasks in `update()` or `draw()`

**To Add Persistence**
- Use `localStorage` with keys prefixed `skyfighter.`
- Save in-game: use LocalStorage API directly or add to `endGame()` (line 1469)
- Load on startup: add retrieval logic before game loop starts (around line 1650)

## Browser APIs & Compatibility

- **Canvas 2D**: `ctx.drawImage()`, `ctx.fillRect()`, `ctx.arc()` for rendering
- **Web Audio API**: `AudioContext`, `OscillatorNode`, `GainNode`, `DynamicsCompressor` for sound synthesis
- **LocalStorage**: `localStorage.getItem/setItem()` for persistence
- **Touch Events**: `touchstart`, `touchmove`, `touchend` for mobile input
- **DevicePixelRatio**: `window.devicePixelRatio` for high-DPI canvas scaling
- **Media Queries**: `prefers-reduced-motion` for accessibility preference detection
- **requestAnimationFrame**: Main game loop timing

All APIs are widely supported in modern browsers (IE11+ with polyfills for some).

## Debugging Tips

- **Game not starting**: Check browser console for errors. Ensure `#gameCanvas` exists in HTML.
- **Sound not playing**: Verify AudioContext initialization in `ensureAudio()`. Browser may require user gesture first (check mute state in `togglePause()` for interaction gate).
- **Input not responding**: Check `canvasPointFromEvent()` coordinate transformation. Verify touch/mouse events reach canvas. Test with `console.log()` in event handlers.
- **Performance issues**: Run `trackPerf()` diagnostic. Check if `quality` variable drops below 0 (max downgrade reached). Profile in DevTools Performance tab.
- **Visual glitches**: Verify canvas resize in `resize()` (line 754). Check sky gradient colors in current theme. Ensure DPI scaling doesn't cause clipping.
