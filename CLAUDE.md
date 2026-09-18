# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (~1,669 lines).

## Development

### Running the Game

The game runs directly in a browser with no build step required:

```bash
# Simply open the file in a browser
open index.html
# Or serve locally for testing
python -m http.server 8000
# Then visit http://localhost:8000/index.html
```

### Code Style & Format

The codebase uses:
- **ES6 JavaScript** with no external dependencies
- **Canvas API** for rendering
- **Web Audio API** for synthesized audio (no external audio files)
- **Touch and keyboard input** with adaptive performance

Since this is a single-file project, all code is in the IIFE (Immediately Invoked Function Expression) scope within `<script>` tags to avoid global pollution.

## Architecture

### Game Loop

The game runs on a `requestAnimationFrame`-based loop (`loop(now)` function) that:
1. Calculates delta time (dt) since last frame
2. Calls `update(dt)` for game logic and state changes
3. Calls `draw()` to render the current frame
4. Tracks adaptive performance metrics to maintain 60 FPS

### Core Game State

All game state is managed in module-scoped variables:

```javascript
let player, bullets, enemies, enemyBullets, particles, powerUps, boss;
let score, lives, running, paused, gameOver;
let chain, chainTimer, multiplier;  // Combo system
let frame, stage, stageKills;       // Progress tracking
```

**Key Objects:**

- **player**: Player ship state (position, velocity, weapon, level, cooldown, invulnerability)
- **bullets**: Player projectiles with physics and behavior
- **enemies**: Enemy ships with AI patterns, health, and projectiles
- **boss**: Boss entity for stage bosses with complex firing patterns
- **particles**: Visual effects (explosions, hits)
- **powerUps**: Collectible items that modify player stats (health, bombs, weapons, multiplier)

### Weapon System

Multiple weapons with progression levels (1-5):
- **Vulcan**: Rapid-fire multi-projectile weapon
- **Laser**: High-damage concentrated beam

Weapons are selected via UI buttons and stored in `player.weapon`. Each weapon fires using distinct firing patterns (`fireWeapon()` function) with projectile spread determined by weapon level.

### Enemy Patterns

Boss and enemy firing patterns are implemented as separate functions:
- `fireRadial(b, count, speed, offset)`: Projectiles in circular patterns
- `fireAimedSpread(b, count, speed, spreadStep)`: Spread aimed at player
- `fireColumns(b, speed)`: Vertical columns
- `fireSpiralBurst(b, speed)`: Spiraling patterns
- `fireCross(b, speed)`: Cross patterns
- `fireHomingSeeded(b, count, speed)`: Homing projectiles with noise

Each enemy uses a `patternIndex` to vary firing over time, creating dynamic difficulty.

### Collision Detection

Simple rectangular AABB collision via `rectsOverlap(a, b)`:
```javascript
a.x < b.x + b.w && a.x + a.w > b.x &&
a.y < b.y + b.h && a.y + a.h > b.y
```

Used for:
- Player-bullet collisions (damage)
- Player-powerup collisions (buffs)
- Enemy-bullet collisions (destruction)
- Boss-bullet collisions (phase progression)

### Difficulty System

Three difficulty levels defined in `DIFFICULTIES` object with parameters:
- `lives`: Starting health count
- `bombs`: Max bomb count
- `enemyHealth`: Enemy hit points
- `bossHealth`: Boss hit points
- `spawnRate`: Enemy spawn frequency

### Combo & Scoring

- **Chain**: Hit counter incremented on successful hits, reset on misses
- **Multiplier**: Score multiplier based on chain length (caps at 16x)
- **Score**: Base score multiplied by current multiplier
- Power-ups can reset or extend chains

### Audio

Synthesized WebAudio with no external files:
- `beep(freq, dur, type, vol, opts)`: Sine/square wave tones
- `noiseBurst(dur, vol, opts)`: Noise effects for explosions/impacts
- Stereo panning via `panFor(x)` based on canvas position
- Mute toggle via `updateMuteBtn()`

### Visual Effects

