# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game. The entire game (HTML, CSS, and JavaScript) is contained in `index.html`. It's designed to run in any modern browser with no build process required.

### Key Technologies
- HTML5 Canvas for 2D graphics rendering
- Vanilla JavaScript (no frameworks)
- Responsive CSS Grid/Flexbox for UI overlays
- LocalStorage for persistence (best score, settings)

## Running the Game

**Development**: Simply open `index.html` in a modern browser to play. No build or dev server required.

**Distribution**: The file can be served statically over HTTP or used locally.

## Code Architecture

The entire game logic is in `index.html` within a single IIFE (Immediately Invoked Function Expression) starting at line 423. The structure is:

### 1. **DOM Elements & Constants** (lines 426-467)
- Query selectors for all UI elements (HUD, overlays, buttons, canvas)
- Storage keys for localStorage persistence
- Constants for difficulty settings, kill counts, visual themes

### 2. **Core Systems**

#### Game State & Settings (lines 485-520)
- `DIFFICULTIES`: Three difficulty presets that control:
  - Enemy spawn rates (`spawnBase`, `spawnMin`)
  - Enemy behavior (`enemySpeed`, `enemyShootChance`)
  - Player stats (`lives`, `bombs`, `bossHpMul`)
- `THEMES`: Six visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) with sky gradients and color palettes
- `KILLS_PER_STAGE`: Threshold to progress (set to 18)

#### Utility Functions (lines ~522-600)
- `announce()`: Screen reader announcements for accessibility
- `trackPerf()`: Adaptive performance quality scaling (0=full, 1=medium, 2=low quality)

#### Game Objects & Entities
The game uses classes/constructors for:
- **Player**: Position (x, y), velocity (vx, vy), health, weapon state, bomb count
- **Enemies**: Position, velocity, health, behavior (random movement, shooting)
- **Bosses**: Similar to enemies but with specific names, higher health, coordinated attacks
- **Bullets**: Position, velocity, damage, owner (player vs enemy)
- **Explosions**: Particle effects for visual feedback
- **Collectibles**: Weapon power-ups that upgrade player weapons

#### Main Game Loop
- `update(dt)`: Updates all entity positions, checks collisions, handles spawn logic
- `render()`: Draws all entities to canvas, updates HUD
- `requestAnimationFrame()`: Drives the 60fps render loop

### 3. **Input Handling**
- **Keyboard**: Arrow keys/WASD for movement, Space for fire, X/Shift for bombs, P/Esc to pause
- **Mouse**: Position tracks player location, click to fire
- **Touch**: Mobile joystick (left 55% of screen for movement) and action buttons (right 45% for fire/bomb)

### 4. **Game Flow**
1. **Title Screen**: Difficulty selection, show high score
2. **Stage Start**: Banner displaying current stage (with 6 themes cycling by stage)
3. **Gameplay Loop**: Spawn enemies based on difficulty, handle player input, check win/loss conditions
4. **Boss Encounter**: Special enemies with higher health spawn after killing KILLS_PER_STAGE enemies
5. **Game Over**: Show score, check for new best, return to title screen

### 5. **UI System**
- **HUD** (top center): Score, Best Score, Multiplier, Weapon name/level, Hearts, Bombs
- **Boss Bar** (top center when boss active): Boss name and health bar
- **Overlay** (modal): Title screen, game over screen with stats
- **Top Right Buttons**: Mute toggle, Reduced Motion toggle, Pause button
- **Touch Controls** (mobile): Joystick and action buttons in bottom area
- **Vignette Effect**: Red pulse when player takes damage
- **Screen Flash**: White flash on hit, yellow flash on bomb

### 6. **Game Mechanics**

#### Weapons
- **Vulcan** (default): Rapid-fire basic bullets
- **Spread**: 3-way spread shot
- **Laser**: Single powerful beam
- **Missiles**: Guided projectiles
Each weapon has 3 levels, each level increases fire rate or damage.

#### Difficulty Impact
- **Easy**: Lower spawn rates, slow enemies, less accurate enemy fire, 4 lives, 3 bombs, bosses have 20% less HP
- **Normal**: Balanced spawn rates and behavior, 3 lives, 2 bombs, standard boss HP
- **Hard**: High spawn rates, fast enemies, accurate enemy fire, 3 lives, 2 bombs, bosses have 30% more HP

#### Score System
- Kill multiplier resets when hit, rewards consecutive kills
- Weapon upgrades visible in HUD
- Best score persists in localStorage

#### Performance Adaptation
Monitors frame time over 45 frames and reduces quality (particle count, effect complexity) if FPS drops below 40.

## Modification Guide

### Adding New Enemies
1. Define new enemy type in the entities section (stats like speed, health, color, shooting pattern)
2. Add to `spawnEnemies()` with conditional logic based on stage or difficulty
3. Add collision handler in `checkCollisions()`

### Adding New Weapons
1. Add to weapons array/object with properties: `name`, `fireRate`, `bulletSpeed`, `damage`, `spread`
2. Update rendering logic in `render()` for visual distinction
3. Add firing logic in `handlePlayerFire()` with unique bullet patterns

### Adjusting Difficulty
Edit `DIFFICULTIES` object values:
- `spawnBase`/`spawnMin`: Lower = harder (enemies spawn more frequently)
- `enemySpeed`: Higher = harder
- `enemyShootChance`: Higher = harder
- `bossHpMul`: Higher = harder (boss health multiplier)

### Adding Visual Themes
Add new object to `THEMES` array with:
- `sky`: Array of 3 gradient colors
- `cloud`: RGB string for cloud color
- `sun`: RGB string for sun color
- `name`: Display name for theme

Themes cycle by stage: Stage 1→Daylight, Stage 2→Sunset, etc.

## Accessibility

- **Reduced Motion**: Toggle disables all animations (pulsing, screen shake, effects)
- **Screen Reader**: `announce()` function provides updates for assistive devices
- **Color Contrast**: HUD uses high-contrast white/yellow/green on dark backgrounds
- **Touch-Friendly**: Large 78px buttons, full-screen canvas with safe-area insets for notched phones

## Performance Considerations

- **Canvas Rendering**: Single 2D canvas context, no WebGL
- **Collision Detection**: Rectangular bounding boxes (simple AABB)
- **Entity Count**: Typically 50-200 entities during gameplay (enemies, bullets, particles)
- **Adaptive Quality**: Automatically reduces particle effects on slower devices
- **No External Assets**: All graphics are generated on-the-fly or via CSS gradients

## Browser Compatibility

- Requires `requestAnimationFrame`, `localStorage`, `Canvas 2D Context`
- Tested on modern Chrome, Firefox, Safari, Edge
- Works on mobile browsers with touch events
- Fallback to keyboard if touch not supported

## Future Enhancement Ideas

- Local multiplayer (split-screen or co-op)
- Leaderboard API integration
- Additional weapon types (beam, homing, AOE)
- Enemy formations and scripted patterns
- Parallax scrolling backgrounds
- Sound effects and music (currently mutable only)
- Mobile-native app packaging (Cordova/React Native)
- Procedurally generated enemy waves
