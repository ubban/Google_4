# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5/Canvas application. The entire game (~1700 lines) is contained in `index.html` with embedded CSS and JavaScript.

## Getting Started

### Running the Game

Simply open `index.html` in a modern web browser. The game runs entirely client-side with no build process or external dependencies required.

**Browser Compatibility**: Chrome, Firefox, Safari, Edge (any modern browser with HTML5 Canvas, Web Audio API support).

### Testing/Verification

- Open the HTML file in a browser (file:// protocol is fine)
- Verify game loads with blue gradient sky background
- Test controls: arrow keys or mouse/touch for aiming, spacebar or click to fire
- Check that audio plays (if browser allows Web Audio)
- Test on mobile by resizing browser or using device emulator

## Architecture Overview

The game uses a **canvas-based game loop** architecture with discrete entity systems:

### Core Game Loop
```javascript
requestAnimationFrame(loop)  // Main loop (calls update → draw → render)
```

- **update(dt)**: Updates all game state (player position, bullets, enemies, collisions, etc.)
- **draw()**: Renders everything to canvas
- **Frame-based timing**: dt capped at 50ms to prevent spiral of death

### Entity Systems

1. **Player**: Single player ship at bottom of screen
   - Position: `player.x`, `player.y`
   - Health: `player.hp` (3 lives by default)
   - Weapon: switches between laser and bullet (press W)
   - Smart bombs: limited-use area attack (press B or Q)
   - Multiplier: increases with consecutive hits (decays on miss)

2. **Enemies**: Array of enemy ships with different types
   - Types: `scout` (fast, low hp), `fighter` (balanced), `tank` (slow, high hp)
   - Behavior: movement patterns, periodic shooting, health bars
   - Drop power-ups on death based on config

3. **Bullets**: Player projectiles
   - Two types: `laser` (continuous stream, high rate) vs regular (slower but more powerful)
   - Simple rect-based collision detection

4. **Enemy Bullets**: Enemy projectiles
   - Circular detection
   - Rendered separately from player bullets

5. **Power-ups**: Dropped by defeated enemies
   - `life`: restore health
   - `bomb`: add smart bomb charge
   - `weapon`: temporary weapon upgrade

6. **Boss**: Special enemy with multi-phase behavior
   - Appears after certain score/wave thresholds
   - More complex movement and attack patterns
   - Larger health pool

### Collision Detection

- **Pixel-perfect rect-vs-circle for bullets** vs enemies/player
- **AABB (axis-aligned bounding box)** for powerups
- Collisions trigger damage, score updates, particle effects

### Game State

Key global variables:
- `running`: whether game is active
- `paused`: whether currently paused (ESC key)
- `score`, `wave`, `frame`: progression metrics
- `quality`: performance level (0=glow effects on, 1=reduced)

## Code Structure (index.html)

```
<head>
  ├─ Meta tags
  └─ <style> (all CSS)
<body>
  ├─ <canvas id="gameCanvas">
  ├─ HUD display elements (score, hearts, bombs, etc.)
  └─ <script> (all JavaScript, ~1600 lines)
```

The JavaScript is organized as:

1. **Config/Constants** (ENEMY_TYPES, PROJECTILE, PLAYER, etc.)
2. **Game State** (player, enemies, bullets, score, etc.)
3. **Helper Functions** (collision, particle effects, drawing primitives)
4. **Update Logic** (player input, entity movement, spawning, collision handling)
5. **Render Functions** (canvas drawing, HUD updates)
6. **Game Loop** (requestAnimationFrame)
7. **Event Listeners** (keyboard, mouse, touch, UI buttons)

## Key Configuration Constants

Located near the top of the `<script>` block:

- `ENEMY_TYPES`: Defines scout/fighter/tank stats (hp, speed, fire rate, score)
- `PROJECTILE`: Bullet and laser behavior (speed, damage, fire rate)
- `PLAYER`: Player stats (hp, speed, bomb count)
- `SPAWN_RATE`, `WAVE_DURATION`: Difficulty progression
- `quality`: 0 = glow effects (better visuals), 1 = reduced (better performance)

## Making Modifications

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with stats (hp, speed, fireRate, score, etc.)
2. Update enemy spawn logic in `update()` to spawn your new type
3. Add drawing logic in `drawPlane()` if it looks different

### Adding a New Weapon
1. Define projectile properties in `PROJECTILE` section
2. Modify weapon-switching logic (search for "weapon" in update)
3. Update bullet rendering in `draw()`

### Tweaking Game Balance
- `SPAWN_RATE`: Control enemy spawn frequency
- `WAVE_DURATION`: How long each difficulty wave lasts
- Enemy type stats: hp, speed, fireRate, scoreValue
- Player speed and bomb count
- Weapon fire rates and damage values

### Performance Optimization
- The `quality` variable toggles glow effects (shadow blur is expensive)
- Particle count can be reduced by lowering particle emission in collision handlers
- Enemy/bullet limits can be capped if needed
- Use `trackPerf(dt)` to monitor frame time

## Mobile & Touch Support

- Game detects mouse position for aiming or accepts touch
- Mobile: tap/drag to aim, tap to fire
- Viewport is responsive; game scales to fit screen
- Safe area insets applied for notched devices

## Audio

- Web Audio API with generated sine wave tones (no external audio files)
- Muted on page load; unmuted on first user interaction
- Sound effects for: shooting, hits, explosions, power-ups

## Common Tasks

**Find where player position is updated**: Search `player.x`, `player.y` in update logic

**Find collision detection**: Search `checkCollision` or scan the collision section in update()

**Adjust difficulty**: Modify `WAVE_DURATION`, `SPAWN_RATE`, or enemy stats in config

**Change visual style**: Search CSS gradients, colors, and Canvas fill/stroke styles

**Add screen shake**: Use `shakeTime` variable (already implemented for bombs)
