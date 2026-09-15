# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game implemented as a single-file web application. Players navigate a fighter aircraft through procedurally spawned stages, battle enemies with weapon upgrades, deploy smart bombs, and face stage bosses. The game features adaptive difficulty, procedurally generated audio effects, and full accessibility support including screen reader announcements and reduced-motion preferences.

## Architecture

### Single-File Structure
- **index.html** (1669 lines): Complete game implementation with embedded CSS and JavaScript
- No build system, dependencies, or external assets required
- Pure HTML5/Canvas/WebAudio API implementation
- Playable directly in any modern browser (open file locally or serve via HTTP)

### Game Loop Architecture
The game runs on a single `requestAnimationFrame` loop:
```
loop() -> update(dt) -> draw()
```
- `update(dt)`: Advances physics, collision detection, enemy AI, projectiles
- `draw()`: Canvas-based 2D rendering with effect overlays (shake, flash, vignette)
- Frame time capped at 50ms to prevent spiral (dt = min(dt, 0.05))
- Adaptive quality scaling: if average frame time drifts, quality level (0=full, 1=medium, 2=low) adjusts automatically

### Core Game Objects (Tracked in Arrays)
All game entities are tracked in global arrays updated each frame:
- **player**: Single player object with position, health, weapon state, bomb count
- **bullets**: Player projectiles with position, velocity, damage, lifetime
- **enemies**: Spawned enemies (grunt/interceptor/tank) with position, velocity, health, AI behavior
- **enemyBullets**: Enemy projectiles, collision detected against player
- **explosions**: Particle effects for visual feedback
- **powerUps**: Weapon/utility pickups dropped by defeated enemies
- **boss**: Active boss object (null when no boss) with complex animation state and attack patterns
- **bossBullets**: Boss attack projectiles with specialized patterns

### Key Subsystems

#### 1. Difficulty System (lines 485-489)
Three preset configurations stored in `DIFFICULTIES` object (easy/normal/hard):
- Enemy spawn rates, speed multipliers, shoot chances
- Player lives and bomb count
- Boss HP multiplier
- Changed via `setDifficulty(name)`, persisted to localStorage

#### 2. Audio System (lines 543-676)
WebAudio API implementation with no external audio files:
- **beep()**: Synthesized tones with frequency sweep, filters, panning, ADSR envelope
- **noiseBurst()**: Procedural white noise with lowpass filter decay
- **sfx object**: Named sound effects (shoot, laser, explode variants, bomb, powerup, boss sounds, etc.)
- Audio context lazy-initialized on first user interaction (required by browser policies)
- All sounds are completely procedural and bandwidth-free

#### 3. Input Handling (lines 768-870)
Supports keyboard (arrows/WASD, space, X/Shift for bomb), mouse (movement + click), and touch (joystick UI):
- `keys` object: Tracks pressed key state
- `joystickStart/Move/End()`: Touch joystick tracking with visual feedback
- `mouseTarget`: Mouse position for aiming
- `canvasPointFromEvent()`: Converts DOM coordinates to canvas space accounting for scaling

#### 4. Rendering & Visual Effects (lines 698-1519)
- **flashHit/flashBomb()**: CSS animations for hit/bomb feedback
- **screenShake()**: Temporal shake effect with mag and duration
- **showComboPop()**: Floating combo text feedback
- **drawPlane()**: Renders player and enemy ships with rotation, health, engine flame
- **drawBossShip()**: Complex boss visual with hull + wings + core components
- **drawClouds()**: Parallax cloud backgrounds with theme colors

#### 5. Collision & Physics (lines 1069-1087)
- **rectsOverlap()**: AABB collision detection for all entity pairs
- Score multiplier system: increases on consecutive kills, resets on player hit
- Collision checks: player vs enemy bullets, enemy vs player bullets, pickups, boss patterns

#### 6. Boss AI & Attack Patterns (lines 1043-1058, 1324-1367)
Boss object selected randomly from `BOSSES` array (6 unique boss types). Each boss has:
- Unique visual colors (hull, wings, core)
- Custom attack pattern sequence via `fireXxx()` functions:
  - **fireRadial()**: Bullets in all directions
  - **fireAimedSpread()**: Spread toward player
  - **fireColumns()**: Vertical columns
  - **fireSpiralBurst()**: Spinning pattern
  - **fireDoubleRing()**: Concentric rings at different speeds
  - **fireHomingSeeded()**: Tracking shots
- Attack selection via `updateBoss()` state machine; no explicit turn-based system

