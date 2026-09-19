# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 canvas application. The game features multiple difficulty levels, dynamic enemy spawning, weapon upgrades, smart bombs, boss encounters, and full mobile/touch support.

## Architecture

The game is implemented as a single 1669-line `index.html` file with three major sections:

### 1. **Markup & DOM Structure** (lines 1–420)
- Canvas element for rendering
- HUD layer showing score, multiplier, health (hearts), bombs, and weapon status
- Boss health bar (shown during boss encounters)
- Overlay system for menus, pause, and game-over screens
- Touch control zones (joystick-style movement + action buttons)
- Accessibility layer (screen reader announcements)

### 2. **Styling** (lines 10–359)
- CSS Grid/Flexbox for responsive UI
- Dark theme optimized for arcade feel
- Responsive font sizing with `clamp()`
- Animations for HUD elements, bosses, and effects (combos, stage banners)
- Reduced motion support (respects `prefers-reduced-motion`)
- Safe-area insets for notched devices

### 3. **Game Logic** (lines 422–1669)
Organized roughly in this order:
- **Constants**: Difficulty settings, theme definitions, enemy/boss specs
- **State variables**: Game state (running, paused, score), entity lists (enemies, projectiles, explosions)
- **Input handling**: Keyboard, mouse, touch
- **Audio system**: WebAudio synth for all sound effects (no external files)
- **Entity rendering**: Player ship, enemies, bosses, explosions, particles
- **Game loop**: `update()` for logic, `draw()` for rendering
- **Collision detection**: Rectangle overlap checks
- **Power-up system**: Weapon upgrades, multiplier chains, health/bombs
- **Event listeners**: Start button, pause, difficulty selection

## Key Game Systems

### Entity Management
- **Player**: Single controlled ship with health, weapon, bomb count
- **Enemies**: Three types (grunt, interceptor, tank) spawn based on difficulty and stage
- **Bosses**: Six unique boss ships appear after 18 enemy kills per stage
- **Projectiles**: Tracked separately for player bullets and enemy fire
- **Explosions**: Particle effects spawned on kills
- **Power-ups**: Weapon pickups, health restores, bomb refills

### Weapon System
Three weapon types (Vulcan, Laser, Spread) with 3 upgrade levels each. Weapon choice affects firing pattern (single burst, beam, or spread). Collected power-ups increment the current weapon's level.

### Scoring & Chains
- Base score from kills varies by enemy type
- Multiplier starts at 1x and increases for consecutive kills
- Chain breaks when the player takes damage or time elapses
- Combo pop-up displays current chain

### Difficulty Levels
Three presets (Easy, Normal, Hard) adjust enemy spawn rate, speed, fire rate, and boss health. Player chooses difficulty at start; preference is saved to local storage.

### Adaptive Performance
The game tracks frame time over 45 frames and adjusts visual quality (skips certain effects) if fps drops below ~25fps. Quality scale: 0 = full effects, 1 = medium, 2 = minimal.

### Mobile/Touch Support
- Left half of screen: drag to move player (virtual joystick)
- Right half: FIRE and BOMB action buttons
- Touch controls are hidden on desktop; joystick only shows when touched

### Audio
All sound is synthesized at runtime using WebAudio:
- `beep()`: Creates tones with optional frequency sweep, filter, or pan
- `noiseBurst()`: White noise for explosions and impacts
- No external audio files required

## Development Notes

### Modifying Gameplay
- **Enemy types**: Edit `ENEMY_TYPES` constant to adjust speed, HP, or scoring
- **Bosses**: `BOSSES` array defines name and color scheme; `spawnBoss()` creates instances
- **Difficulty tuning**: Adjust `DIFFICULTIES` object (spawn rates, enemy speed, lives, bombs)
- **Themes**: `THEMES` array defines sky gradients and colors per stage
- **Weapon patterns**: Functions like `fireRadial()`, `fireAimedSpread()`, `fireColumns()` define firing logic

### Canvas Sizing
Base resolution is 480×720 (portrait). The game uses responsive canvas sizing with `resize()` to fit viewport while maintaining aspect ratio and scaling game coordinates accordingly.

### Collision System
`rectsOverlap()` performs simple axis-aligned bounding-box checks. Collision detection happens during `update()` for player bullets vs enemies, enemy bullets vs player, and projectile vs player collisions.

### Debugging Tips
- Player state: `player` object holds position, health, weapon, bomb count
- Active enemies: `enemies` array; active boss: `boss` object
- Frame counter: `frame` increments each tick (useful for animation timings)
- Performance quality: `quality` variable (0–2); lower values = fewer effects rendered

### Common Edits
- **Change default lives**: Modify `DIFFICULTIES.normal.lives` (and easy/hard)
- **Adjust starting bombs**: Edit `lives` and `bombs` fields in `DIFFICULTIES`
- **Tweak spawn rate**: Lower `spawnBase` = more frequent spawns
- **Add a new weapon**: Create a handler in `fireWeapon()` and add color/name to HUD
- **New boss design**: Add entry to `BOSSES` array and update `drawBossShip()` color logic

### State Persistence
- Best score, mute state, difficulty, and reduced-motion setting are saved to `localStorage` with keys like `skyfighter.bestScore`
- Cleared only by browser clear-cache or manual deletion from dev tools Storage tab

## Browser Compatibility
- Requires ES6 (arrow functions, const, let, template literals)
- WebAudio API for sound
- Canvas 2D context
- LocalStorage for persistence
- No external dependencies; fully self-contained
