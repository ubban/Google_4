# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 Canvas application. The entire game (styling, game logic, and rendering) is contained in `index.html` (1669 lines).

No build system or external dependencies are required—the game runs directly in a browser.

## Architecture & Key Concepts

### Single-File Structure
- **index.html**: Contains all HTML markup, CSS styling, and JavaScript game logic in one file
- No external scripts, frameworks, or build tools
- Direct browser compatibility (modern ES6 JavaScript)

### Game Architecture
The JavaScript is organized into functional sections within an IIFE:

1. **Canvas Setup & Context**: Canvas element retrieval and 2D rendering context
2. **DOM Elements**: Caching of all DOM references (HUD, buttons, overlays, etc.)
3. **Storage Constants**: LocalStorage keys for best score, mute state, difficulty, motion preferences
4. **Utility Functions**: `announce()` for screen reader accessibility, performance tracking (`trackPerf()`)
5. **Difficulty Configuration**: `DIFFICULTIES` object defining spawn rates, enemy behavior, lives, and boss HP multipliers for Easy/Normal/Hard modes
6. **Game Constants**: `KILLS_PER_STAGE` (18) for progression, `THEMES` array with sky gradients and visual themes
7. **Game State**: Tracked variables (score, lives, bombs, current weapon, multiplier, game paused/running state, etc.)
8. **Game Loop**: `update()` function for frame-by-frame logic, `draw()` for rendering, `gameLoop()` as the RAF-driven main loop
9. **Input Handling**: Keyboard, mouse, and touch event listeners for player movement and actions
10. **Enemy & Bullet Systems**: Enemy spawning, movement, collision detection, bullet firing
11. **Player & Weapon Systems**: Player state management, weapon switching/upgrades, shooting mechanics
12. **Boss System**: Boss spawning at stage milestones, HP tracking, special attack patterns
13. **UI & Overlay Management**: Screen state (menu, playing, game over), HUD updates, visual feedback (flashes, vignette, combo popups)
14. **Accessibility & Preferences**: Screen reader announcements, reduced motion support, mute toggle, motion preferences

### Key Game Mechanics
- **Player Movement**: Controlled via mouse (crosshair follow), arrow keys/WASD, or touch drag
- **Weapon System**: Multiple weapons (Vulcan, Laser, Spread) with upgrade levels, toggled with weapon pickups
- **Smart Bombs**: Limited use bombs that clear screen and damage boss
- **Difficulty Scaling**: Enemy spawn rates and behavior change with difficulty; player lives and bomb count vary
- **Combo Multiplier**: Consecutive hits without player death increase score multiplier
- **Themes**: Six different visual themes with different sky gradients, clouds, and aesthetics applied per stage
- **Touch Controls**: Virtual joystick and on-screen buttons for mobile play
- **Performance Adaptation**: Automatic quality reduction (fewer effects, simpler rendering) if frame rate drops below 40 FPS

### Important Numbers & Thresholds
- **KILLS_PER_STAGE**: 18 enemy kills required to advance to next stage and spawn boss
- **Difficulty adjustments**:
  - Easy: 75 base spawn interval, 40 min, 1.0x speed, 0.45 shoot chance, 4 lives, 3 bombs, 0.8x boss HP
  - Normal: 58 base, 26 min, 1.4x speed, 0.7 shoot chance, 3 lives, 2 bombs, 1.0x boss HP
  - Hard: 40 base, 18 min, 1.9x speed, 0.95 shoot chance, 3 lives, 2 bombs, 1.3x boss HP

### Persistent State
- **Best Score**: Saved in localStorage as `skyfighter.bestScore`
- **Mute State**: Stored in localStorage as `skyfighter.muted`
- **Difficulty Selection**: Persisted as `skyfighter.difficulty`
- **Reduced Motion**: User preference stored as `skyfighter.reducedMotion`

## Development Guidelines

### Modifying Game Logic
- The main game loop runs via `requestAnimationFrame()` at approximately 60 FPS
- State updates happen in `update(dt)` where `dt` is delta time in milliseconds
- Rendering happens in `draw()`, reading from game state
- Always test changes across all three difficulty levels

### Adding New Features
- **New Enemy Types**: Add to enemy spawning logic and collision detection sections; define behavior in the enemy update section
- **New Weapon Types**: Add to the `WEAPONS` configuration object (if it exists as such), update weapon switching logic, and rendering
- **New Visual Effects**: Consider performance impact (the game has adaptive quality reduction); add to draw functions and respect `quality` and motion preference flags
- **New Difficulty Tweaks**: Modify the `DIFFICULTIES` object; test that progression scales appropriately

### Testing & Validation
- Test in desktop browsers (Chrome, Firefox, Safari) and mobile browsers
- Verify touch controls work correctly on actual touch devices
- Test all three difficulty levels for balanced progression
- Ensure accessibility features (screen reader announcements, reduced motion) work
- Monitor performance: check that adaptive quality reduction engages gracefully on slower devices

### Performance Considerations
- Quality drops from 0 (full) → 1 (medium) → 2 (low) when frame time exceeds 1/40th of a second
- Effects disabled under reduced motion mode (vignette pulse, heart beat animation, combo popup, etc.)
- Canvas rendering is the main bottleneck; minimize draw calls when possible
- Reuse Canvas rendering context methods; avoid creating temporary objects in hot loops

### Input & Controls
- **Keyboard**: Arrow keys or WASD move; Space fires; X or Shift drops bombs; P or Esc pauses
- **Mouse**: Move player toward mouse position; click to fire
- **Touch**: Drag on left side to move; tap FIRE button; tap BOMB button; optional virtual joystick display on certain setups

## Common Modifications

### Adjusting Difficulty Balance
Edit the `DIFFICULTIES` object to tweak spawn rates, enemy speed, shoot chances, and boss HP multipliers.

### Changing Visual Themes
The `THEMES` array defines sky gradients and color schemes. Add new entries or modify existing ones to change the game's appearance per stage.

### Adding Sound Effects
The game currently has mute toggle but no sound assets. Audio would need to be loaded and played based on game events (shooting, enemy death, bomb detonation, etc.).

### Tweaking Progression
Modify `KILLS_PER_STAGE` to change how many enemies need to be killed before advancing stages and spawning bosses.

## Browser Support & Compatibility
- Requires HTML5 Canvas support
- Uses modern JavaScript (ES6)
- Mobile-first responsive design using viewport units (clamp, vw, vh)
- Respects `prefers-reduced-motion` media query for accessibility
- Touch-action and pointer-events CSS for reliable mobile input

## Running & Testing
Simply open `index.html` in a modern web browser. No server, build step, or dependencies required.
To test locally during development, you can use any simple HTTP server:
```bash
# Python 3
python -m http.server 8000
# Or Node.js with http-server
npx http-server
```
Then navigate to `http://localhost:8000` and open `index.html`.
