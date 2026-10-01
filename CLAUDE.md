# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (HTML markup, CSS styling, and JavaScript game logic) is contained in `index.html` (1669 lines). This is a client-side canvas-based game with no external dependencies.

## Running the Game

The game runs directly in any modern web browser. No build step is required.

**To test locally:**
- Open `index.html` directly in a web browser, or
- Serve via HTTP (recommended for best performance): `python3 -m http.server 8000` then visit `http://localhost:8000`

The game auto-detects the browser environment and provides both keyboard and touch controls.

## High-Level Architecture

### Monolithic Structure
All code lives in one `index.html` file to ensure portability and self-containedness. This includes:
- **HTML**: Single canvas element + overlay UI for menus
- **CSS**: Embedded styles with dark theme, responsive design using `clamp()` for mobile-friendly scaling, safe-area insets for notched devices
- **JavaScript**: Complete game engine (game loop, collision detection, rendering, state management)

### Game Engine Components

1. **Game State & Config**
   - Global state variables track player HP, score, weapon, multiplier, bomb count
   - Stage/difficulty system affects enemy spawn rates and damage
   - Difficulty modes: Easy, Normal, Hard each with different parameters

2. **Gameplay Loop** (`gameLoop()` function)
   - Runs at fixed time steps (20ms, ~50 FPS)
   - Clears canvas, updates game objects, renders everything
   - Handles collision detection between player bullets and enemies/bosses, and enemy bullets and player

3. **Object Types**
   - **Player**: Single plane at bottom; position controlled by mouse/touch; health tracked; weapons switch
   - **Enemies**: Spawned in waves; multiple types (fighter, tank, etc.) with varying behaviors and HP
   - **Boss**: One per stage; longer fight; different attack patterns than regular enemies
   - **Bullets**: Tracked in separate arrays for player bullets (`bullets`) and enemy bullets (`enemyBullets`)
   - **Power-ups**: Drop from certain enemies; restore health, increase score multiplier, or grant bombs

4. **Weapon System**
   - Player cycles through weapon types: Bullets → Lasers → Missiles
   - Weapon switching is automatic based on fired shots count
   - Lasers have glow effects; missiles fire in spread pattern
   - Each weapon type has different damage and fire rates

5. **UI Elements**
   - **HUD** (top center): Score, health hearts, bombs, current weapon, multiplier
   - **Boss Bar** (top center when fighting boss): Boss name and health bar
   - **Overlay** (full-screen modal): Title screen, game-over screen, difficulty selection
   - **Flash/Vignette Effects**: Screen flashes on hit; red vignette pulses during low health
   - **Combo Pop**: Floating text showing multiplier increases

### Rendering & Visual Effects

- **Canvas 2D API**: All drawing uses `ctx.` methods (fillRect, arc, drawImage via canvas operations)
- **Gradient backgrounds**: Sky gradient on canvas; glass-morphism UI cards with backdrop-filter blur
- **Responsive design**: Uses CSS `clamp(min, preferred, max)` for font sizes and spacing that scale from mobile to desktop
- **Safe area insets**: Respects notched phones with `env(safe-area-inset-*)` in CSS
- **Touch optimization**: Disables tap highlight, sets `touch-action: none`, uses no hover states

### Input Handling

- **Keyboard**: Arrow keys or WASD for player movement
- **Mouse**: Click or move to aim; weapons fire continuously toward cursor
- **Touch**: Drag to move player; tap to toggle aiming; visual touch zones shown on mobile

### Data Persistence

- High score stored in `localStorage` under key `skyFighterHighScore`
- Persists across browser sessions

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with `hp`, `speed`, `fireRate`, `dmg`, `color`, `points`
2. Update spawn logic in `updateEnemies()` to decide when this type appears
3. Implement behavior in `updateEnemies()` loop (movement, firing pattern)

### Adjusting Game Balance
- **Enemy difficulty**: Modify `ENEMY_TYPES` values or spawn frequency in `updateEnemies()`
- **Player damage**: Change bullet damage multipliers in weapon fire logic
- **Health/bombs**: Adjust `PLAYER_HEALTH` constant or power-up drop rates
- **Difficulty modes**: Modify parameters in the difficulty selection UI and their effects on game state

### Adding a New Boss
1. Define boss properties in the boss initialization section
2. Implement attack pattern in the boss AI update logic within `updateEnemies()`
3. Add boss name to the stage progression (`stageNames` array)

### Tweaking Visual Effects
- **Colors**: Modify hex values in CSS (`:root`, animations) or in JavaScript drawing code
- **Animations**: CSS keyframes for UI elements; JavaScript-driven particle effects (bullets, explosions) use opacity/transform
- **Mobile responsiveness**: Adjust `clamp()` values in CSS for better mobile scaling

### Performance Optimization
The game runs at 50 FPS to balance responsiveness and battery drain on mobile. If optimizing:
- Reduce effect glow intensity in `quality` variable checks
- Batch canvas operations to reduce draw calls
- Cull off-screen bullets/enemies before rendering

## Key Code Patterns

- **State machine via overlay visibility**: Title → Game → Game Over → Title
- **Delta time**: Uses fixed time steps; frame-rate independent via consistent update intervals
- **Object pooling**: Bullets and particles reuse array slots to avoid garbage collection
- **Simple physics**: Linear movement + basic velocity; collisions use AABB (axis-aligned bounding box) overlap
- **Power-up system**: Drop-on-death mechanics with random selection

## Browser Compatibility

Targets modern browsers supporting:
- Canvas 2D (`<canvas>`, `ctx` methods)
- `localStorage` for high score
- Viewport units and CSS custom properties (`:root`)
- Touch events (iOS Safari, Android browsers)
- Keyboard events

## Notes for Future Development

- The single-file design ensures easy distribution (no build step, no external assets needed)
- Minification would reduce file size but keep the single-file design
- Mobile testing is critical—always verify touch controls and safe-area insets on actual devices
- The fixed 50 FPS loop can be adjusted by changing the `dt` (delta time) calculation
- Sound/music would require either embedded data URIs or external files (breaks self-containedness)
