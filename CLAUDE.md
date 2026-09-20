# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file browser-based Raiden-style vertical shoot-'em-up game. The entire game is implemented in `index.html` with inline HTML, CSS, and JavaScript (~1,670 lines).

### Key Features
- Vertical scrolling shooter with wave-based enemies
- Boss encounters at stage transitions
- Weapon upgrade system (bullet types and power levels)
- Power-ups (health, bombs, weapon upgrades)
- Difficulty settings affecting enemy spawn rates and patterns
- Touch controls for mobile devices
- Audio feedback using Web Audio API
- Responsive design with mobile viewport support

## Code Architecture

The game code is organized in a single file with the following major sections:

### 1. **HTML Structure & Styling** (Lines 1-422)
- Canvas-based rendering for the game viewport
- HUD elements: score, health, weapon info, boss health bar, combo notifications
- Mobile touch controls UI (joystick, fire/bomb buttons)
- Overlay menu for start screen, game over, and pause states
- CSS animations for UI effects (heart beat, combo pulse, stage banner)

### 2. **JavaScript Game Loop** (Lines 423+)

The game is wrapped in an IIFE (`(function() { ... })()`) containing:

#### **Core Game Objects**
- `player`: Player ship with position, velocity, health, weapons, bombs
- `bullets`: Array of player bullets with different types (regular, laser)
- `enemies`: Array of active enemies with types (grunt, interceptor, tank)
- `boss`: Current stage boss (if active)
- `powerUps`: Array of dropped power-ups (life, bombs, weapons)
- `particles`: Temporary visual effects (explosions, hits)
- `enemyBullets`: Enemy projectiles

#### **Game State Variables**
- `score`, `bestScore`: Score tracking and persistence
- `multiplier`: Kill combo multiplier (increases with consecutive kills)
- `stage`: Current stage/wave number
- `wave`: Current wave within a stage
- `lives`, `bombs`: Player resources
- `running`, `paused`: Game state flags
- `difficulty`: Current difficulty setting (affects spawn rates)
- `weaponType`, `weaponLvl`: Player weapon configuration

#### **Key Game Constants**
- `DIFFICULTIES`: Defines enemy spawn rates for Easy/Normal/Hard
- `KILLS_PER_STAGE`: Number of enemies per stage before boss appears (18)
- `ENEMY_TYPES`: Define grunt, interceptor, tank with HP, speed, and reward values
- `BOSSES`: Array of boss definitions (each with unique attack patterns)
- `THEMES`: Color schemes for different areas/difficulty

#### **Main Functions**

**Game Loop Functions:**
- `update(dt)`: Updates all game state each frame (player, enemies, bullets, collisions)
- `draw()`: Renders all game objects to canvas
- `loop(now)`: RequestAnimationFrame callback, coordinates update/draw and performance tracking

**Player Management:**
- `resetGame()`: Initialize game state for new game
- `movePlayer(x, y)`: Update player position (bounded to canvas)
- `firePlayerBullet()`: Create player projectiles based on weapon type
- `useBomb()`: Trigger bomb effect (clear bullets, damage bosses, hit effect)

**Enemy Spawning:**
- `spawnWave()`: Generate new enemy wave based on stage/difficulty
- `spawnBoss()`: Create boss encounter for current stage

**Collision Detection:**
- `checkCollisions()`: Main collision loop (enemy bullets vs player, player bullets vs enemies)
- Includes projectile-entity, entity-entity collision logic

**Audio System:**
- `ensureAudio()`: Initialize Web Audio context (deferred until first user interaction)
- `beep()`: Generate tone-based sound effects (accepts frequency, duration, type)
- `noiseBurst()`: Generate noise-based effects (explosions, power-ups)
- `sfx`: Object containing pre-configured sound effect calls

**Input Handling:**
- Desktop: Mouse position tracked for aiming, keyboard (Z for fire, X for bomb, P for pause)
- Mobile: Touch-based joystick and fire/bomb buttons
- `ensureTouchControls()`: Set up mobile input handlers

**UI Rendering:**
- `renderHearts()`, `renderBombs()`: Draw HUD icons
- `flashHit()`, `flashBomb()`: Screen flash effects on events
- `showComboPop()`: Animate combo multiplier text
- `drawBossShip()`: Special rendering for boss entity

## Development Workflow

### Running the Game
- Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge)
- No build process or server required—it's a single static file
- Game works on desktop, tablet, and mobile devices

### Testing
- **Desktop Testing**: Use keyboard (arrow keys or WASD to move, Z to fire, X for bomb)
- **Mobile Testing**: Use browser developer tools device emulation or test on actual device
- **Performance**: Monitor frame rate using DevTools Performance tab; game targets 60 FPS
- **Sound Testing**: Check audio output in different browsers (some require user interaction first)

### Making Changes

**Adding Game Features:**
1. Identify relevant section in the code (enemy types, bosses, weapons, etc.)
2. Modify constants at the top (ENEMY_TYPES, BOSSES, DIFFICULTIES, etc.)
3. Test in browser immediately (no compilation needed)
4. Use DevTools console for debugging (e.g., `console.log()` statements)

**Balancing:**
- `DIFFICULTIES` object controls enemy spawn rates and intervals
- `ENEMY_TYPES` object defines each enemy's health, speed, and kill rewards
- `weaponType` and power levels affect bullet fire rate and damage
- `multiplier` increases with consecutive kills; resets on damage taken

**Visual Changes:**
- CSS animations and styling can be tweaked directly in the `<style>` block
- Canvas rendering code in `draw()` function controls entity appearance
- Colors are defined in THEMES array and used throughout

**Enemy AI:**
- Enemy movement and attack patterns are in the `update()` function
- Boss attack patterns are defined inline in the boss update logic
- Adjust x/y velocity and firing intervals to change behavior

## Key Game Mechanics

### Scoring System
- Base score: 10 × enemy type multiplier
- Multiplier bonus: Score × current multiplier value
- Multiplier increases on consecutive kills; resets on player damage
- Best score persisted to browser localStorage

### Difficulty Levels
- **Easy**: Slower spawn rates, more generous enemy patterns, extra starting health
- **Normal**: Baseline difficulty
- **Hard**: Faster spawns, aggressive bosses, fewer starting resources

### Weapon System
- **Regular bullets**: Standard projectiles, medium fire rate
- **Laser**: Piercing weapon with different visual effect
- Weapon level increases with power-up pickups, affects fire rate and spread
- Weapons reset on new game

### Boss Encounters
- One boss per stage (after 18 kills)
- Boss health displayed in top bar
- Each boss has unique attack pattern and movement
- Boss defeat triggers stage transition and wave reset

## Browser Compatibility

- Requires modern browser with:
  - HTML5 Canvas API
  - RequestAnimationFrame
  - Web Audio API (for sound effects)
  - LocalStorage (for score persistence)
  - Touch Events (for mobile input)

## Common Modifications

**Change enemy spawn rate**: Adjust `DIFFICULTIES[difficulty].spawnInterval`

**Adjust player health**: Modify starting `lives` value in `resetGame()`

**Add new boss**: Add object to `BOSSES` array with `name`, `hp`, and update logic in boss attack section

**Modify weapon behavior**: Change fire rate, spread, or projectile properties in `firePlayerBullet()`

**Tweak colors/themes**: Modify hex color values in CSS and THEMES array
