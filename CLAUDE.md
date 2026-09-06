# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 game — a Raiden-style vertical shoot-'em-up game. The entire implementation lives in `index.html` (1669 lines). No build tools, package managers, or external dependencies are required.

## Quick Start

1. **Run the game**: Open `index.html` in a web browser
   - Direct file open: `file://` URLs work fine
   - Or serve via: `python3 -m http.server` (or any static server)
   
2. **Make changes**: Edit `index.html` directly, save, and refresh the browser

## Architecture & Code Organization

The game is structured as a single HTML file with embedded CSS and JavaScript:

### HTML Structure (Lines 1-100)
- Game canvas container (`#gameCanvas`, `#container`)
- HUD overlay displaying score, health, multiplier, weapon level
- Control buttons: mute, motion toggle, pause
- Difficulty selector (Easy/Normal/Hard)
- Overlay system for menus and end-game screens

### CSS Styling (Lines 10-800+)
- Root styling sets dark theme (`#05122b` background)
- Gradient sky background for game canvas
- Responsive design using `clamp()` for responsive fonts and gaps
- Visual effects: screen shake, pulse animations for impacts
- Accessibility features included (reduced motion support via `prefers-reduced-motion`)

### JavaScript Game Engine (Lines ~850-1669)

**Core Architecture**:
- Single IIFE (Immediately Invoked Function Expression) encapsulates all game logic
- Canvas 2D rendering context for all graphics
- Main game loop using `requestAnimationFrame`
- Delta time (`dt`) for frame-rate-independent movement

**Key Game Objects**:

1. **Player** (`player`)
   - Position, velocity, health
   - Current weapon and weapon level
   - Methods: `shoot()`, `takeDamage()`, `heal()`, `setWeapon()`

2. **Weapon Systems** (`weapons` object)
   - `vulcan`: Rapid fire, spread pattern
   - `laser`: Single focused beam
   - `missile`: Homing projectiles with guidance logic
   - Each weapon has fire rate, damage, and spread characteristics
   - Weapon upgrades increase level and modify behavior

3. **Enemies** (`enemies` array)
   - Multiple enemy types with different behaviors
   - Pathfinding logic (sine wave patterns, spirals, etc.)
   - Drop items when defeated (weapons, health, bombs, multiplier)
   - Stage-based spawning with difficulty scaling

4. **Projectiles** (`projectiles` array)
   - Player shots and enemy shots
   - Collision detection against enemies/player
   - Homing missile logic calculates trajectory toward target

5. **Particles** (`particles` array)
   - Visual effects: explosions, impacts, screen shake
   - Fade-out animations

6. **Stage System** (`stages` array)
   - Pre-defined boss encounters per stage
   - Progressive difficulty scaling
   - Boss patterns with distinct attack patterns

7. **Game State** (`gameState`)
   - `idle`: Menu screen
   - `playing`: Active gameplay
   - `paused`: Paused
   - `gameOver`: End screen
   - `boss`: Boss encounter

**Data Management**:
- LocalStorage for high scores (`best`, `volume`, `motionEnabled`)
- Multiplier system (affects score rewards)
- Bomb mechanic (limited smart bombs that clear screen)

## Key Implementation Details

### Collision Detection
- Axis-aligned bounding box (AABB) collision using distance calculations
- Separate systems for: player-projectile, enemy-projectile, player-item

### Rendering Pipeline
1. Clear canvas with gradient background
2. Draw stars/parallax background
3. Draw game entities (enemies, player, projectiles, particles)
4. Draw HUD overlay
5. Apply screen shake effect via canvas transform

### Input Handling
- Mouse/touch for aim and shoot
- Arrow keys or WASD for movement
- Spacebar to pause
- Buttons for mute, motion toggle, difficulty selection

### Performance Considerations
- Pooling pattern for projectiles (reuse objects rather than creating new ones)
- Particle limit to prevent lag
- Request animation frame throttling
- Mobile-optimized touch controls

## Common Development Tasks

### Adding a New Enemy Type
1. Define enemy stats in the enemy spawning section
2. Implement pathfinding logic (update position based on `dt`)
3. Add drop table (what items it gives)
4. Add visual rendering in the main draw loop

### Adding a New Weapon
1. Add weapon definition to `weapons` object with: `rate`, `damage`, `spread`, `speed`
2. Implement weapon's `fire()` method to create projectiles
3. Add weapon level behaviors (how upgrades change the weapon)
4. Register in weapon selection UI

### Adjusting Game Balance
- Enemy spawn rates and patterns: Located in `stages` array
- Weapon damage/fire rate: In `weapons` object
- Player health/movement speed: In `player` object initialization
- Difficulty multipliers: Scale damage and spawn rate based on `difficulty`

### Adding Visual Effects
- Particle system handles most effects
- Screen shake via canvas transform
- Pulse/flash effects in CSS animations
- Reduced motion support: Check `motionEnabled` flag before adding effects

## Browser Compatibility

- Requires Canvas 2D API support
- LocalStorage for persistence (graceful fallback if unavailable)
- Touch support for mobile play
- Tested on modern browsers (Chrome, Firefox, Safari, Edge)

## Testing the Game

1. **Manual testing**: Play through each difficulty and stage
2. **Edge cases to verify**:
   - Weapon upgrades near max level
   - Health/bomb pickup edge cases
   - High score persistence across sessions
   - Pause during boss fights
   - Mute toggle functionality
   - Reduced motion mode (no shakes/flashes)
3. **Mobile testing**: Test touch controls and screen scaling

## Notes for Future Development

- The single-file architecture keeps dependencies minimal but makes the file large — consider splitting if complexity grows
- No external assets required (all graphics drawn via Canvas)
- Sound effects and music could be added via Web Audio API
- Leaderboard/multiplayer would require a backend
