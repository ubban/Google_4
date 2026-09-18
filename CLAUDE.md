# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game engine, graphics, and logic are contained within `index.html` (~1669 lines) using Canvas 2D API for rendering and vanilla JavaScript for game logic.

## Development & Testing

Since this is a single-file HTML project with no build process:

- **Run/Test**: Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
- **Edit**: Modify the CSS (in `<style>`) or JavaScript (in `<script>`) sections directly
- **Deploy**: The HTML file is self-contained and production-ready

## Game Architecture

The game follows a classic shoot-'em-up structure:

### Core Systems
- **Canvas Rendering**: 2D game loop drawing to `#gameCanvas` at ~60 FPS
- **Input Handling**: Keyboard (WASD/Arrow keys, Space/Z for fire/bomb) and touch controls
- **Physics**: Collision detection, projectile/enemy movement, damage/health tracking

### Game State
- **Player**: Position, health (0-3 hearts), current weapon, bombs (0-3), score, stage multiplier
- **Enemies**: Grid-based spawning, wave patterns, projectile fire, health/damage
- **Projectiles**: Player bullets, enemy fire, visual effects (impacts, explosions)
- **Bosses**: Stage bosses with pattern attacks and health bars

### Key Classes/Objects
- `Game`: Main controller managing state, input, collision
- `Player`: Tracks position, weapon, health, bombs
- `Enemy`: Enemy entities with AI firing patterns
- `Projectile`: Bullets and enemy fire
- `ParticleEffect`: Visual feedback (explosions, flashes, vignette)

### HUD Display
Stats shown at top: Health (hearts), Score, Wave/Stage, Current Weapon, Multiplier, Bombs

## Common Modifications

### Adding Weapon Types
Weapons are defined in the player firing logic. Modify the weapon cycle and projectile behavior in the `fire()` method.

### Adjusting Game Balance
- Enemy health/damage: Look for `enemy.health`, `damage` values
- Player health/invuln time: Search `PLAYER_` constants and collision handlers
- Difficulty/spawn rates: Enemy wave pattern arrays and timing constants

### Visual Polish
- Particle effects: `ParticleEffect` class and vignette/flash animations
- Color scheme: Update CSS `background`, `color` properties and gradient values
- Canvas styling: Adjust in `<style>` section (blur, shadows, etc.)

## Mobile Support

The game includes:
- Touch controls (movement zone on left, fire zone on right)
- Viewport configuration for mobile devices
- Safe area insets for notched devices
- Responsive sizing via `clamp()` CSS functions

## Performance Considerations

- Single-threaded game loop with delta-time calculations
- Pooled object patterns for bullets and enemies to reduce GC pressure
- Offscreen canvas rendering avoided; direct 2D context drawing
- No external dependencies (pure vanilla JavaScript)
