# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file (`index.html`). The game features multiple difficulty levels, weapon progression, boss fights with distinct attack patterns, and adaptive performance scaling.

## Architecture

The entire game is implemented in a single self-contained HTML file:

- **CSS (lines 10–358)**: Styling for canvas, UI elements, HUD, overlays, animations (flash, vignette, combo popups), and responsive layout
- **JavaScript (lines 422–1666)**: Complete game engine implementing:
  - **Game state management**: Player, bullets, enemies, projectiles, particles, power-ups
  - **Input handling**: Keyboard (arrows/WASD, Space, X/Shift), mouse (move to steer, click to fire), touch (joystick + buttons)
  - **Rendering**: Canvas 2D with adaptive performance (quality levels 0–2), sprite drawing (planes, bosses), particle effects
  - **Audio**: WebAudio API synth — no external assets, all procedural sounds for weapons, explosions, UI clicks
  - **Gameplay systems**: Difficulty modes, enemy spawning, weapon power-ups, bomb mechanics, boss encounters, score multiplier chain

## Key Game Systems

### Difficulty Scaling
Three presets (`easy`, `normal`, `hard`) stored in `DIFFICULTIES` constant (lines 485–489):
- Affect spawn rate, enemy speed, shoot chance, lives, bombs, boss HP

### Player Progression
- **Weapons**: VULCAN (spread shots) vs LASER (piercing, multi-lane)
- **Levels**: Each weapon has 3 levels, increasing spread/lanes
- **Power-ups**: Life, bomb, weapon (collected from defeated enemies)

### Enemy Types
Three types in `ENEMY_TYPES` constant (lines 502–506):
- **Grunt**: Basic red enemy
- **Interceptor**: Fast, weaving orange enemy
- **Tank**: Slow, durable purple enemy with visual HP bar

### Boss Fights
Six bosses in `BOSSES` constant (lines 508–515) with unique color schemes. Each boss has 3 attack patterns defined in `BOSS_PATTERNS` (lines 1372–1409):
- VANGUARD: radial burst, aimed fan, wide V-fan
- BEHEMOTH: columns, homing spread, dense radial
- SPECTER: teleport dash, spiral, double aimed shots
- DREADNOUGHT: 8-way cross, homing volley, radial wall
- WRAITH: teleport barrage, double ring, cross-spread
- TITAN: columns, full ring, homing volley

### Adaptive Performance
The game tracks frame times and adjusts quality level (0 = full, 1 = medium, 2 = low) by reducing particle counts and shadow effects (lines 474–483).

### Visual Themes
Six sky themes in `THEMES` constant (lines 493–500) that cycle per stage, affecting sky gradient, cloud color, and sun glow.

## Core Game Loop

1. **Input processing** (lines 768–861): Keyboard, mouse, touch input handling
2. **Update** (lines 1148–1302): Player movement, weapon fire, collision detection, enemy/boss AI
3. **Rendering** (lines 1520–1638): Sky gradient, clouds, entities, particles, HUD
4. **Loop control** (lines 1640–1665): RAF-based loop with delta-time capping at 50ms

The game uses a screen shake effect for impact feedback, flash animations on hit/bomb, and a vignette that pulses when health is critical.

## Development Notes

### Adding Content
- **New enemy type**: Add to `ENEMY_TYPES`, assign minStage, define draw logic in the enemy drawing section
- **New boss**: Add to `BOSSES`, define 3 attack patterns in `BOSS_PATTERNS` (use helper functions like `fireRadial`, `fireColumns`, `fireAimedSpread`)
- **New weapon**: Modify `fireWeapon()` to add branches for the weapon name, adjust spread/lanes based on level
- **New theme**: Add sky/cloud/sun colors to `THEMES`

### Game Constants
All tuning values are at the top of the script:
- `STORAGE_*`: Local storage keys
- `DIFFICULTIES`: Spawn rates, speeds, lives
- `KILLS_PER_STAGE`: Enemies to defeat before boss
- `ENEMY_TYPES`, `BOSSES`, `THEMES`: Content definitions

### Audio
All sound effects use WebAudio synth (no external files):
- `beep(freq, dur, type, vol, opts)`: Tone generation with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: Noise-based explosions
- `sfx.*`: Named sound effects that call beep/noiseBurst combos

Muting and disabled animations are controlled via localStorage. The `ensureAudio()` call initializes the AudioContext on first interaction (required for browser autoplay policy).

### Collision Detection
Rect-based using `rectsOverlap()` (line 1069) for player/enemy, bullet/enemy, bomb/enemy. Boss hit detection checks `!boss.entering` to avoid collisions during the entry animation.

### Canvas Sizing
Base resolution is 480×720 (lines 753–765). The canvas scales up to fill the window with a max scale of 1.6. Touch-based joystick and buttons only show on touch devices.

### Performance Considerations
- Particle spawning scales with quality setting (full/medium/low)
- Boss telegraph (wind-up) visual uses shadow bloom only at quality 0
- Frame-per-frame optimizations: bullets/enemies filtered after movement, invulnerability flicker uses frame counter
- LocalStorage reduces initial load (best score, difficulty, mute state)

## No Build or Lint Steps

This is a static HTML file. To develop:
1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Edit the file directly in a text editor
3. Refresh the browser to see changes
4. Weapon/enemy balance tweaking: adjust constants in `DIFFICULTIES`, `ENEMY_TYPES`, `BOSS_PATTERNS`
5. Difficulty/content tuning: modify spawn rates, HP multipliers, attack pattern frequencies

## Browser Compatibility

Requires:
- Canvas 2D context
- WebAudio API (AudioContext)
- ES6 (arrow functions, const/let, template literals)
- LocalStorage
- Touch events (for mobile)
