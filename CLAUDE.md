# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single self-contained HTML file. The entire game (HTML, CSS, and JavaScript) lives in `index.html` (~1,670 lines). It features:

- Vertical scrolling shooter gameplay with a player-controlled ship
- Multiple weapon types (standard shot, wide spread, laser)
- Smart bombs (limited use)
- Stage bosses with patterns
- Score tracking and combo multipliers
- Lives and health system
- Responsive design (scales to container, works on mobile/tablet)
- Audio feedback (on-demand audio context initialization for browser restrictions)

## How to Run

1. **Open directly**: Simply open `index.html` in a web browser
2. **Local server** (recommended for testing): Run a simple HTTP server:
   ```bash
   python3 -m http.server 8000
   ```
   Then visit `http://localhost:8000`

The game runs entirely client-side with no build step, bundler, or external dependencies.

## Code Architecture

The `index.html` file is organized into distinct sections within a single `<script>` block:

### Game State & Constants
- Game dimensions, spawn rates, boss patterns
- Asset definitions (sprites as inline canvas draws)
- Configuration for difficulty, scoring, weapon stats

### Utility Functions
- Math helpers (distance, collision detection)
- Canvas drawing helpers (rotation, scaling, particles)
- Audio playback with context management

### Game Objects (Player, Enemies, Bullets, Particles)
- Each has an update and render pass
- Projectiles and powerups managed in arrays
- Collision detection happens during update phase

### Game Loop & Control Flow
- `update(dt)` - Physics, collision, AI, spawning
- `draw()` - Render all game objects to canvas
- `loop(now)` - RequestAnimationFrame callback; handles delta time, calls update/draw
- Input handling via mouse/touch events

### UI & State Management
- HUD overlay (lives, score, weapon, multiplier)
- Screen overlays (start menu, game over, pause)
- LocalStorage for high score persistence

## Common Development Tasks

### Modifying Game Behavior

**Adjust difficulty/balance:**
- Spawn rates, enemy health, boss patterns: search for numeric constants near top of script
- Weapon damage: look for bullet velocity, fire rate, or damage values
- Player movement speed: search "player speed" or similar

**Add a new weapon type:**
1. Define weapon properties in the weapons array at top
2. Add firing logic in the player update section
3. Add rendering in the player render section
4. Update weapon cycle logic

**Tweak visual effects:**
- Explosion particles: modify particle generation in `createExplosion()`
- Vignette/flash: adjust opacity or timing in `#vignette.on` keyframes or flash triggers
- Trail effects: modify particle color/lifetime in player/bullet render sections

**Change level pacing:**
- Boss spawn time: `bossSpawnTime` constant
- Stage duration: `stageDuration` constant
- Wave patterns: modify `waves` array and spawn logic in update

### Testing Gameplay

The game has no separate test suite. To test:

1. Open index.html in browser
2. Click "Start" to begin
3. Manually play through stages and bosses
4. Verify audio, collision, scoring, and visuals work as expected
5. Test on mobile/tablet for touch controls if responsive design was modified

### Debugging

- Browser DevTools (F12): Open console to see any JS errors
- Pause the game (spacebar) to inspect state
- The HUD displays current score, lives, weapon, and multiplier in real-time
- Performance monitor shows FPS in bottom-right corner

### Modifying UI/Styling

- HUD styling: edit the `#hud` and related CSS rules in `<style>`
- Overlay text and buttons: edit HTML in the `<body>` section
- Game canvas appearance: modify the gradient background in `#gameCanvas` CSS

## Browser Compatibility

- Requires HTML5 Canvas, requestAnimationFrame, and Web Audio API support
- Touch events for mobile; mouse events for desktop
- Audio context requires user interaction (click) before sound plays (browser security)
- Tested on modern Chrome, Firefox, Safari, Edge

## Git Workflow

- Branch: `claude/inspiring-planck-oau686` is the active development branch
- Main branch: `main`
- Commit messages should describe gameplay changes or bug fixes clearly
