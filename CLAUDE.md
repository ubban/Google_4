# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 game — a Raiden-style vertical shoot-'em-up built with canvas-based rendering. The game includes weapon switching, smart bombs, stage bosses, combo scoring, difficulty levels, and both mouse/keyboard and touch controls.

## Running & Testing

### Running Locally

The game is a static HTML file. To test:

```bash
# Option 1: Use Python's built-in server (Python 3)
python -m http.server 8000

# Option 2: Use Node's simple server
npx http-server .

# Option 3: Use any static server (Nginx, Apache, etc.)
```

Then open `http://localhost:8000` in a browser.

### Playing the Game

- **Mouse/Keyboard**: Move mouse, click to shoot, press 1/2 to switch weapons, Space for bombs
- **Touch**: Use on-screen joystick and buttons on mobile
- **Pause**: Press P (mouse/keyboard mode)
- **Difficulty**: Select before starting (Easy/Normal/Hard)

## Architecture Overview

The entire game logic is in `index.html` (~1669 lines). The structure is:

1. **HTML/Metadata** (lines 1-9): Title, viewport config, favicon
2. **CSS Styling** (lines 10-450): Game canvas, HUD display, UI overlays, animations, responsive design
3. **JavaScript Game Engine** (lines 451-1669):
   - **Initialization**: Canvas setup, event listeners (mouse, keyboard, touch)
   - **Game Loop**: `gameLoop()` function drives 60 FPS updates via `requestAnimationFrame`
   - **Entity Management**: 
     - `player`: Player ship with position, velocity, health, bombs, weapon state
     - `enemies`: Array of spawned enemy planes (various types: fighter, tank, boss)
     - `bullets`: Array of player bullets (standard and laser)
     - `enemyBullets`: Array of bullets fired by enemies
     - `powerUps`: Array of power-up pickups (health, weapons, bombs)
   - **Collision Detection**: AABB (Axis-Aligned Bounding Box) collision checks between:
     - Player bullets ↔ Enemies
     - Enemy bullets ↔ Player
     - Power-ups ↔ Player
   - **Rendering Pipeline**:
     - Background gradient (sky blue fading darker)
     - Enemy ships and bosses
     - Player ship with damage state (dark red tint when hit)
     - Bullets (laser vs standard with different colors/glow)
     - HUD: Score, health, bombs, weapon type, combo multiplier
     - Boss health bar
     - Visual feedback: Flash effects (white on hit, full white on bomb), vignette on damage
   - **Gameplay Systems**:
     - **Weapons**: Standard bullets (3/sec) and Laser (continuous), switching with 1/2 keys
     - **Bombs**: Limited use, clears bullets and damages/kills enemies
     - **Enemy AI**: Move down screen, fire at player, spawn patterns per stage
     - **Bosses**: Tank-style HP bars, movement patterns, intense bullet patterns
     - **Scoring**: Base points + combo multiplier (resets on damage, grows with hits)
     - **Difficulty**: Easy/Normal/Hard change spawn rates, damage, health values

## Key Code Sections

- **Enemy Spawn Patterns** (`spawnWave()`): Defines which enemies appear, where, when
- **Boss Definitions** (`BOSS_TYPES`): Boss parameters (HP, speed, movement, bullet patterns)
- **Collision Detection** (`checkCollisions()`): Central collision resolution
- **Rendering** (`ctx` canvas commands): Drawing player, enemies, bullets, visual effects
- **Touch/Input Handling**: Separate handlers for mouse events and touch (mobile joystick)

## Development Tips

### Modifying Game Balance

- **Difficulty parameters**: Look for `DIFFICULTY` object (defines enemy health, spawn rates per difficulty)
- **Weapon damage**: Search `bullet` creation — adjust damage values
- **Boss HP/behavior**: Modify `BOSS_TYPES` object
- **Enemy spawning**: Adjust `spawnWave()` spawn intervals and enemy counts

### Adding Visual Features

- Canvas rendering is centralized in the game loop's drawing section
- CSS defines HUD styling; modify `#hud`, `.stat`, `#hearts`, `#bombs` for interface changes
- Animations use CSS keyframes (`@keyframes`) — add new ones as needed

### Touch/Mobile Optimization

- Touch controls are managed separately; tweak virtual joystick sensitivity in touch event handlers
- Safe area insets via `env(safe-area-inset-*)` for notched devices

### Performance Considerations

- Quality setting (accessible via settings) disables glow effects at quality=0 for lower-end devices
- Canvas rendering is immediate (no double-buffering abstraction needed)
- Enemy/bullet pooling is implicit (arrays are cleared each frame)

## Browser Compatibility

The game uses:
- Canvas 2D Context (universal)
- `requestAnimationFrame` (all modern browsers)
- Touch Events (mobile)
- CSS: `safe-area-inset-*`, `backdrop-filter`, `environment` variables (modern browsers)

Works on all modern browsers (Chrome, Firefox, Safari, Edge) and mobile platforms.

## No Build/Test Setup

This is a single-file game with no build process, linting, or unit tests. Changes are immediate—just edit and refresh the browser.
