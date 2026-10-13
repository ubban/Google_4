# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML game. It runs directly in the browser with no build process, external dependencies, or package managers. The game is playable immediately by opening `index.html` in any modern browser.

## Running & Testing

### Play the Game
Open `index.html` directly in a web browser. The game loads immediately and is fully playable. Use:
- **Mouse/Trackpad**: Move the crosshair to aim
- **Keyboard**: Arrow keys or WASD to move
- **Mobile/Touch**: Virtual joystick appears on touch devices

### Testing Locally
```bash
# Option 1: Open directly (simplest)
open index.html

# Option 2: Use a local server (recommended for network features)
python3 -m http.server 8000
# Then visit http://localhost:8000
```

## Architecture & Structure

The entire game is implemented in a single `index.html` file (~1670 lines) with inline JavaScript and CSS. Key architectural sections:

### HTML Structure
- **Game Container**: Main canvas element (`#gameCanvas`) and overlay HUD elements
- **HUD Elements**: Score display, weapon indicator, heart/bomb counters, boss health bar
- **Visual Effects**: Vignette overlay (damage indicator), flash effects, combo popups, stage banners

### Core Game Systems

#### 1. **Game Loop & Rendering** (lines ~250-400)
- Single `requestAnimationFrame` loop driving all game updates
- Canvas-based 2D rendering with custom drawing functions
- Game state managed globally (playerPlane, enemies, bosses, projectiles, etc.)
- FPS tracking and performance monitoring built-in

#### 2. **Player System** (playerPlane object)
- **Position & Movement**: Constrained to playable area; smooth movement with acceleration
- **Weapon System**: 5 different weapons (spread, gatling, laser, homing, plasma)
  - Weapon-switching cycles through types
  - Fire rate and projectile behavior varies per weapon
  - Weapon upgrades via power-ups increase fire rate/effectiveness
- **Health & Invulnerability**: Multiple lives (hearts), temporary invulnerability after hit
- **Bombs**: Limited smart bombs that clear large enemy clusters with visual effects

#### 3. **Enemy System**
- **Enemy Types** (ENEMY_TYPES object, line ~540):
  - `grunt`: Fast, weak, yellow-colored
  - `interceptor`: Medium speed/health, blue-colored, higher scoring
  - `tank`: Slow, heavily armored, high-value target
- **Spawning Logic** (`spawnEnemy()`): Randomized spawning from top of screen based on difficulty
- **Enemy Behavior**: Linear downward movement with no AI; collision damage and point values
- **Boss System** (`spawnBoss()`): Large themed ship with 3-phase health bar, spawns at wave 5+

#### 4. **Combat & Collision** 
- **Collision Detection**: `rectsOverlap()` function for AABB rectangle-based hitboxes
- **Projectile System**: Pooled bullets fired by player and bosses
- **Damage & Explosions**: Hit detection triggers visual explosions and screen shake
- **Score Multiplier**: Chain system - continuous hits increase combo multiplier, broken on death

#### 5. **Audio System**
- **Web Audio API**: Procedurally generated sound effects (beeps, noise bursts)
- **Sound Effects** (sfx object): Different tones for weapon fire, powerups, explosions
- **Explosion Variants**: Different pitch/tone per enemy type
- **Global Mute Toggle**: Audio can be disabled via settings

#### 6. **Difficulty & Progression**
- **Difficulty Levels** (DIFFICULTIES object): Easy, Normal, Hard, Insane
  - Controls enemy spawn rates, health, movement speed
  - Can be changed mid-game via difficulty selector
- **Wave-Based Progression**: Enemies spawn in increasing difficulty over time
- **Boss Waves**: Bosses appear at wave 5 and every 2-3 waves after

#### 7. **Input Handling**
- **Keyboard**: Arrow keys or WASD mapped to directions
- **Mouse/Touch**: Click/tap anywhere to set target position
- **Virtual Joystick**: Auto-appears on mobile; drag-based movement with visual feedback
- **Accessibility**: Touch-action CSS prevents browser interference on mobile

