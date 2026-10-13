# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 Canvas application. Players control an aircraft, destroy enemies across multiple stages with escalating difficulty, fight bosses, and accumulate a high score. The entire game logic and rendering is contained in `index.html` (~1670 lines).

## High-Level Architecture

The codebase is organized as a **single-file, functional game engine** with the following structure:

### Core Game Loop (lines 1640-1652)
```javascript
function loop(now) {
  dt = calculate delta time
  update(dt)        // Update game state
  draw()            // Render to canvas
  requestAnimationFrame(loop)
}
```

The loop runs at 60 FPS, calling `update()` for physics/logic and `draw()` for rendering.

### Main Game Systems

1. **Entity Management**
   - **Player**: Single aircraft with position, weapon, health, invulnerability frames, fire cooldown
   - **Enemies**: Array of enemies with types (grunt, interceptor, tank), each with position, HP, movement patterns
   - **Bullets**: Player bullets and enemy bullets, tracked separately; support piercing and laser effects
   - **Boss**: Single boss entity with special AI, attack patterns, and telegraph system
   - **Power-ups**: Weapon/bomb pickups from destroyed enemies
   - **Particles**: Visual effects (explosions, hit indicators)

2. **Collision System** (`rectsOverlap()` at line 1069)
   - Simple AABB (Axis-Aligned Bounding Box) collision detection
   - Checked each frame for: player-enemy, player-bullets, bullets-enemies, bullets-boss, player-powerups

3. **Weapon System**
   - **Vulcan**: Spread shot with multiple projectiles; damage type is pierce-resistant
   - **Laser**: Straight shot with piercing capability; can hit multiple enemies
   - Weapons and ammo upgraded by collecting power-ups
   - Fire rate: 0.15s (Vulcan) or 0.1s (Laser)

4. **Enemy System**
   - Enemy types defined in `ENEMY_TYPES` constant (lines 502-506)
   - Spawning driven by difficulty parameters and stage progression
   - Shooting patterns: straight or spread formation
   - Weaving movement for interceptors
   - Kill counter triggers boss spawn after 18 kills per stage

5. **Boss System**
   - Six boss designs with unique color schemes and attack patterns (lines 508-515)
   - Boss patterns are function arrays indexed by boss type (lines ~1350-1409)
   - Telegraph system (visual cue before attack patterns fire)
   - Scaling HP multiplier based on difficulty
   - Attack patterns include: aimed spreads, radial fire, homing bullets, columns, rings

6. **Audio System** (lines 542-595)
   - Uses Web Audio API with procedural synthesis (no external audio files)
   - `beep()`: Oscillator-based tones with envelope and filtering
   - `noiseBurst()`: Generated white noise for impact sounds
   - Stereo panning based on screen position
   - Master gain and compressor for dynamic control

7. **Visual Rendering** (`draw()` at lines 1520-1638)
   - Canvas-based 2D rendering
   - **Sky rendering**: Linear gradient per-stage theme
   - **Clouds**: Procedurally animated overlapping ellipses
   - **Entities**: Custom draw functions for player (`drawPlane()`) and boss (`drawBossShip()`)
   - **Effects**: Screen shake, flash overlay, particle system
   - **Reduced motion**: Disables shake/flash when accessibility preference set

### Game State Variables

Core state tracked globally:
- `running`, `paused`, `gameOver`: Game mode flags
- `score`, `multiplier`, `kills`, `stageKills`: Scoring system
- `lives`, `bombCount`: Player resources
- `stage`: Current stage (incremented on boss defeat)
- `player`: { x, y, w, h, speed, weapon, level, invuln, cooldown }
- `enemies[]`, `enemyBullets[]`, `bullets[]`: Entity arrays
- `boss`: Current boss (null when not active)
- `powerUps[]`, `particles[]`: Effects arrays
- `frame`: Frame counter for animation timing

### Difficulty System (lines 485-489)

Three difficulty levels with parameters:
- `spawnBase`, `spawnMin`: Enemy spawn rates
- `enemySpeed`: Difficulty multiplier
- `enemyShootChance`: Enemy fire probability
- `lives`, `bombs`: Starting resources
- `bossHpMul`: Boss health scaling

