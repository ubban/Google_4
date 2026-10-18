# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas application. The game features:
- Multiple difficulty levels (Easy, Normal, Hard)
- Progressive enemy spawning across 6 stages per run
- Boss encounters at stage transitions
- Weapon upgrades and power-ups
- Smart bomb mechanics
- Responsive controls (mouse, touch, keyboard)
- Difficulty-adaptive performance scaling

## How to Run

The game is a single `index.html` file. To play:
1. Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge)
2. The game starts with a difficulty selection screen
3. No build process, dependencies, or server required

## Architecture

All game logic is contained in a single IIFE (Immediately Invoked Function Expression) in the `<script>` section starting at line 422. The architecture is organized around these core systems:

### Canvas & Rendering
- Canvas 2D context handles all drawing
- Adaptive quality system (`quality` variable tracks 0=full, 1=medium, 2=low) automatically reduces particle count and effects if FPS drops below target
- Frame time tracking for performance monitoring

### Game State Management
The game cycles through states:
- **TITLE** - Difficulty selection screen
- **PLAYING** - Active gameplay
- **PAUSED** - Game paused
- **BOSS** - Boss battle (enters this state when boss spawns)
- **GAME_OVER** - Player defeated
- **GAME_WON** - All stages cleared

Key state variables:
- `gameState` - Current state enum
- `stage`, `kills` - Track progress (KILLS_PER_STAGE = 18 kills per stage)
- `score`, `multiplier` - Scoring system with combo multiplier
- `lives`, `bombs` - Player resources

### Entity System
The game manages several entity arrays:
- **Player** - Single player ship with collision, weapon, bomb mechanics
- **Enemies** - Array of enemies with types: grunt, interceptor, tank
- **Bullets** - Player projectiles (two weapon types with different speeds/damage)
- **Enemy bullets** - Enemy fire
- **Particles** - Visual effects (explosions, impacts)
- **Drops** - Power-ups and collectibles

### Enemy & Boss Data
- `ENEMY_TYPES` - Defines grunt, interceptor, tank (hp, speed, score, appearance)
- `BOSSES` - 6 unique boss designs with color palettes and HP multipliers
- `THEMES` - 6 visual themes with sky gradients and color schemes (one per stage)

### Game Balance Configuration
**Difficulty settings** (`DIFFICULTIES` object):
- `spawnBase/spawnMin` - Controls enemy spawn rate
- `enemySpeed`, `enemyShootChance` - Enemy behavior scaling
- `lives`, `bombs`, `bossHpMul` - Player resources and boss difficulty

### Input Handling
- **Mouse/Touch** - Crosshair aiming; player follows cursor
- **Keyboard** - WASD or arrow keys for movement; Space/Ctrl for fire/bomb
- **Touch controls** - Virtual joystick (`.moveZone`, `.fireBtn`, `.bombBtn`) for mobile

### Rendering Order (Z-index)
- Background sky and clouds (z=0)
- Game entities and particles (z=1-2)
- HUD elements (z=5)
- Vignette/flash effects (z=3, 7)
- Combo/stage banners (z=6)
- UI overlay (z=10+)

### Key Game Constants
- `KILLS_PER_STAGE = 18` - Kills needed to advance stage
- Spawn rate: adjusts based on difficulty and kills counter
- Score multiplier: increases with consecutive kills without being hit, resets on damage

## Development Notes

### Single-File Structure
The entire game lives in `index.html` with three sections:
1. **HTML markup** (lines 1-421) - Canvas, HUD, UI elements, overlay screens
2. **CSS** (lines 10-421) - Responsive styling, animations, dark theme
3. **JavaScript** (lines 422+) - Game engine in a single IIFE closure

### Accessibility Features
- ARIA live region (`#srLive`) for screen reader announcements
- Motion reduction support (`--motion-ok` class, `reducedMotion` flag)
- Button controls labeled with semantic HTML

### Storage & Persistence
- `localStorage` keys (prefixed `skyfighter.`):
  - `bestScore` - High score across sessions
  - `muted` - Sound mute state
  - `difficulty` - Last selected difficulty
  - `reducedMotion` - Accessibility preference

### Performance Optimization
- Particle systems limited by quality setting
- Enemy spawning uses frame-skip logic to avoid frame drops
- Canvas clearing and redrawing each frame (~60 FPS target)
- Adaptive performance avoids quality reduction until FPS consistently drops

### Common Edits
- **Add new enemy type**: Add entry to `ENEMY_TYPES` object with `w, h, hp, speedMul, color, dark, score, minStage` properties
- **Add new boss**: Add entry to `BOSSES` array with color palette and `hpMul`
- **Adjust difficulty**: Edit the `DIFFICULTIES` object values
- **Change visual theme**: Edit `THEMES` array with new gradient colors
- **Tweak gameplay constants**: Spawn rates, speeds, and collision sizes are near the top of the script
