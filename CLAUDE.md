# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 Canvas application. The entire game logic, UI, and styling is contained in `index.html` (1669 lines).

The game features:
- Multiple difficulty levels (Easy, Normal, Hard) with varying enemy spawn rates and behavior
- Weapon system with progressive upgrades (Vulcan, Rocket, Spreader, Laser)
- Smart bomb mechanic
- Boss encounters with health bars
- Score multiplier system based on rapid consecutive hits
- Multiple themed environments with different visual styles
- Touch controls for mobile, keyboard/mouse for desktop
- Audio effects (with mute option) and visual feedback (screen shake, flash, vignette effects)
- Adaptive performance scaling based on frame times

## Running the Game

The game is a static HTML file and requires no build process:

1. **Open in browser**: Simply open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
2. **Local server (optional)**: For consistent behavior, serve via HTTP:
   ```bash
   # Using Python 3
   python -m http.server 8000
   # Then visit http://localhost:8000
   
   # Using Node.js (if available)
   npx http-server
   ```

3. **Deploy**: Copy `index.html` to any static hosting service

## Game Architecture

### Single-File Structure
The entire game is self-contained in `index.html` with three embedded sections:

1. **CSS (lines 10-359)**: Styling for canvas, HUD, UI overlays, animations, and responsive layout
2. **HTML (lines 361-420)**: DOM structure with canvas element, HUD displays, buttons, and touch controls
3. **JavaScript (lines 422-1669)**: Complete game logic inside an IIFE

### Key Game Systems (JavaScript)

**State & Persistence**:
- `STORAGE_BEST`, `STORAGE_MUTE`, `STORAGE_DIFF`, `STORAGE_REDUCED`: LocalStorage keys for saving player preferences and best score
- `running`, `paused`, `gameOver`: Game state flags

**Entity System**:
- `player`: Object tracking player position, health, weapons, bombs, score
- `enemies`: Array of active enemies
- `bosses`: Array of active bosses
- `bullets`: Array of projectiles from player and enemies
- `particles`: Visual effect particles (explosions, pickups)
- `pickups`: Power-ups (life, bomb, weapon upgrades)

**Difficulty System** (`DIFFICULTIES` object):
- `easy`, `normal`, `hard`: Presets for spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- `KILLS_PER_STAGE`: Number of enemy kills required to advance to next stage (18)

**Rendering System**:
- `draw()`: Called each frame to render canvas
- Canvas 2D context used throughout (no WebGL or external libraries)
- Rendering order: background → bosses → player → enemies → bullets → particles

**Update Loop**:
- `update(dt)`: Called each frame with delta time (capped at 50ms)
- `loop(now)`: RequestAnimationFrame callback that manages frame timing and calls update/draw

**Enemy & Boss Behavior**:
- Enemies spawn at top, move downward with random X velocity
- Boss appears when stage kills reach threshold
- Enemies and bosses shoot bullets with probability based on difficulty

**Combat System**:
- Weapons: Vulcan (rapid fire), Rocket, Spreader, Laser
- Each weapon has a cooldown between shots
- Collision detection for bullets hitting enemies/bosses
- Score multiplier increases for consecutive hits, resets on miss

**Input Handling**:
- Keyboard: Arrow keys/WASD for movement, Space/X/Shift for fire/bomb, P/Esc for pause
- Mouse: Move to steer, click to fire
- Touch: Drag on left side to move, tap FIRE/BOMB buttons
- Joystick visualization for touch controls

**Visual Effects**:
- `shakeTime` & `shakeAmount`: Screen shake (triggered on player damage)
- Vignette effect: Red pulse when player is critical (1 HP)
- Flash effects: White flash for hits, larger flash for bombs
- Particles: Explosions, pickup pickups, dust effects

**Accessibility**:
- Reduced motion support via `prefers-reduced-motion` media query
- Screen reader announcements via `announce()` function
- High contrast colors for readability

### Theme System
The `THEMES` array (lines 493-499) defines 6 visual themes with sky gradients, cloud colors, and sun colors:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- Themes rotate through stages

### Performance
- Adaptive quality system (`quality` variable) scales particle count and effects based on frame times
- Frame time tracking via `trackPerf()` to detect performance issues
- Quality levels: 0 (full), 1 (medium), 2 (low)

## Common Development Tasks

### Adding a New Weapon
1. Locate `WEAPONS` object (around line 600)
2. Add new weapon definition with `cooldown`, `damage`, `bullet` properties
3. Update weapon cycling logic in input handler to include new weapon
4. Add weapon sprites/visuals in the `drawPlayer()` function

### Adjusting Difficulty
1. Modify `DIFFICULTIES` object to change spawn rates, speeds, or HP multipliers
2. Test with different difficulty settings via the difficulty buttons in overlay

### Adding Visual Effects
1. Create particle effects in `particles` array during game events
2. Modify `drawParticles()` to render new particle types
3. Trigger screen shake via `shakeTime = value`
4. Add flash effects via `flashEl.classList.add('hit')` or `'bomb'`

### Modifying Game Balance
- Enemy spawn rates: `DIFFICULTIES[difficulty].spawnBase` and `spawnMin`
- Player damage: Modify individual weapon `damage` values
- Boss health: Adjust `boss.hp` calculation with `bossHpMul` multiplier
- Lives/bombs: Adjust `DIFFICULTIES[difficulty].lives` and `bombs`

### Browser Compatibility
- Tested on modern browsers (Chrome, Firefox, Safari, Edge)
- Uses HTML5 Canvas 2D context (widely supported)
- LocalStorage for persistence
- RequestAnimationFrame for frame updates
- CSS Grid/Flexbox for responsive UI

## Testing Checklist

- **Gameplay**: Verify player movement, firing, bombs all work on desktop (mouse/keyboard) and mobile (touch)
- **Difficulty levels**: Test Easy/Normal/Hard and verify enemy behavior changes
- **Audio**: Test mute/unmute toggle and sound playback
- **Visual effects**: Screen shake, vignette, flash effects render correctly
- **Performance**: No frame drops on different devices (use Chrome DevTools Performance tab)
- **Accessibility**: Test reduced motion toggle, screen reader announcements
- **Saves**: Verify best score and preferences persist after page reload
