# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file (index.html) with embedded Canvas graphics, JavaScript game logic, and procedurally-generated audio using the Web Audio API.

## Architecture

### Single-File Game Structure

The entire game is contained in `index.html`:
- **HTML**: Game layout with canvas, HUD elements, overlay screens, and touch controls
- **CSS**: Game styling, animations, and responsive design with safe-area support for mobile devices
- **JavaScript**: Game loop, input handling, game state, collision detection, rendering, and audio synthesis

### Key Game Systems

#### Game State Management
- **Player object**: Position, speed, weapon type/level, health, invulnerability timer
- **Enemies array**: Spawned dynamically with different types (grunt, interceptor, tank) and behaviors
- **Bullets/Enemy bullets**: Player projectiles and incoming fire with collision detection
- **Boss system**: Stage-based boss encounters with pattern-based attacks
- **Power-ups**: Weapon upgrades, extra lives, and bombs
- **Particles**: Explosion and visual effects that fade over time

#### Input Handling (Multi-Platform)
- **Keyboard**: Arrow keys/WASD for movement, Space/Click for fire, X/Shift for bombs, P/Esc for pause
- **Mouse**: Movement-based steering (plane glides toward pointer) + click to fire
- **Touch**: Virtual joystick in left 55% of screen + on-screen FIRE and BOMB buttons in right 45%
  - Joystick shows visual feedback (base circle + movable stick)
  - Buttons are 78px circles at bottom right

#### Audio System
- **Synthesized only**: All sounds generated with Web Audio API oscillators, filters, and noise bursts
- **No external assets**: Complete audio dictionary (`sfx` object) with sounds for:
  - Weapon fire (vulcan shoot + laser)
  - Enemy explosions (grunt/interceptor/tank variants)
  - Player hit/bomb
  - Boss telegraph + defeat
  - Power-up collect
  - UI click
- **Spatial audio**: Panning calculated from screen position
- **Master gain + compression**: Dynamic range controlled via compressor node

#### Difficulty System
Three difficulty levels stored in localStorage:
- **Easy**: Slower spawning, weaker enemies, 4 lives, slower bullets
- **Normal** (default): Balanced gameplay, 3 lives
- **Hard**: Intense enemy spawning, aggressive bosses, faster bullets

#### Rendering Pipeline
- **Sky backgrounds**: 6 animated themes with gradient + cloud layers
- **Entity drawing**: 
  - Player/enemy/boss planes drawn with ctx.fillStyle and paths
  - Laser weapon has glowing aura
  - Bullets: rectangles (player) vs circles (enemy)
  - Power-ups: rotating squares
- **Effects**: 
  - Screen shake (random offset to canvas translation)
  - Flash vignette on critical health
  - Hit flash (white overlay)
  - Combo popup text
  - Boss health bar

### Score and Progression

- **Kill chains**: Consecutive kills without gaps increase multiplier (1x to 10x)
- **Chain timer**: 2.2 seconds between kills to maintain chain
- **Stages**: Triggered after 18 enemy kills + boss defeat
- **Boss selection**: Cycles through 6 bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)

### Boss Attack Patterns

Each of the 6 bosses has 3 unique attack patterns:
- **Pattern functions**: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
- **Telegraph system**: Visual telegraph (core glows) for 22 frames before attack fires
- **Attack cycle**: Boss fires, pauses, telegraphs next attack

### Weapon System

- **Vulcan (default)**: Spread shot that increases width with levels 1-3
- **Laser**: Piercing shots with higher fire rate, glowing aura, and lane separation at level 2+
- **Switch mechanic**: Collecting 3 weapon upgrades switches weapon type

### Performance Optimization

- **Adaptive quality**: Frame time tracking adjusts particle/shadow quality
  - Quality 0 (full): all effects, shadow blur
  - Quality 1 (medium): reduced particles (60%), minimal shadows
  - Quality 2 (low): minimal particles (35%), no shadows
- **Object pooling**: Bullets/particles recycled via array filter
- **Canvas size**: Fixed 480×720 logical, scaled to fit viewport (max 1.6x)

### Accessibility Features

- **Reduced motion**: Toggle to disable animations (screen shake, flash, pulsing) per user preference
- **Screen reader support**: Live region announcements for game state changes
- **Color contrast**: High contrast UI with dark backgrounds and light text
- **Touch support**: Full touch control with 100px joystick base

### State Persistence (localStorage)

Keys stored:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute state (1/0)
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Motion preference override

## Common Development Tasks

### Testing the Game
1. Open `index.html` directly in a browser (no build required)
2. Test on desktop with keyboard and mouse
3. Test mobile by toggling device mode in DevTools or using physical device
4. Check all three difficulty levels via the start screen buttons

### Tweaking Game Balance
- **Enemy difficulty**: Modify `DIFFICULTIES` object (spawnBase, enemySpeed, enemyShootChance, lives, bombs)
- **Boss health**: Adjust `baseHp` in `spawnBoss()` and `hpMul` in boss designs
- **Weapon balance**: Edit fireWeapon() function spread/lanes and fire delay constants
- **Enemy types**: Add to `ENEMY_TYPES` object with hp, speed multiplier, and color

### Adding New Content
- **New boss**: Add to `BOSSES` array with colors and hpMul, then add pattern array to `BOSS_PATTERNS`
- **New enemy type**: Add to `ENEMY_TYPES` with stats, then filter by `minStage`
- **New theme**: Add to `THEMES` array with sky gradient colors
- **New SFX**: Add function to `sfx` object using `beep()` or `noiseBurst()` helper functions

### Sound Design
- **Beep function**: `beep(freq, duration, type, volume, opts)` where opts can include:
  - `sweepTo`: frequency ramp (exponential)
  - `filterFreq`/`filterType`: biquad filter
  - `pan`: stereo panning (-1 to 1)
  - `delay`: attack delay
- **Noise burst**: `noiseBurst(duration, volume, opts)` for explosion/impact sounds
- **Helper**: `panFor(x)` converts screen X position to stereo pan value

### Collision Detection
- Uses AABB (axis-aligned bounding box) via `rectsOverlap(a, b)` function
- Check: `Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2`

## Code Organization

The JavaScript code flows roughly as:
1. **Setup** (~line 423-867): Canvas, DOM elements, constants, event listeners, input handling
2. **Audio** (~line 542-676): Web Audio context, synthesis functions, SFX dictionary
3. **Rendering** (~line 680-1639): Draw functions for planes, bosses, backgrounds, particles
4. **Game Logic** (~line 863-1439): Update loop, collision, spawning, scoring, boss patterns
5. **Main Loop** (~line 1640-1666): requestAnimationFrame driver

## Known Patterns

- **Invulnerability frames**: 1.5 second window after taking damage
- **Screen dimensions**: 480×720 (portrait, 2:3 aspect)
- **Frame counter**: Incremented every update, used for animations/timing
- **Time delta**: Clamped to 50ms max to prevent jumps on lag spikes
- **Boss telegraph**: 22 frames (≈370ms at 60fps) of visual warning before attack

## Viewport Handling

The game uses `safe-area-inset-*` CSS env() variables for notched/rounded corner devices. HUD elements are positioned relative to safe area insets.
