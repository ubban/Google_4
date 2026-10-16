# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:
- A player-controlled aircraft that shoots enemies and bosses
- Multiple difficulty levels (Easy, Normal, Hard)
- 6 stages with different enemy waves and boss encounters
- 6 boss variants with unique visual designs
- A combo/chain system for score multipliers
- Weapon upgrades (Vulcan and Laser)
- Smart bombs and a health/lives system
- Procedurally generated visual effects and particle systems
- WebAudio synthesis for all sound effects (no external audio assets)
- Responsive design supporting keyboard, mouse, and touch controls
- Multiple visual themes with different sky/color schemes

## Code Architecture

### Single File Structure (`index.html`)
The entire game is contained in one HTML file (~1700 lines) with:
- **HTML Section** (lines 330-420): DOM elements for canvas, HUD, and UI controls
- **CSS Section** (lines 10-320): Styling for responsive layout, animations, and visual effects
- **JavaScript Section** (lines 423-1669): All game logic

### Core JavaScript Organization

#### Global Game State (lines 863-869)
```javascript
let player, bullets, enemies, enemyBullets, particles, powerUps, boss;
let score, lives, maxLives, running, paused, gameOver;
let bombCount, maxBombs, multiplier, chain, chainTimer;
```

#### Configuration Constants (lines 485-515)
- **DIFFICULTIES**: Three difficulty presets with spawn rates, enemy speed, lives, bombs, and boss HP multipliers
- **ENEMY_TYPES**: Three enemy variants (grunt, interceptor, tank) with properties
- **BOSSES**: Six boss variants with color schemes and HP multipliers
- **THEMES**: Six visual themes with sky gradients and color palettes
- **KILLS_PER_STAGE**: 18 kills to trigger stage/boss progression

#### Key Game Systems

1. **Player Movement & Weapon System**
   - Player object with position, speed, cooldown, rapid-fire counter, weapon type, level, and invulnerability timer
   - Mouse steering (when cursor over canvas, plane glides toward pointer)
   - Keyboard/gamepad input handling (ArrowKeys or WASD for movement, Space to fire)
   - Touch/joystick input with virtual joystick on mobile
   - Two weapon types: Vulcan (rapid bullets) and Laser (faster projectiles)
   - Weapon upgrades tracked by player.level (max 3)

2. **Enemy Spawn System** (`spawnEnemy()` at line 1022)
   - Spawns from top of screen at random x-position
   - Three enemy types with different speeds, HP, and behaviors
   - Enemies shoot at player based on difficulty (higher spawn rates on harder difficulties)
   - Enemies always move downward; interceptors "weave" for variety
   - Spawn rate decreases as difficulty increases (faster spawning)

3. **Boss System** (`updateBoss()` at line 1411)
   - Triggered after KILLS_PER_STAGE (18 kills) per stage
   - Bosses spawn at top-center with a stage banner announcement
   - Boss behavior controlled by telegraph counter (0-180 frames between attack patterns)
   - Boss patterns include:
     - Spiraling shot patterns
     - Bullet waves from sides
     - Rapid-fire spreads
   - Boss colors and visual design rotate through BOSSES array
   - Boss HP scales by difficulty multiplier

4. **Collision Detection**
   - Player bullets vs. enemies and boss: AABB (axis-aligned bounding box)
   - Enemy bullets vs. player: Circle collision from player center
   - All collision checks use width/height properties

5. **Scoring & Combo System**
   - Base score from enemy kills (10-35 points depending on type)
   - Combo counter increments on consecutive kills within 1.5 seconds (chainTimer)
   - Multiplier increases at combo milestones (5, 10, 15, 20+ hits)
   - Combo resets if player takes damage

6. **Rendering Pipeline**
   - Background: Gradient sky that changes per stage (cycling through THEMES)
   - Entities drawn in order: enemies, bosses, particles, bullets, player
   - HUD overlays: Score, Lives, Bombs, Weapon, Multiplier, Boss health bar
   - Visual effects: Screen shake, flash on damage, vignette when low health

7. **Audio System** (lines 545-622)
   - WebAudio API synthesis (no external sound files)
   - `beep()` function for tone generation with frequency, duration, wave type
   - `noiseBurst()` for percussion-like sounds
   - sfx object contains all sound effects: weaponFire, laserFire, hit, bossHit, explosion variants
   - Master gain controls volume; muted flag disables audio

8. **Particle System** (`spawnExplosion()` at line 1073)
   - Creates 8-12 particles on explosion
   - Particles have position, velocity, lifetime, and color
   - Used for visual feedback on enemy death and bomb blasts

