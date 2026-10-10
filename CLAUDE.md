# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game implemented as a single-file HTML5 web application. The game features:
- Multiple difficulty levels (Easy/Normal/Hard) with persistent preferences
- Stage-based progression with boss encounters
- Weapon power-ups and smart bomb mechanics
- Procedurally-generated audio (WebAudio synthesis, no external assets)
- Responsive mobile support with touch controls
- Adaptive rendering quality based on frame time performance
- Accessibility features: reduced motion support, screen reader announcements

## Quick Start

To run the game:
1. Open `index.html` in a modern web browser
2. Select difficulty and click "Start Game"
3. Controls: Mouse movement to fly, click to shoot, X/Shift for bombs
   - Mobile: drag left to position, tap FIRE/BOMB buttons
   - Keyboard: Arrow keys/WASD to move, Space to fire, P to pause

No build process, development server, or external dependencies required.

## Architecture Overview

### Single-File Structure
All game code lives in `index.html` (~58KB):
- **HTML** (lines 1-420): DOM elements for canvas, HUD, UI controls, overlay
- **CSS** (lines 10-398): Styling, animations (comboPulse, vignettePulse, flashHit, etc.), responsive design
- **JavaScript** (lines 422+): Game logic and engine

### Core Game Loop
Driven by `requestAnimationFrame`, the main loop:
1. Processes player input (keyboard, mouse, touch)
2. Updates game state (entity positions, timers, collisions)
3. Renders the scene to canvas
4. Plays audio cues
5. Adapts rendering quality if frame times exceed targets

Quality adaptation (lines 474-483): Tracks 45-frame average; reduces quality if avg > 25ms or increases if < 17.9ms to maintain 40+ FPS.

### Game State Management
- **Difficulty settings**: `DIFFICULTIES` object defines spawn rates, enemy speed, lives, bombs, boss HP multiplier
- **Entity systems**: Separate arrays for player, enemies, bullets, bombs, powerups
- **Collision detection**: Axis-aligned bounding box (AABB) checks between projectiles, enemies, player
- **Stage progression**: Tracks kills; 18 kills per stage triggers boss spawning and next stage

### Visual Systems
- **Canvas rendering**: 2D context for sprites, particles, HUD elements
- **Theme system** (line 493-500): 6 procedurally-colored sky themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) with rotating background gradients
- **Enemy types** (line 502-506): Grunt, Interceptor, Tank with unique colors, speeds, HP, and behaviors
- **Boss system** (line 508-515): 6 named bosses (Vanguard, Behemoth, Specter, etc.) with distinct color schemes and HP multipliers
- **HUD overlays**: Real-time score, multiplier, weapon/level, hearts, bombs, boss health bar, stage banners, combo popups
- **Visual effects**: Vignette flashing on critical health, white screen flash on bomb, damage indicators

### Audio System
Procedurally synthesized with Web Audio API (no external samples):
- **Oscillator-based**: beep() generates tones with frequency sweeps, filters, panning
- **Noise bursts**: noiseBurst() creates explosion and impact sounds
- **Sound effects** (lines 624-676): shoot, laser, explode (entity-type specific), hit, bomb, powerup, boss hit, boss telegraph, boss death, stage clear, UI click, gameover
- **Master gain and compressor**: Dynamic range compression for audio balance
- **Stereo panning**: X-position of events pans left/right for spatial audio

### Mobile & Accessibility
- **Touch controls**: Virtual joystick in move zone, FIRE and BOMB buttons
- **Reduced motion**: Respects `prefers-reduced-motion` media query; toggles animations via `reduced-motion` class
- **Screen reader support**: `#srLive` region announces score, level-ups, game events via `announce()` function
- **Safe areas**: Respects `env(safe-area-inset-*)` for notch/punch-hole displays

### Persistence
Uses `localStorage` for:
- Best score (`skyfighter.bestScore`)
- Mute preference (`skyfighter.muted`)
- Selected difficulty (`skyfighter.difficulty`)
- Reduced motion preference (`skyfighter.reducedMotion`)

## Key Mechanics

### Weapons & Power-ups
- **Weapon tiers**: Dual cannons → Heavy turret → Laser array (names, fire rates, damage)
- **Power-up drops**: Fallen enemies drop weapon power-ups and bonus points
- **Bomb capacity**: Upgradable from initial count (difficulty-dependent)

### Enemy Behaviors
- **Spawn waves**: Spawn interval decreases as stage progresses and quality increases
- **Movement**: Vertical descent with optional weaving (Interceptor type)
- **Shooting**: Probabilistic based on difficulty; enemies fire downward at player
- **Collision**: Enemies deal damage to player on touch; player bullets destroy enemies

### Boss Encounters
- Spawns after 18 kills per stage
- Boss-specific HP multiplier from `BOSSES` array
- Multi-phase health bar displayed during fight
- More aggressive attack patterns than regular enemies
- Stage clears on boss defeat; difficulty and themeset advance

### Multiplier System
Consecutive hits without taking damage increase score multiplier (visual feedback via orange combo counter).

## Editing Guidelines

### Adding New Content
- **New enemy type**: Add entry to `ENEMY_TYPES` object, update `EXPLODE_SFX` with sound effect
- **New boss**: Add entry to `BOSSES` array with name, colors, HP multiplier
- **New theme**: Add to `THEMES` array with sky gradient, cloud/sun RGB values, display name
- **New sound effect**: Add to `sfx` object; compose with `beep()` and `noiseBurst()` calls

### Modifying Game Balance
- **Difficulty tuning**: Adjust `DIFFICULTIES` object (spawnBase, spawnMin, enemySpeed, shootChance)
- **Enemy stats**: Modify `ENEMY_TYPES` (w, h, hp, speedMul, score)
- **Boss HP**: Adjust base hpMul in `BOSSES` or overall `bossHpMul` per difficulty
- **Stage length**: Change `KILLS_PER_STAGE` constant

### Common Customizations
- **Canvas size**: Adjust in style (`#gameCanvas`) and dynamically in JavaScript after canvas creation
- **Color scheme**: Theme colors in `THEMES`; HUD colors in CSS variables (--primary, --accent, etc.)
- **Control scheme**: Input handling in keyboard/mouse/touch listeners; modify key bindings and button mappings

## Testing & Iteration

Run locally by opening the file in a browser. Test across:
- **Desktop**: Multiple browsers (Chrome, Firefox, Safari, Edge)
- **Mobile**: iOS (Safari) and Android (Chrome) with touch controls
- **Accessibility**: Enable reduced motion, mute audio, test screen reader with different assistive technologies
- **Performance**: Use DevTools to monitor frame rate; test on low-end devices to verify adaptive quality kicks in

## Dependencies

**Zero external dependencies.** All functionality is self-contained:
- Canvas 2D API for graphics
- Web Audio API for sound
- localStorage for persistence
- No npm, build tools, or libraries

## Notes for Future Development

1. **Collision optimization**: Current AABB checks are O(n²) per entity type; spatial hashing would improve performance at higher enemy counts.
2. **Asset loading**: To add sprite graphics, embed base64 data URIs or canvas drawing routines (no HTTP requests to maintain offline functionality).
3. **Persistent data**: Extended leaderboards or progress beyond localStorage would require a backend.
4. **Modding**: Game balance and content are easily tweakable; consider extracting configs into a separate JSON for non-programmer customization.
