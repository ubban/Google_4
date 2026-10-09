# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire game is contained in a single `index.html` file (~1700 lines) with embedded HTML, CSS, and JavaScript. It runs entirely in the browser with no build process or external dependencies.

## Project Structure

- **index.html**: The complete game (HTML structure + CSS + JavaScript game engine)
  - No build step required; open directly in a browser or serve via any HTTP server
  - Contains inline styles and a complete game loop implementation

## Code Architecture

The game is organized into logical sections within the single file:

### 1. **HTML Structure** (top ~30 lines)
- Canvas element (`#gameCanvas`) for rendering
- HUD elements for displaying stats (health, bombs, weapon, score multiplier)
- Control overlays (start button, pause, game over screen)
- Hidden divs for vignette effects, flashes, and feedback

### 2. **CSS Styling** (~150 lines)
- Dark theme with dark blue background (`#05122b`, `#0d2f5e`)
- Responsive typography with `clamp()` for mobile scaling
- Animations: heartbeat effect for critical health, vignette pulse when taking damage
- HUD uses semi-transparent pills with backdrop blur
- Gradient background for the game canvas

### 3. **JavaScript Game Engine** (~1500 lines)

#### **Core Data Structures**
- `player`: Object with position, velocity, weapon state, health, bombs
- `bullets`: Array of player projectiles (with `laser` flag for beam weapon)
- `enemies`: Array of active enemy ships
- `boss`: Single boss entity when active
- `enemyBullets`: Array of incoming fire
- `powerUps`: Array of collectibles (life, bomb, weapon upgrades)
- `particles`: Array for explosions and visual effects

#### **Constants & Configuration** (near top of script)
- `CANVAS_W / CANVAS_H`: Fixed game dimensions (usually 800x600)
- `ENEMY_TYPES`: Dictionary defining stats for each enemy type (fighter, tank, fast)
- `WAVES`: Array defining enemy spawn patterns for each stage
- `BOSS_TYPES`: Boss definitions with behavior patterns

#### **Game Loop** (`requestAnimationFrame`)
1. **update(dt)**: Physics, movement, collision, spawn logic (called every frame)
2. **draw()**: Render all entities to canvas (called every frame)
3. **trackPerf(dt)**: Monitor frame time and adjust quality settings

#### **Key Functions**

- **Player Control**
  - `handleMouseMove()`: Aiming the crosshair
  - `handleKeyDown/Up()`: WASD for movement
  - `update()` section: Player position clamping, collision with powerups/enemy fire

- **Weapon System**
  - Player can switch between two weapons (laser and bullet spreads)
  - Different ROF, damage, and bullet patterns
  - Powerups modify weapon level/type

- **Enemy AI**
  - Each enemy type has `move()` and `shoot()` behavior
  - Waves defined in `WAVES` array control spawn timing and enemy patterns
  - Boss has complex behavior with phases

- **Collision Detection**
  - Circle-to-circle or rect-to-circle checks in `update()`
  - Player bullets vs enemies, player vs enemy bullets, player vs powerups

- **Visual Feedback**
  - Screen shake via `shakeTime` offset to canvas transform
  - Red vignette pulse when player takes damage
  - Flash effects (white screen for hits, bomb detonation)
  - Particle effects for explosions

- **Rendering**
  - `draw()`: Clears canvas, applies shake, renders all entities
  - `drawPlane()`: Draws player and enemy ships (triangle + small shapes)
  - `drawBossShip()`: Boss-specific sprite rendering

- **Game States**
  - `running`: Game active vs paused/ended
  - `paused`: Pause toggle
  - `stage`: Current level
  - `finalWave`: Tracks progression through stage waves

#### **Physics & Timing**
- Delta time (`dt`) used for frame-rate independent movement
- Clamped to max 0.05s to prevent large jumps
- Performance monitoring adjusts visual quality (`glow` effects) based on frame time

## Making Common Modifications

### Change Difficulty
- Adjust `WAVES` array to add/remove enemies or change spawn timing
- Modify `ENEMY_TYPES` stats (health, damage, fire rate)
- Alter `BOSS_TYPES` behavior or health values

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `hp`, `speed`, `firerate`, `color`
2. Add `move()` and `shoot()` methods to the enemy type
3. Add sprite rendering in `draw()` if needed
4. Reference in `WAVES` for spawning

### Modify Weapons
- Update weapon object in player creation section
- Change bullet patterns in the weapon's `shoot()` method
- Adjust damage, fire rate, or visual appearance

### Adjust Game Balance
- Player stats: `player.maxHealth`, `player.maxBombs`, starting weapon
- Bullet damage and enemy health values
- Powerup drop rates in enemy `die()` section
- Enemy fire rate and damage

### Change Visual Style
- Modify color constants for player, enemies, UI
- Adjust CSS backdrop blur, animations, HUD styling
- Change canvas gradient background
- Alter particle effects colors

## Running the Game

**Development & Testing:**
```bash
# Serve the file locally
python3 -m http.server 8000
# or
npx http-server

# Open http://localhost:8000 in browser
```

**No build or compilation needed.** Changes to the HTML file are immediately visible by refreshing the browser.

## Performance Considerations

- The quality system monitors frame time and disables glow effects if FPS drops
- Large canvas size increases rendering cost; adjust `CANVAS_W/H` if performance is an issue
- Enemy and particle counts can be tuned in `WAVES` and explosion particle counts
- Mobile devices benefit from viewport scaling via the meta viewport tag

## Game Mechanics Reference

- **Player Movement**: WASD keys, constrained to canvas bounds
- **Aiming**: Mouse/touch position determines crosshair and bullet direction
- **Weapons**: Press Space to cycle (cycles between available types)
- **Bombs**: Smart bombs clear screen and deal damage; limited quantity; regain via powerups
- **Health**: Display via heart icons; lose one per enemy bullet hit
- **Score Multiplier**: Increased by rapid consecutive enemy kills; resets after a pause
- **Powerups**: Spawn from defeated enemies
  - `life`: Restore one health
  - `bomb`: Add one bomb
  - `weapon`: Upgrade weapon level/unlock new weapon type

