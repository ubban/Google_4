# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire game is implemented in a single `index.html` file (~1669 lines) with embedded CSS and JavaScript. No build process, dependencies, or compilation is needed.

## Running the Game

- **Development**: Open `index.html` directly in any modern web browser
- **No server needed**: The game is completely client-side and requires no build tools or HTTP server
- **Responsive**: Works on desktop (keyboard/mouse) and mobile (touch/on-screen controls)

## Game Architecture

### Core Game Loop
The main game loop runs via `requestAnimationFrame` and cycles through:
1. **Input handling**: Keyboard, mouse, and touch events update player state
2. **Update**: Game logic including collision detection, movement, and spawning (line ~1148, `update(dt)`)
3. **Draw**: Render the game state (line ~1520, `draw()`)

### Key Game Systems

**Player** (`player` object):
- Position (`x`, `y`), dimensions (`w`, `h`)
- Health/lives (`health`)
- Invulnerability timer (`invuln`)
- Weapon type and level
- Cooldown between shots
- Bombs (smart bombs) - limited use, can be recharged

**Enemies**:
- Multiple types: fighters, tanks, snipers (each with different AI patterns)
- Spawn with `spawnEnemy()` based on difficulty and stage progress
- Use `weave` property for sine-wave movement patterns
- Shoot at player with either spread-pattern or homing bullets
- Drop power-ups when killed

**Bosses**:
- Enter with `spawnBoss()` after defeating enough stage enemies
- Have their own `updateBoss()` logic (line ~1411)
- Boss-specific attack patterns and movement
- Require multiple hits to defeat

**Combat System**:
- Player fires with Space or on-screen button
- Two weapon types: `vulcan` (rapid fire) and `laser` (piercing shots)
- Enemies fire `enemyBullets` with different patterns
- Collision detection using `rectsOverlap()` for all entities

**Power-ups**:
- Weapons: Increase weapon level and unlock laser
- Health: Restore lives
- Bombs: Add charges to smart bomb
- Spawn from defeated enemies via `maybeSpawnPowerUp()`

**Scoring & Progression**:
- Score multiplier increases with consecutive hits (combo system via `chainTimer`)
- Chain breaks after 2.5 seconds of no damage
- Stages progress: Complete a stage by defeating KILLS_PER_STAGE enemies, then face a boss
- Difficulty automatically ramps by reducing enemy spawn intervals

### Difficulty Settings
Three preset difficulties (`easy`, `normal`, `hard`) with different spawn rates and enemy patterns. Configured in the `difficulties` object around line ~1100.

## Important Code Sections for Modifications

**Game Configuration**:
- `KILLS_PER_STAGE`: Number of enemies to defeat before boss spawns
- `difficulties`: Spawn timing and bullet patterns per difficulty
- `player.health = X`: Starting lives
- `fireDelay`: Weapon cooldown (vulcan: 0.1s, laser: 0.15s)

**Graphics**:
- Plane drawing: `drawPlane()` (line ~913)
- Boss drawing: `drawBossShip()` (line ~970)
- Explosions and effects use particle system with `spawnExplosion()`
- Critical health vignette effect enabled at 1 HP

**Audio/Effects**:
- Web Audio API synth (`beep()`, `noiseBurst()`) for sound effects
- Spatial audio: `panFor()` pans sound left/right based on screen position
- Mutable via UI button; stores preference in localStorage

**Canvas and Rendering**:
- `canvas.width` and `canvas.height` adapt to window size (handled by `resize()`)
- All drawing is done on `ctx` (2D canvas context)
- Background uses gradient sky color
- HUD displays via DOM elements (not canvas)

## Development Notes

- **Single File**: The entire codebase is in `index.html`. While this is not ideal for large projects, it works well for this self-contained game.
- **No Linting/Testing**: This is a game with visual output; test changes by running the game and playing through scenarios.
- **Mobile Support**: Touch input is fully integrated with virtual joystick and on-screen buttons.
- **Accessibility**: Includes screen reader support with `aria-live` region for announcements.
- **Performance**: The file is large but single-threaded. Framerate is displayed when developer tools are open (shows FPS if profiling is enabled).

## Common Tasks

**Add a new enemy type**: Look at existing enemy definitions in `spawnEnemy()` (~line 1055); add new type with its `pattern`, `speed`, `canShoot`, and graphics.

**Adjust difficulty**: Modify the `difficulties` object to change spawn intervals (`spawnBase`, `spawnMin`) and bullet patterns for each difficulty level.

**Modify weapon behavior**: Edit `fireWeapon()` (~line 1230) to change weapon mechanics, projectile speed, or add new weapon types.

**Add visual effects**: Extend `spawnExplosion()` or create new particle systems; adjust vignette/flash behavior for hit feedback.

**Change game balance**: Adjust `player.health`, power-up spawn rates in `maybeSpawnPowerUp()`, score multipliers via `chainTimer`, or enemy health/damage values.

## Browser Compatibility

Requires:
- ES6+ JavaScript support
- Canvas 2D API
- Web Audio API
- Touch Events API
- LocalStorage (for preferences)