### Theme System (lines 493-500)

Six visual themes cycling through stages with color gradients, cloud colors, and sun colors.

## Development Notes

### File Organization
- All code is in `index.html`
- HTML structure: Canvas, overlay UI, touch controls
- CSS: Inline styles for game UI, animations, responsive layout
- JavaScript: IIFE (Immediately Invoked Function Expression) wrapping all game logic

### Key Functions by Category

**Rendering**: `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`, `renderHearts()`, `renderBombs()`

**Game Logic**: `update()`, `updateBoss()`, `spawnEnemy()`, `spawnBoss()`, `resetGame()`, `endGame()`

**Combat**: `fireWeapon()`, `useBomb()`, `updateBoss()` (boss attack patterns)

**Input**: `canvasPointFromEvent()`, `joystickStart/Move/End()` (touch controls)

**Audio**: `beep()`, `noiseBurst()`, `sfx` object with sound effect functions

**UI**: `setDifficulty()`, `updateMotionBtn()`, `updateMuteBtn()`, `togglePause()`

### Adaptive Performance (lines 474-483)
The engine tracks frame times and auto-scales quality between three levels:
- Level 0: Full quality
- Level 1: Medium quality (enabled if avg frame time > 25ms)
- Level 2: Low quality (enabled if avg frame time > 25ms for 3 frames)

This allows the game to degrade rendering quality under performance stress.

### Accessibility Features
- Keyboard and mouse input support
- Touch-based virtual joystick for mobile
- Reduced motion preference (prefers-reduced-motion media query)
- Screen reader announcements via `srLive` ARIA live region
- Mute option for audio

### Persistent Storage (localStorage)
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion preference

## Common Development Tasks

### Adjusting Game Balance
- Difficulty parameters: `DIFFICULTIES` object (line 485)
- Enemy definitions: `ENEMY_TYPES` object (line 502)
- Boss patterns: Function array at lines ~1350-1409
- Enemy spawn rate: `spawnTimer` logic in `update()` (lines 1198-1203)
- Kill threshold per stage: `KILLS_PER_STAGE` constant (line 491)

### Adding New Enemy Type
1. Add entry to `ENEMY_TYPES` object with stats
2. Add spawn logic in `spawnEnemy()` function
3. Add color/visual rendering in relevant draw functions
4. Add attack pattern if needed in enemy update loop

### Adding New Boss Pattern
1. Add boss design to `BOSSES` array if needed
2. Add pattern functions to `BOSS_PATTERNS` array
3. Pattern receives boss object `b` and should push bullets to `enemyBullets[]`
4. Use helper functions like `fireRadial()`, `fireColumns()`, `fireAimedSpread()`

### Adjusting Visual Effects
- Colors: Theme data in `THEMES` array
- Shake intensity: `screenShake()` function parameters
- Flash effects: `flashHit()`, `flashBomb()` functions
- Particle effects: `spawnExplosion()` function

## Testing and Running

### Running the Game
1. Open `index.html` in a web browser (no build step required)
2. Single-player, browser-based gameplay
3. No external dependencies or assets

### Manual Testing Checklist
- **Gameplay**: Player movement, shooting, bomb usage work correctly
- **Difficulty**: Easy/Normal/Hard changes spawn rates and enemy behavior
- **Bosses**: Each boss spawns after correct kill count and has unique patterns
- **Audio**: Sound effects play (test with mute button)
- **Mobile**: Virtual joystick and touch buttons functional
- **Accessibility**: Keyboard control works, screen reader announcements present
- **Persistence**: High score and settings survive page refresh

### Performance Testing
- Frame time tracking: `frameTimes` array in global scope
- Adaptive quality adjustment should trigger at ~25ms avg frame time
- Monitor canvas rendering cost on lower-end devices

## Notes for Future Work

- **Single-file constraint**: All code in one HTML file for simplicity; refactoring to modular structure would require build tooling
- **No external assets**: All audio is procedural, all graphics are canvas-drawn
- **Mobile-first**: Designed for touch input; keyboard/mouse are secondary
- **Canvas 2D only**: No WebGL or advanced graphics APIs used