### Main Game Loop (`update()` at line 1148)
Called every frame (using `requestAnimationFrame`):
1. Update player position from input (keyboard, mouse, or joystick)
2. Fire bullets when cooldown reaches 0
3. Update all bullets (move, check collisions with enemies/boss)
4. Update all enemies (move, fire, check collisions with player)
5. Update boss if active (fire patterns, take damage, die)
6. Update particles (move, fade, remove when dead)
7. Update combo/chain counter
8. Spawn new enemies based on spawn timer
9. Handle stage progression and boss triggers
10. Render everything to canvas
11. Update HUD displays

## Making Changes

### To Modify Game Balance
- **Difficulty settings**: Edit `DIFFICULTIES` object (line 485)
- **Enemy properties**: Edit `ENEMY_TYPES` object (line 502)
- **Boss properties**: Edit `BOSSES` array (line 508)
- **Spawn rates**: Adjust `KILLS_PER_STAGE` constant (line 491)

### To Add/Modify Game Features
- **Player mechanics**: Modify `resetGame()` (line 875) and `update()` collision sections
- **Enemy behavior**: Edit `spawnEnemy()` (line 1022) and enemy update logic in `update()` (line 1148)
- **Boss patterns**: Modify `updateBoss()` (line 1411) to change attack sequences
- **Visual effects**: Adjust `spawnExplosion()` particle counts/colors (line 1073)
- **Sound effects**: Modify functions in `sfx` object (line 624) or add new ones using `beep()/noiseBurst()`
- **Visual themes**: Add entries to `THEMES` array (line 493)

### To Test Changes
Since this is a single-file HTML game, testing is straightforward:
1. Open `index.html` in a web browser
2. Play through different difficulty levels to verify balance
3. Use browser DevTools (F12) to inspect canvas and debug collisions
4. Test on multiple screen sizes and devices (mobile/tablet)
5. Check audio behavior with muted flag toggled
6. Verify touch/joystick controls work on mobile devices

### Performance Considerations
- All rendering happens on a single canvas (480x720 internal resolution, scaled to fit screen)
- Enemy/bullet count grows with stage progression; performance should remain smooth up to ~200 entities
- Particle effects are removed when dead to prevent memory leaks
- LocalStorage is used for persistent high score, difficulty, mute, and motion preference

## Input Controls

**Keyboard:**
- Arrow keys or WASD: Move player
- Space: Fire weapon
- X or Shift: Drop smart bomb
- P or Escape: Pause/resume
- Number keys 1-3: Switch difficulty (outside gameplay)

**Mouse:**
- Move cursor over canvas: Player glides toward pointer (keyboard/joystick input takes priority)
- Click to fire
- No bomb action from mouse

**Touch/Mobile:**
- Left side (move zone): Virtual joystick for movement
- Right side (fire button): Hold to fire
- Bottom center: Bomb button
- Automatically enabled on touch devices (`isTouchDevice` check at line 778)

## Browser Compatibility Notes
- Requires HTML5 Canvas and WebAudio API support
- Uses ES6 features (arrow functions, let/const, template literals)
- Touch events handled for mobile/tablet compatibility
- SafeArea insets respected for notch-friendly layout on mobile browsers

## Game Flow
1. **Init** → Load saved settings from localStorage
2. **Menu** → Player selects difficulty
3. **Game Loop** → Spawn enemies, update player/enemies, check collisions, render
4. **Stage Progression** → After 18 kills, trigger boss battle
5. **Boss Battle** → Boss spawns, shows health bar, uses attack patterns
6. **Next Stage** → Boss defeated, increment stage, repeat
7. **Game Over** → Lives reach 0, show final score, save to high score if beaten
8. **Retry** → Player can restart from menu

## Visual Theme System
The game cycles through 6 visual themes (one per stage):
1. Daylight Front (blues)
2. Sunset Corridor (purples/oranges)
3. Night Approach (dark blues)
4. Storm Front (greens/grays)
5. Arctic Wastes (bright blues)
6. Volcanic Ridge (reds/oranges)

Each theme has a sky gradient, cloud color, and sun color for parallax effect.

## Accessibility Features
- Reduced motion preference support (toggleable via motion button)
- Screen reader announcements via `announce()` function (line 468)
- Live region (`#srLive`) for status updates
- Keyboard shortcuts for all functions
- High contrast colors for HUD elements
- Icon buttons with title attributes for tooltips
