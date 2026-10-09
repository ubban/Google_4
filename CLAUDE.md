# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML5 file with embedded JavaScript and CSS. It features procedural difficulty scaling, multiple weapons, boss battles across themed stages, and full touch/keyboard/mouse input support.

## Architecture

The entire game is implemented in a single `index.html` file within an IIFE (Immediately Invoked Function Expression) for scope isolation. The architecture consists of:

### Core Subsystems

1. **Game State Management**
   - Player: position, velocity, weapon, health, invulnerability frames
   - Enemies: spawned based on difficulty/stage with type variants (grunt, interceptor, tank)
   - Bullets: player projectiles with pierce mechanics for laser weapon
   - Enemy Bullets: spawned from enemies and bosses with various patterns
   - Power-ups: weapon upgrades, extra lives, bombs
   - Boss: single large enemy per stage with complex attack patterns
   - Particles: explosion and visual effect particles

2. **Rendering Pipeline** (Canvas 2D)
   - Sky background with theme-based gradient and sun glow
   - Parallax cloud layers
   - Player/enemy ships drawn with polygonal shapes
   - Bullet and particle rendering
   - HUD overlay showing score, lives, weapon, multiplier
   - Boss health bar
   - Screen shake effects (with reduced motion support)

3. **Audio** (Web Audio API)
   - Procedural sound synthesis (no external files)
   - Beep/noise generation functions for SFX
   - Spatial panning based on screen position
   - Master gain and compression
   - Sound effects for: shooting, explosions, hits, bosses, powerups, UI

4. **Input Handling**
   - Keyboard: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
   - Mouse: Move to steer, click to fire
   - Touch: Drag joystick (left), tap FIRE button, tap BOMB button
   - Keyboard and joystick input prioritized over mouse steering

5. **Game Loop**
   - 60 FPS target via `requestAnimationFrame`
   - `update(dt)`: Game state changes (30ms max frame clamp)
   - `draw()`: Canvas rendering
   - Adaptive performance tracking: scales down particle/shadow effects if FPS drops

## Game Mechanics

### Difficulty Modes
Three difficulty presets control: enemy spawn rate, movement speed, shoot chance, starting lives, bombs, and boss HP multiplier:
- **Easy**: Slower spawns, safer mechanics, 4 lives
- **Normal**: Balanced (default), 3 lives
- **Hard**: Dense spawns, aggressive enemies, 3 lives

### Weapon System
Two weapons with 3 levels each:
- **Vulcan**: Spread shot that increases spread angle with level (up to ±0.24 radians at level 3)
- **Laser**: Piercing beam; level 3 adds multiple lanes

Power-ups cycle: weapon level 1→2→3, then toggle weapon and reset level.

### Scoring & Multiplier
- Each kill increments combo chain
- Multiplier = 1 + floor(chain/5), max 10x
- Chain breaks after 2.2 seconds without kills
- Boss kill grants 500 flat points

### Boss System
Six boss designs cycle per stage (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN). Each has:
- Unique visual design (hull colors, core, wings)
- 3 attack patterns with varying bullet formations (radial, aimed spread, columns, homing, spirals, etc.)
- Telegraph phase (22 frames) before each attack
- Attack cooldown (85 frames between patterns)
- HP multiplier applied per design and difficulty

### Stage Progression
- 18 kills per stage triggers boss spawn after 1.2s warning
- Each defeated boss advances stage, spawning next theme and boss
- Themes cycle across 6 procedural color schemes (daylight, sunset, night, storm, arctic, volcanic)

### Power-ups
Randomly spawned on enemy death:
- **Life** (6% chance): Restore 1 hp
- **Bomb** (8% chance): Restore 1 bomb charge
- **Weapon** (20% chance): Upgrade weapon level or switch weapon

## Key Code Sections

### State Initialization (line 863-910)
- `resetGame()`: Prepare canvas, player, empty arrays, reset difficulty settings
- Loads best score and difficulty from localStorage

### Input Processing (line 767-861)
- Key tracking via keydown/keyup
- Mouse steering toward target position with smooth interpolation
- Touch joystick with visual feedback; priority over mouse
- Fire and bomb buttons (Space/X/Shift keys, mouse click, touch buttons)

### Enemy Spawning & AI (line 1022-1041)
- Random spawn position at top of screen
- Type distribution based on stage (tank appears at stage 2+)
- Weaving interceptors follow sine-wave patterns
- Two shoot patterns: spread (fan shape) or aimed (toward player)

### Collision Detection (line 1069-1294)
- Rectangle overlap for all entity types
- Bullet-to-enemy: damage, pierce, explosion
- Bullet-to-boss: reduced damage (3 per hit vs 1 for enemies)
- Enemy/bullet-to-player: lose life, invulnerability frames
- Power-up collection

### Boss Update Loop (line 1411-1439)
- Horizontal patrol with boundary reversal
- Telegraph timer → attack pattern firing
- Pattern selection via design index and patternIndex counter
- Radial bullet movement (stored as dx/dy, not speed/angle)

### Drawing Functions
- `drawPlane()` (line 913): Player and enemy ship polygons with flame effect
- `drawBossShip()` (line 970): Boss with animated wings, glowing core, telegraph effect
- `drawClouds()` (line 1507): Parallax cloud layers
- `draw()` (line 1520): Full scene: sky, clouds, entities, HUD

## Development Notes

### Canvas Sizing
- Base resolution: 480×720 (portrait)
- Scales up to 1.6x max
- Responsive to window resize and orientation change

### Performance Optimization
- Quality scaling: 0 (full) → 1 (medium glows) → 2 (minimal effects)
- Particle count reduced at lower quality
- Shadow blur disabled at quality 2+
- Frame time tracking to auto-adjust quality

### Accessibility
- Reduced motion support (respects `prefers-reduced-motion` media query)
- ARIA live region for screen reader announcements
- Semantic HTML buttons with title attributes
- Keyboard shortcuts for all gameplay functions

### Storage
Keys saved to localStorage:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Explicit reduced motion preference

### To Modify

**Add a new enemy type**: Add entry to `ENEMY_TYPES` object (line 502), update `DIFFICULTIES` spawn filtering if needed.

**Add a new boss**: Extend `BOSSES` array (line 508), add attack pattern functions and entry to `BOSS_PATTERNS` array (line 1372).

**Adjust difficulty**: Tweak `DIFFICULTIES` constants (line 485) or stage progression (`KILLS_PER_STAGE` at line 491).

**Customize visuals**: Modify `THEMES` (line 493) for sky colors and clouds, color constants in drawing functions for entity colors, CSS for UI styling.

**Change audio**: Modify `sfx` object (line 624) to adjust existing sound parameters or add new SFX using `beep()` and `noiseBurst()` utilities.

## Testing

No build step or external dependencies. To test:
1. Open `index.html` in a modern browser
2. Test all three difficulty modes
3. Verify keyboard, mouse, and touch input paths
4. Check audio synthesis works (enable sound via button)
5. Verify localStorage persistence across page reloads
6. Test on mobile for touch controls and responsive canvas scaling

## Common Tasks

**Adjust enemy difficulty**: Modify `DIFFICULTIES` object parameters
**Add stage theme**: Add object to `THEMES` array with sky colors and cloud style
**New boss pattern**: Create fire function in `BOSS_PATTERNS`, add to boss design
**Change weapon balance**: Modify spread angles, pierce counts, fire rates in `fireWeapon()` or weapon type definitions
**Audio balance**: Adjust volumes in `beep()`/`noiseBurst()` calls, modify filter frequencies