#### 8. **UI & Effects**
- **HUD Stats**: Real-time display of lives, bombs, weapon type, score, combo
- **Visual Feedback**: Screen shake on hit, flash overlay on hit/bomb, vignette flash on damage
- **Combo Notifications**: Large animated text popup showing current combo
- **Stage Banners**: Announce boss/stage transitions with smooth animation
- **Boss Bar**: Health indicator for active boss (separate colored bar)

#### 9. **Mobile & Responsive Design**
- **Viewport Meta Tags**: Disables zoom, maximizes display area (`viewport-fit=cover`)
- **CSS Clamp**: Responsive font sizing (scales with viewport size)
- **Safe Area Insets**: Respects notched phones with `env(safe-area-inset-*)`
- **Touch Handling**: `touch-action: none` prevents scroll interference; `pointer-events` managed per element

## Key Constants & Configurations

- **DIFFICULTIES**: Object mapping difficulty names to spawn rates, health multipliers, speed factors
- **ENEMY_TYPES**: Defines each enemy's color, health, points, speed
- **sfx**: Sound effect definitions (frequency, duration, waveform type)
- **Canvas Size**: Defaults to 800x600 but responsive (`window.innerWidth/Height`)
- **Game Constants** (top of script): Wave size, powerup rates, weapon specs, score multipliers

## Common Development Tasks

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with name, color, health, score, speed
2. Update `spawnEnemy()` if special spawn logic needed
3. Add explosion SFX to `EXPLODE_SFX` object if desired
4. Test collision/damage in game

### Modify Weapon Behavior
- Edit weapon specs in the weapon-firing section (around line 1100)
- Adjust projectile speed, fire rate, pattern (spread angle, etc.)
- Fire rate increased by weapon upgrades (`playerPlane.weaponLevel`)

### Tweak Difficulty
- Modify `DIFFICULTIES` object constants
- Test by selecting difficulty from in-game menu
- Affects enemy spawn rate, health, and movement speed

### Change Game Balance
- Score multipliers: Search for `addScore()` calls; adjust base point values
- Enemy health: Modify `ENEMY_TYPES` health values
- Player health/invulnerability: Edit `playerPlane.hearts` and invuln timing

### Add Visual Effects
- New explosions: Call `spawnExplosion(x, y, color, count)` with custom colors
- Screen effects: `screenShake(magnitude, duration)` or modify CSS animations
- Particle trails: Add to the main rendering loop (line ~300)

## Important Implementation Notes

1. **No External Dependencies**: Game is 100% vanilla JavaScript—no libraries or build tools
2. **Single Canvas Element**: All rendering is immediate-mode 2D canvas drawing
3. **Global Game State**: All game objects (playerPlane, enemies, bosses, etc.) stored in global scope or attached to game object
4. **Audio Context**: Lazily initialized on first user interaction to comply with browser autoplay policies
5. **Performance**: FPS counter built-in (`trackPerf()`); most calculations are O(n) where n = number of active game objects
6. **Mobile-First Responsive**: Uses CSS `clamp()`, viewport meta, safe-area-inset for responsive behavior
7. **Input Unification**: Mouse, touch, and keyboard inputs all map to the same target-following system

## Testing Checklist for Changes

- [ ] Game loads and renders correctly in browser
- [ ] Player can move and fire weapons
- [ ] Enemies spawn, move, and take damage correctly
- [ ] Collision detection works (player hits enemy, projectile hits enemy)
- [ ] Audio plays (if not muted)
- [ ] Score updates and multiplier chains work
- [ ] Bomb mechanic functions (limited uses, clears enemies)
- [ ] Difficulty changing works mid-game
- [ ] Game works on mobile (touch input, responsive sizing)
- [ ] Boss appears at correct wave and behaves correctly
- [ ] No console errors or performance issues (check FPS counter)
