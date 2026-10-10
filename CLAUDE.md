# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application. It features player aircraft combat, multiple difficulty levels, stage progression, boss battles, weapon power-ups, and a scoring system.

## Quick Start

- **Open the game**: Simply open `index.html` in a web browser
- **No build required**: The entire game is self-contained in one file with embedded CSS and JavaScript
- **Development**: Edit `index.html` directly; changes apply immediately on refresh

## Game Architecture

### High-Level Structure

The game is implemented as a single IIFE (Immediately Invoked Function Expression) that encapsulates all game state and logic. Key components:

1. **State Management**: Game variables tracking player, enemies, bosses, bullets, scores, lives, stage, etc.
2. **Game Loop**: `loop()` function runs via `requestAnimationFrame`, managing update and render cycles
3. **Rendering**: `draw()` function renders all game elements to canvas using 2D context
4. **Update Logic**: `update(dt)` processes physics, collisions, enemy spawning, and game logic

### Core Systems

#### Player & Movement
- `player` object: position, dimensions, health, invulnerability frames, current weapon
- Keyboard input: Arrow keys / WASD for movement, Space to fire, X/Shift for bombs
- Mouse input: Move cursor to steer, click to fire
- Touch input: Drag left area to move, tap right area buttons to fire/bomb

#### Weapons System
- Four weapon types: VULCAN, RIFLE, LASER, MULTI
- Each weapon has base level and upgradeable levels (1-3+)
- Weapon switching via power-ups
- `fireWeapon()` handles bullet spawning based on current weapon type
- Player bullets stored in `bullets[]` array

#### Enemy System
- `enemies[]` array stores all active enemies
- Enemy types defined in `ENEMY_TYPES` with properties (spawn rate, speed, fire chance, HP, behavior)
- Enemies spawn based on `spawnTimer` and difficulty settings
- `spawnEnemy()` creates new enemy with random type and position
- Enemy bullets in `enemyBullets[]` array with different behavior (radial patterns vs. aimed)

#### Boss System
- `boss` object spawned after defeating 18 enemies per stage
- Six unique boss designs in `BOSSES[]` array: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Boss patterns defined in `BOSS_PATTERNS[]`—each boss has 3 attack patterns that cycle
- `updateBoss(dt)` handles boss movement and attack timing with telegraph phase
- `killBoss()` triggers stage progression
- Boss health bar UI at top of screen

#### Difficulty System
- Three preset difficulties: easy, normal, hard
- Controlled by `DIFFICULTIES` object defining spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- Player can select difficulty before each game starts
- Settings persist in localStorage

#### Scoring & Multiplier
- Score increases with enemy kills and boss defeats
- Multiplier (`combo`) increases with consecutive kills; breaks on taking damage
- Multiplier displays on HUD and triggers visual combo popup at x2, x4, x6, etc.
- Best score persists in localStorage

#### Power-Ups
- `powerUps[]` array stores floating collectibles
- Types: weapon upgrades, extra life, bomb refill
- `maybeSpawnPowerUp(x, y)` spawns after certain enemy kills
- `applyPowerUp(type)` applies effect when collected

#### Collision Detection
- `rectsOverlap(a, b)` checks axis-aligned bounding box collisions
- Player-bullet: award points and remove enemy
- Enemy-bullet: reduce player health, trigger invulnerability
- Power-up collection: apply benefit

#### Visual Effects
- Screen shake via `screenShake(magnitude, duration)`
- Flash effects (white hit flash, bomb flash) via CSS animations
- Particle explosions via `spawnExplosion(x, y, color, count)`
- Combo pop text display
- Stage banner announcement
- Health/bomb indicators on HUD

#### Audio System
- Web Audio API for sound effects
- Lazy-initialized audio context via `ensureAudio()`
- Oscillator-based beeps with frequency sweep, filters, panning
- Noise burst for explosions
- Mute state persists in localStorage

#### Themes & Visuals
- Six themed stage backgrounds in `THEMES[]` with sky gradient, cloud colors, sun effects
- Stage theme cycles as difficulty increases
- Cloud parallax effect at two depths
- Smooth transitions between stages

### Key Data Structures

```javascript
// Player
player = { x, y, w, h, weapon, weaponLvl, hp, invuln, score, combo }

// Enemy
{ x, y, w, h, type, color, dark, hp, shootTimer }

// Boss
{ x, y, w, h, design, hp, maxHp, vx, entering, telegraph, attackTimer, patternIndex }

// Bullet (player)
{ x, y, w, h, laser, damage }

// Bullet (enemy)
{ x, y, w, h, radial, dx, dy }

// Power-up
{ x, y, w, h, type, vx, vy, life }

// Particle
{ x, y, color, dx, dy, life, maxLife }
```

## Important Files & Sections