#### 7. UI & Persistence (lines 463-540, 875-912)
- HUD displays score, best score, multiplier, weapon level, health hearts, bombs
- Overlay system for start screen and game over screen with difficulty selection
- LocalStorage persistence:
  - `skyfighter.bestScore`: High score
  - `skyfighter.difficulty`: Last played difficulty
  - `skyfighter.muted`: Audio mute state
  - `skyfighter.reducedMotion`: Accessibility setting
- Screen reader announcements via `announce()` for game state changes

#### 8. Progression System
- **Stages**: Each stage requires `KILLS_PER_STAGE` (18) enemy kills to advance
- **Themes**: 6 visual themes (Daylight Front, Sunset Corridor, Night Approach, etc.) cycle per stage
- **Enemy Types**: Grunts from stage 1; Interceptors from stage 1; Tanks from stage 2+
- **Weapon Progression**: Picking up weapon power-ups advances weapon level (displayed as numeric level)
- **Power-ups**: Weapon (W icon), bomb (B icon), health (H icon) randomly spawned from defeated enemies

## How to Run & Develop

### Playing the Game
```bash
# Option 1: Open directly in browser
open index.html

# Option 2: Serve locally with any HTTP server
python3 -m http.server 8000
# Then navigate to http://localhost:8000
```

Controls:
- **Desktop**: Mouse movement to aim, click to fire, Space/X to bomb, Arrows/WASD to move, P/Esc to pause
- **Mobile**: Drag left zone to move, tap FIRE button, tap BOMB button

### Making Changes
Since this is a single-file project with no build step:
1. Edit `index.html` directly
2. Reload browser to see changes
3. Use browser DevTools (F12) for debugging and inspecting game state

### Testing & Debugging
- Open browser console (F12) to see errors
- Game state variables are in global scope within the IIFE (use debugger breakpoints)
- Difficulty/best score/audio settings are preserved in localStorage (`skyfighter.*`)
- Remove localStorage entries to reset: `localStorage.clear()`

### Common Development Tasks

**Adding a new enemy type**: 
- Add entry to `ENEMY_TYPES` object (line 502) with sprite dimensions, hp, speed, score, minStage
- Update `spawnEnemy()` (line 1022) to include new type in spawn pool
- Add explosion SFX mapping in `EXPLODE_SFX` (line 678)

**Adding a new weapon pattern**:
- Create new `fireXxx(boss, count, speed, ...)` function following existing patterns (lines 1324-1367)
- Call it from `updateBoss()` state machine (line 1411)

**Adding visual effects**:
- Use `screenShake(magnitude, duration)` for impact feedback
- Use `flashHit()` or `flashBomb()` for flash animations
- Add vignette with `vignetteEl.classList.toggle('on', condition)`

**Adjusting difficulty values**:
- Modify `DIFFICULTIES` object (line 485): spawn rates, enemy speed, lives, bombs, boss HP
- Values are in range 0-1 or specific counts (lives, bombs)

**Adding a new boss**:
- Add entry to `BOSSES` array (line 508) with visual colors and HP multiplier
- Boss is selected randomly when `spawnBoss()` is called (line 1043)

## Performance Considerations

- **Adaptive Quality**: Frame time tracking (line 476) automatically scales quality (1-3 levels) based on performance
- **Canvas Rendering**: 2D context is used for all visuals; no WebGL or 3D effects
- **Entity Pooling**: Explosions and projectiles are created/destroyed dynamically (no pooling currently)
- **Audio**: Procedural synthesis is CPU-light compared to audio file playback, but many simultaneous sounds can accumulate (master gain compressor helps)

## Accessibility Features

- **Screen Reader Support**: Game state announced via ARIA live region (`srLive`)
- **Reduced Motion Preference**: Respects browser's `prefers-reduced-motion` media query; setting persisted to localStorage
- **Keyboard Navigation**: Full keyboard control, no mouse required (except optional mouse steering)
- **Visual Feedback**: Color contrast, text shadows for readability on gradient backgrounds

## Browser Compatibility

- Requires HTML5 Canvas 2D context
- Requires ES6 JavaScript (arrow functions, const/let, template literals)
- WebAudio API for sound (gracefully degrades to silent if unavailable)
- Touch events for mobile support (fallback to keyboard/mouse on desktop)
- LocalStorage for persistence (gracefully degrades if unavailable)

Tested on modern browsers: Chrome, Firefox, Safari, Edge. Mobile support: iOS Safari, Android Chrome.
