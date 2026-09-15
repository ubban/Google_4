# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built entirely as a single HTML5 file with Canvas rendering. Players control a fighter jet, switch between weapons, drop smart bombs, and face off against stage bosses. The game features touch controls for mobile devices and keyboard controls for desktop.

## Running the Game

- **Local Development**: Open `index.html` directly in a modern web browser. No build step or server required.
- **Modern Browsers Only**: Chrome, Firefox, Safari, Edge (2020+). Requires ES6 support and Canvas 2D API.
- **Responsive Design**: Game adapts to phone, tablet, and desktop viewports with appropriate control schemes (touch vs. keyboard/mouse).

## Code Architecture

The entire game is contained in `index.html` with three sections:

1. **HTML Structure** (lines 1-366): DOM elements for game canvas, HUD (health, bombs, weapon, multiplier), overlays (menu, game over), and touch control zones.

2. **CSS Styling** (lines 369-500+): 
   - Game canvas and container layout
   - HUD elements (stat boxes with backdrop blur)
   - Animations (heart beat, vignette pulse, combo text, stage banner)
   - Touch control buttons (fire, bomb) positioned bottom-right
   - Menu and game-over overlay styling with gradient text
   - Responsive typography using `clamp()` for scaling

3. **JavaScript Game Engine** (lines 500+, bulk of file):
   - **Game State**: difficulty, lives, score, multiplier, weapon, bomb count
   - **Core Loop**: Render loop updates game state, draws sprites, handles collisions
   - **Entity System**: Manages player, enemies, bullets, explosions via update/draw lifecycle
   - **Collision Detection**: Simple rectangular/circular bounding box checks
   - **Input Handling**: Keyboard (WASD, Space, Z for bomb), touch (joystick zones + buttons), mouse aim
   - **Audio**: Web Audio API for sound effects (shoot, explosion, combo, game over) generated via waveform synthesis

## Key Game Systems

- **Player**: Positioned at bottom center, moves via input, has health (lives), weapon state, bomb inventory
- **Weapons**: Toggle between Cannon (rapid fire) and Missiles (homing) with P/M keys
- **Enemies**: Spawn in formations, follow patterns, drop loot (health, bombs, multiplier boost)
- **Bosses**: Large enemies with 3-4 times normal health; activate stage banners and health bar UI when present
- **Scoring**: Base points for enemy kills multiplied by current multiplier (increased by enemy loot drops); combo counter visible at center during sprees
- **Difficulty Modifiers**: Easy/Normal/Hard buttons affect enemy spawn rates, health, and behavior patterns

## Development Guidelines

- **Single File**: All code lives in `index.html`. Structure is linear: markup, styles, then game logic. Keep this organization when adding features.
- **Canvas Coordinate System**: Origin (0,0) at top-left. X increases rightward, Y increases downward. Game area is 800×600 (adjusts responsively via canvas style).
- **State-Driven Rendering**: No DOM manipulation in game loop. All visuals via Canvas 2D API. UI overlays controlled by CSS classes (`.show`, `.active`).
- **Entity Updates**: All moving objects (player, enemies, bullets) follow `entity.update(dt)` then `entity.draw(ctx)` pattern each frame. Check `alive` flag to cull dead entities.
- **Sound Feedback**: Use `playSound(freq, duration, waveType)` helper for quick audio. Oscillator + gain node approach keeps sounds tight and responsive.

## Testing the Game

- **Manual Play**: Open in browser, use keyboard or touch controls, verify enemy patterns, boss behavior, weapon switching, and UI updates during play.
- **Edge Cases**:
  - Test on small viewports (phones) to verify touch controls don't overlap game area
  - Test difficulty buttons reset/change spawn and enemy health correctly
  - Verify game-over card shows correct final score and high-score comparison
  - Check that bomb animations and knockback feel responsive

## Modification Tips

- **Adding Enemy Types**: Define a new class with `update(dt)` and `draw(ctx)`. Add to spawn logic in the main update loop. Assign unique sprite color or pattern.
- **New Weapons**: Add weapon type constant (e.g., `WEAPON.LASER`), implement firing logic in player's weapon update, add weapon indicator color to HUD.
- **Boss Patterns**: Bosses are already large enemies. Extend spawn logic to detect boss conditions (e.g., stage threshold), increase health, and define movement/fire patterns.
- **Mobile Optimizations**: Joystick visual (#joystickBase, #joystickStick) is pre-styled but hidden. Show/hide and position dynamically if implementing virtual joystick replacement.

## Browser Compatibility Notes

- **ES6 Syntax**: Uses arrow functions, `const`/`let`, template literals. No polyfills; requires ES2015+ support.
- **Web Audio API**: Sound generation via `AudioContext`. Fallback to silence if unavailable (no error thrown).
- **Canvas 2D API**: Standard rendering context. No WebGL or advanced features.
- **Touch Events**: Supports `touchstart`, `touchmove`, `touchend` for mobile controls alongside keyboard/mouse.