- **Lines 1–359**: HTML structure and CSS styling
- **Lines 422–467**: DOM element caching and storage key constants
- **Lines 485–517**: Configuration constants (DIFFICULTIES, THEMES, ENEMY_TYPES, BOSSES)
- **Lines 529–727**: UI and feedback functions (buttons, HUD updates, screen shake, combo pop)
- **Lines 785–840**: Input handling (keyboard, mouse, touch, joystick)
- **Lines 875–912**: Game reset and initialization
- **Lines 913–1021**: Rendering functions for player, enemies, bosses, clouds
- **Lines 1022–1087**: Enemy spawning, collisions, explosions, power-ups
- **Lines 1148–1304**: Main update loop, physics, damage, status effects
- **Lines 1324–1409**: Boss attack patterns (radial, aimed, spiral, homing, cross, etc.)
- **Lines 1411–1468**: Boss update, boss death, player death, game over
- **Lines 1507–1639**: Canvas rendering (background, sky, clouds, entities)
- **Lines 1640–1666**: Main game loop and event listeners

## Common Development Tasks

### Adding a New Enemy Type
1. Define new type in `ENEMY_TYPES` object (around line 502)
2. Set properties: `hp`, `width`, `height`, `color`, `shootChance`, `rarity`
3. Adjust `spawnEnemy()` logic if special behavior needed
4. Rendering handled by generic `drawPlane()` if using same visual style

### Adding a New Boss
1. Add to `BOSSES[]` array (around line 508)
2. Add corresponding attack pattern array to `BOSS_PATTERNS[]` (around line 1372)
3. Each pattern is a function that calls fire functions like `fireRadial()`, `fireAimedSpread()`, etc.
4. Boss health scales with difficulty via `bossHpMul` in DIFFICULTIES

### Adding a New Weapon
1. Update weapon data in `fireWeapon()` function (around line 1124)
2. Extend weapon label logic in `weaponLabel()` (around line 871)
3. Add firing logic that pushes bullets to `bullets[]` array with appropriate `laser` flag

### Adjusting Game Balance
- **Enemy difficulty**: Modify `DIFFICULTIES` object (spawn rates, speeds, shoot chances)
- **Enemy count for boss**: `KILLS_PER_STAGE = 18` at line 491
- **Power-up frequency**: Adjust probability in `maybeSpawnPowerUp()` (around line 1058)
- **Weapon damage**: Modify bullet creation in `fireWeapon()` or collision handling
- **Boss patterns**: Edit or add new patterns in `BOSS_PATTERNS[]`

### Performance Optimization
- Adaptive quality system tracks frame times in `trackPerf(dt)` (line 476)
- Lowers visual effects (glow, shadows) when FPS drops
- Check `quality` variable: 0 = full quality, 1 = medium, 2 = low
- Most expensive operations: particle effects, screen shake, shadow rendering

### Accessibility
- Reduced motion preference checked via `prefers-reduced-motion` media query
- Toggle via motion button disables all CSS animations
- Screen reader announcements via `announce()` function
- HUD text uses high contrast colors

## Development Tips

- **Canvas rendering**: All visuals drawn to `#gameCanvas` context in `draw()` function
- **State persistence**: Use `localStorage` with STORAGE_* keys for scores, settings
- **Audio context**: Must be initialized after user interaction (`ensureAudio()`)
- **Collision math**: Simple AABB overlap check; positions use top-left for rectangles, center for circles
- **Entity arrays**: Always iterate safely; enemies/bullets can be removed during iteration
- **Frame timing**: `dt` (delta time) passed to update functions for frame-rate-independent physics
- **Touch vs. mouse**: Game auto-detects input method and shows appropriate UI
- **Canvas sizing**: Responsive via `resize()` function; maintains aspect ratio

## Testing the Game

- **Full playthrough**: Play through all difficulties and reach different stages
- **Weapon cycling**: Verify all four weapons fire correctly and level up
- **Boss patterns**: Confirm each boss uses correct attack pattern set
- **Collision**: Test enemy hits reduce health, bullets destroy enemies
- **Scoring**: Verify combo multiplier increases/breaks, best score saves
- **UI responsiveness**: Check HUD updates on mobile/desktop, touch controls work
- **Performance**: Monitor FPS on low-end devices; should gracefully degrade

## Common Gotchas

- **Audio context**: Browsers require user gesture before playing audio; `ensureAudio()` handles this
- **Canvas coordinate system**: (0,0) is top-left; Y increases downward
- **Collision timing**: Bullets/enemies checked each frame; fast objects may tunnel
- **Z-order**: HUD, boss bar, overlays managed via CSS `z-index` and canvas drawing order
- **localStorage**: Quota limits (~5MB); scores/settings are minimal
- **Touch vs. mouse**: Both can be active simultaneously; game prioritizes one based on recent input