- **Screen shake**: `screenShake(mag, dur)` for impacts
- **Flash effects**: `flashHit()` and `flashBomb()` with CSS animations
- **Particle system**: Explosion particles with velocity and decay
- **Combo popups**: `showComboPop(text)` for score feedback
- **Stage banners**: `showStageBanner(text)` for progression cues
- **Vignette**: Dynamic darkening when health is critical
- **Cloud scrolling**: Parallax background layers via `drawClouds(speed, fillColor, rx, ry)`

### Input Handling

- **Keyboard**: Arrow keys or WASD for movement, Space for fire
- **Mouse**: Aim toward cursor, click to fire
- **Touch**: Virtual joystick with dual-stick support on smaller screens
- Joystick state: `joystickStart()`, `joystickMove()`, `joystickEnd()`

### Performance Optimization

Adaptive performance tracking in `trackPerf(dt)`:
- Monitors frame time to detect performance bottlenecks
- Dynamically adjusts particle count and effect quality
- Skips expensive operations when frame time exceeds target (16.67ms @ 60 FPS)
- Uses `reducedMotion` flag for accessibility (screen shake, flashing disabled)

## Key Functions by Category

### Game Control
- `resetGame()`: Initialize/restart game
- `update(dt)`: Main game logic each frame
- `draw()`: Render all game entities
- `togglePause()`: Pause/resume gameplay
- `endGame()`: Handle game over state

### Player Actions
- `fireWeapon()`: Fire current weapon with spread
- `useBomb()`: Clear screen and damage boss (limited uses)
- `applyPowerUp(type)`: Apply collected power-up effect

### Spawning
- `spawnEnemy()`: Create random enemy at top of screen
- `spawnBoss()`: Create stage boss with health bar
- `spawnExplosion(x, y, color, count)`: Create particle explosion
- `maybeSpawnPowerUp(x, y)`: Randomly drop power-up on enemy death

### State Updates
- `loseLife()`: Reduce player health, check game over
- `killBoss()`: Complete stage, spawn next stage
- `addScore(base)`: Add to score with multiplier, update chain
- `breakChain()`: Reset combo on missed shot

### UI Updates
- `renderHearts(count, max)`: Update health display
- `renderBombs(count, max)`: Update bomb count display
- `updateMotionBtn()`: Toggle reduced motion preference
- `setDifficulty(name)`: Change difficulty level before game start

## Common Modifications

### Adding a New Enemy Type
1. Create spawn logic in `spawnEnemy()` (probability-based branch)
2. Add firing pattern function (e.g., `firePattern()`)
3. Add update logic in `update(dt)` for enemy movement and pattern progression
4. Test collision detection and pattern visibility

### Adjusting Difficulty
Edit `DIFFICULTIES` object constants for lives, bomb count, health values, and spawn rates.

### New Power-up
1. Add type handling in `maybeSpawnPowerUp()`
2. Implement effect logic in `applyPowerUp(type)`
3. Update UI if needed (weapon name, level display, etc.)

### Weapon Tweaks
Modify `fireWeapon()` for fire rate and spread, or adjust individual weapon firing functions (`fireRadial`, `fireAimedSpread`, etc.) for pattern changes.

## Testing Tips

- Use browser DevTools to inspect canvas and pause at breakpoints
- Test on multiple screen sizes (mobile, tablet, desktop) for input and rendering
- Verify audio works with system mute and browser-level audio policies
- Test reduced motion mode for flashing/shake effects
- Verify frame rate stays near 60 FPS during heavy spawn periods

## File Structure

```
index.html  – Complete game (1,669 lines)
  ├─ <style> – CSS for UI, layout, animations
  ├─ <body> – Game canvas and UI elements
  └─ <script> – Game engine and logic
      ├─ Adaptive performance tracking
      ├─ Audio synthesis
      ├─ Input handling (keyboard, mouse, touch)
      ├─ Game state management
      ├─ Rendering functions
      ├─ Game loop
      └─ Event listeners and initialization
```

## Browser Compatibility

The game uses:
- Canvas 2D Context (widely supported)
- Web Audio API (requires modern browser)
- requestAnimationFrame (standard)
- LocalStorage for high score persistence
- Touch events for mobile

Target browsers: Chrome 60+, Firefox 55+, Safari 12+, Edge 79+
