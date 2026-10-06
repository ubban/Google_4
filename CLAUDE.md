# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with HTML5 Canvas and WebAudio API. The entire game is contained in a single `index.html` file (~1700 lines) with no external dependencies, build step, or asset files.

## Running the Game

- **Local testing**: Open `index.html` directly in any modern web browser
- **No build required**: All code is inline (HTML, CSS, JavaScript)
- **No dependencies**: No npm packages, no external audio/image assets

## Code Architecture

The game is organized into functional sections within a single IIFE (Immediately Invoked Function Expression) to maintain a private scope:

### Core Game Systems

1. **Game State & Configuration**
   - Difficulty levels (easy, normal, hard) with scaling parameters
   - Score tracking with localStorage persistence (best score, difficulty preference)
   - 6 theme colors for visual variety
   - Constants for canvas sizing (BASE_W: 480, BASE_H: 720)

2. **Audio System** (WebAudio API, no external files)
   - Synthesized sound effects using oscillators, filters, and noise
   - `beep(freq, dur, type, vol, opts)` - generates tonal sounds with optional sweeps and filters
   - `noiseBurst(dur, vol, opts)` - generates noise-based effects
   - `sfx` object contains named effects: `shoot`, `laser`, `explode`, `bomb`, `powerup`, `bossHit`, `bossDown`, `stageClear`, etc.
   - Stereo panning based on screen position via `panFor(x)`
   - Volume control with master gain and compressor

3. **Rendering & Visual Effects**
   - Canvas resizing with `resize()` to maintain 480×720 aspect ratio while fitting screen
   - Particle explosions via `spawnExplosion(x, y, color, count)`
   - Screen shake effect with `screenShake(mag, dur)` (respects reduced-motion preference)
   - Flash effects for hits and bombs
   - Custom drawing functions: `drawPlane()` for player/enemies, `drawBossShip()` for bosses
   - Vignette and flash overlays for visual feedback

4. **Input Handling**
   - Keyboard controls: Arrow keys + Space/Z for fire + X for bomb
   - Touch/mobile controls with virtual joystick in moveZone
   - `joystickStart/Move/End()` - touch input handlers
   - `canvasPointFromEvent(e)` - converts client coordinates to canvas space

5. **Game Objects**
   - **Player**: Single ship with position, weapon level, health, bombs
   - **Enemies**: Three types with different properties (ENEMY_TYPES):
     - `grunt` (34×34, 1 HP, speed 1.0x, score 10)
     - `interceptor` (26×26, 1 HP, speed 1.7x, weaving pattern, score 15)
     - `tank` (46×46, 4 HP, speed 0.55x, score 35)
   - **Bosses**: Six types (BOSSES array) with unique color schemes and HP multipliers:
     - VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
   - **Projectiles**: Bullets and lasers with different patterns
   - **Powerups**: Drop from enemies, upgrades weapon level
   - **Particles**: Explosions and visual effects

6. **Game Logic**
   - Stage progression: kills per stage (KILLS_PER_STAGE = 18)
   - Enemy spawning with difficulty scaling
   - Boss spawning after stage completion
   - Collision detection via `rectsOverlap(a, b)`
   - Combo/multiplier system that breaks on missed kills
   - Weapon progression system (normal → laser at level 2)
   - Bomb count management with `useBomb()`
   - Score calculation with `addScore(base)` applying current combo multiplier

7. **UI & State Management**
   - HUD displays: score, best score, multiplier, weapon name/level, hearts (health), bombs, combo popup
   - Stage banners announce progression
   - Overlay screens for game over, difficulty selection, start menu
   - Accessibility: reduced-motion preference support, screen reader live region
   - localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Key Development Patterns

### Game Loop
Main update/render loop drives all game state changes. Performance is tracked via `trackPerf(dt)` to adapt in low-frame environments.

### Collision System
Axis-aligned bounding boxes only. `rectsOverlap(a, b)` checks if two rect objects (with x, y, w, h) intersect.

### Weapon System
- Level 0: Normal bullets (single, straight)
- Level 1+: Laser (stronger, wider hitbox)
- Upgraded by picking up powerup drops from enemies
- Displayed in HUD with current level

### Difficulty Scaling
DIFFICULTIES object defines:
- Enemy spawn rate and count multiplier
- Enemy speed scaling
- Boss HP scaling
- Affects combo breakage timing and player health

### Mobile/Touch
- Virtual joystick appears only on touchscreen devices
- Fire and bomb buttons alongside movement zone
- Respects device motion preferences

## Common Editing Tasks

**Add a new enemy type**: Define in ENEMY_TYPES (properties: w, h, hp, speedMul, color, dark, score, minStage). Add spawning logic in `spawnEnemy()`.

**Add a new boss**: Add entry to BOSSES array (name, color scheme, hpMul). Bosses auto-progress through stages.

**Add sound effect**: Add function to `sfx` object using `beep()` or `noiseBurst()`. Call it when event occurs (e.g., `sfx.powerup()` on powerup pickup).

**Adjust difficulty**: Modify DIFFICULTIES object parameters (spawnRate, speedMul, bossHpMul, etc.) or KILLS_PER_STAGE.

**Change visual style**: Theme colors in THEMES array control color scheme per stage. Adjust canvas background gradient in CSS.

## Testing Notes

- No automated tests (single-file game)
- Manual testing: Open in browser, test each difficulty, test all weapon levels, trigger boss fights
- Check mobile responsiveness by resizing browser or testing on actual device
- Verify audio works (may require user gesture to enable AudioContext)
- Check localStorage persistence (best score, mute state, difficulty) across page reloads

## Performance Considerations

- Canvas rendering is optimized for 60 FPS
- Particle limit and draw call count auto-adapt if frame rate drops
- No WebGL or 3D rendering (2D Canvas only)
- Sound synthesis uses minimal CPU (oscillators + filters)
- Mobile performance depends on device CPU
