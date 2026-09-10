# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game, implemented as a single-file HTML5 application with embedded CSS and JavaScript. The entire game runs in the browser with no build tools, dependencies, or external assets.

### Key Characteristics
- **Single-file deployment**: `index.html` contains all code and styling
- **No build process**: Open in browser directly, no compilation needed
- **No external assets**: All graphics are canvas-drawn, all audio is synthesized via WebAudio API
- **Responsive design**: Scales to desktop, tablet, and mobile screens
- **Accessibility**: Keyboard controls, touch controls, reduced-motion support, screen reader announcements

## Game Architecture

### Core Game Loop
The game runs a continuous render loop using `requestAnimationFrame`. Key phases in each frame:
- **Input processing**: Mouse/keyboard/touch input collected and normalized
- **Game state update**: Player movement, enemy spawning, collision detection, weapon firing
- **Rendering**: Draw background theme, all entities, HUD, effects
- **Audio**: Synthesize sound effects for events

### Main Game Systems

#### 1. **Difficulty System**
Difficulty settings affect:
- Enemy spawn rates and minimum delays
- Enemy movement speed and attack frequency
- Player starting lives and bombs
- Boss health multiplier
- Starting weapon power

```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... }
};
```

#### 2. **Enemy System**
Three enemy types with distinct behaviors:
- **Grunt**: Basic red enemy, spawns early, low HP
- **Interceptor**: Fast orange enemy, weaves side-to-side
- **Tank**: Purple, heavy armor, high HP, rare

Each enemy has configurable speed multiplier, health, spawn threshold (minStage), and score value.

#### 3. **Boss System**
Boss entities spawn at 18 kills per stage and have unique visual themes (hull colors, wing colors, core colors). Boss HP is calculated as `baseHP * difficulty.bossHpMul * boss.hpMul`. Six boss variants progress through stages.

#### 4. **Weapon System**
Multiple weapons with upgrade levels:
- **VULCAN**: Basic rapid-fire weapon
- **LASER**: High-damage beam weapon
- **SPREAD**: Cone attack pattern
- **RING**: Circular projectile burst

Weapons upgrade through power-up drops. Each weapon has distinct firing patterns and cooldown rates.

#### 5. **Audio System**
Custom WebAudio synthesizer (no MP3/WAV files):
- **Beep function**: Frequency sweep, envelope, low-pass filter, stereo pan
- **Noise burst**: For explosions and impacts
- All sounds are muted-state aware and dynamically panned based on screen position

#### 6. **Rendering & Themes**
Dynamic background themes with sky gradient, cloud patterns, and sun sprite. Six themes cycle through stages:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge

#### 7. **Input Handling**
Unified input system supporting:
- **Desktop**: Mouse movement, click to fire, keyboard (arrows/WASD/space/X)
- **Touch**: Drag-to-move gesture, on-screen FIRE and BOMB buttons
- **Pause**: P or Esc key

#### 8. **Performance Adaptation**
Frame time tracking automatically reduces rendering quality (entityQuality variable) if average frame time exceeds 1/40s, improving performance on slower devices without manual configuration.

### HUD & Overlays

- **Main HUD**: Score, best score, multiplier, weapon level, health (hearts), bomb count
- **Boss health bar**: Shows when boss spawns, positioned at top center
- **Game over overlay**: Shows final stats, difficulty selector, restart button
- **Accessibility**: Screen-reader announcements for game events via `#srLive` element

## Common Development Tasks

### Modifying Game Balance
Edit the `DIFFICULTIES` object (~line 485) to adjust spawn rates, enemy speed, or player starting resources.

### Adding a New Enemy Type
1. Define in `ENEMY_TYPES` with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add spawn logic in the enemy spawning section that checks `minStage`
3. Enemy rendering uses the `color` and `dark` properties for shading

### Adjusting Boss Progression
Edit `BOSSES` array to change boss hull/core colors, names, or HP multipliers. Bosses spawn sequentially; adding new entries extends the boss roster.

### Changing Themes
Add new theme objects to `THEMES` array with:
- `sky`: Array of 3 gradient colors (hex format)
- `cloud`: RGB values for cloud rendering
- `sun`: RGB values for sun sprite
- `name`: Display name for theme

### Audio Tuning
Modify `beep()` or `noiseBurst()` functions to change sound character. The `opts` parameter controls sweep range, filter frequency, pan, and timing.

## Performance Considerations

- **Canvas rendering**: All graphics drawn procedurally; no sprite sheets or images to load
- **Entity pooling**: Consider implementing object pools for projectiles/particles if frame count becomes very high
- **Quality scaling**: The `quality` variable (0-2) already adapts rendering, but texture resolution and entity LOD could be further optimized
- **Audio synthesis**: WebAudio oscillators are CPU-intensive; excessive concurrent sounds may cause frame drops on mobile

## File Structure

- `index.html`: Complete game (HTML markup, CSS styling, JavaScript logic ~1669 lines)
- `README.md`: Project metadata
- `.git/`: Version control history

## Deployment Notes

- No server required; serve as static file over HTTP(S)
- Works offline after first load (no external CDN dependencies)
- Mobile-friendly; touch controls activate automatically on touch devices
- Safe for direct browser opening (file:// protocol works)
- IndexedDB/localStorage used for best score persistence and settings (no server backend needed)
