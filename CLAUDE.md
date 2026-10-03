# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game. It's a **single-file HTML application** (~1669 lines) containing all game logic, rendering, audio synthesis, and UI. No build process, dependencies, or external assets are required—the game runs directly in any modern browser.

## Architecture

The game is organized as a self-contained IIFE (Immediately Invoked Function Expression) with clear logical sections:

### 1. **Initialization & DOM Elements**
- Captures HTML canvas, buttons, and UI elements
- Initializes game constants and localStorage

### 2. **Game Configuration**
- **`DIFFICULTIES`** object: Defines `easy`, `normal`, `hard` modes with lives, bombs, spawning rates, enemy types per stage
- **`WEAPON_TYPES`**: laser and vulcan configurations (spread, cooldown, pierce)
- **`ENEMY_TYPES`**: grunt, interceptor, tank with HP, speed, score, visual properties, stage availability
- **`BOSSES`** array: 6 boss definitions (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with color schemes and HP multipliers

### 3. **Audio System (Synthetic)**
Uses **Web Audio API**—no external sound files. Key functions:
- `beep(freq, dur, type, vol, opts)`: Generates tones with optional frequency sweep, filters, panning
- `noiseBurst(dur, vol, opts)`: White-noise bursts for explosions, impacts
- `sfx` object: Pre-defined sound effects (shoot, laser, impact, bomb, etc.)

The audio context is lazily initialized on first user input (required for browser autoplay policy).

### 4. **Canvas Rendering Pipeline**
- **`drawPlane(x, y, w, h, color, darkColor, flame)`**: Draws player/enemy ships with optional flame effect
- **`drawBossShip(boss)`**: Complex multi-layer boss rendering (hull, wings, core, damage vignette)
- **`draw()`**: Main render function called each frame—draws background, player, bullets, enemies, boss, effects, UI overlays

Screen shake is applied via canvas translation; vignette effect intensifies near boss defeat.

### 5. **Game Loop & Time**
- **`loop(now)`**: `requestAnimationFrame` callback; computes delta-time, calls `update()` then `draw()`
- Delta-time clamped to 50ms to prevent large jumps on frame drops
- Performance tracking stored in `perf` array for debugging

### 6. **Game State & Reset**
Core state variables:
- `player`: position, speed, weapon type, level, cooldown timers, invulnerability frames
- `bullets`, `enemies`, `enemyBullets`: pooled arrays
- `particles`, `powerUps`, `boss`: dynamic objects
- `score`, `lives`, `kills`, `stage`, `chain`, `multiplier`: game progress metrics

`resetGame()` reinitializes all state based on selected difficulty.

### 7. **Update Logic**
`update(dt)` is the heart of the game:
- **Player movement**: Read input state (keyboard, touch, mouse), clamp to canvas bounds
- **Firing**: Weapon cooldown decrement; weapon-specific bullet spawning (vulcan has spread; laser has pierce & multiple lanes by level)
- **Collision detection**: AABB (`rectsOverlap`) checks player↔enemy bullets, bullets↔enemies, bullets↔boss
- **Enemy spawning**: Timer-based; difficulty/stage determines spawn rates and available enemy types
- **Boss logic**: State machine (entering → attack patterns → defeat); pattern includes bullet spreads, movement arcs, player telegraphs
- **Power-up handling**: Collect for extra lives, bombs, weapon upgrades
- **Chain combo system**: Resets if player gets hit; multiplier increases with consecutive kills

### 8. **Particle System**
- Generic particle objects with position, velocity, color, lifetime
- Used for explosions, puffs, visual feedback
- Rendered with alpha fade and optional glow

### 9. **Input Handling**
- **Keyboard**: Arrow keys or WASD for movement; Space for bomb (fallback if touch unavailable)
- **Touch**: Direct pointer follows finger; two-finger tap triggers bomb; velocity-based on sustained drag
- **Mouse**: Follows cursor; click to start and set firing state
- Canvas-relative input is normalized via `canvasPointFromEvent()` to handle window scaling

### 10. **Settings & Persistence**
- `localStorage` stores: best score, mute toggle, difficulty, reduced-motion preference
- Reduced-motion mode disables camera shake and screen flashes if user has OS preference set
- Difficulty selector regenerates game state but preserves high-score

## Key Mechanics for Modification

### Weapon System
Modify **`WEAPON_TYPES`** to adjust cooldown, pierce rating, or firing pattern. Update **`fireWeapon()`** to implement new spread angles or bullet behaviors.

### Enemy Types & Progression
- Add to **`ENEMY_TYPES`** to define new enemies (adjust `minStage` to control availability)
- Enemy spawning rate controlled by `d.spawnRate` in `DIFFICULTIES`
- AI patterns are hardcoded per enemy class; extend `update()` for new movement or attack patterns

### Boss Patterns
Each boss has an `attackTimer` and `patternIndex`. Boss attack functions (`bossBulletSpread`, `bossBulletSpinAttack`, etc.) define bullet patterns. Add new patterns and cycle through them by incrementing `patternIndex`.

### Visual Tuning
- Colors: found in `ENEMY_TYPES`, `BOSSES`, gradient definitions in `draw()`
- Size: width/height fields in entity definitions
- Animation speed: keyframe durations in CSS or timing constants in `update()`

### Difficulty Scaling
All difficulty-based values live in **`DIFFICULTIES[difficulty]`**. Tweak `lives`, `bombs`, `spawnRate`, `enemySpeedMul`, `enemyHPMul`, `bossSpeedup`.

## Testing & Debugging

Since this is a single HTML file, testing is manual:
1. Open `index.html` directly in a browser (no server required)
2. Use browser DevTools console to inspect `score`, `player`, `enemies`, `boss`, `frame`, `chain` during gameplay
3. Add `console.log()` in `update()` or `draw()` for frame-by-frame inspection
4. Performance profiling: access `perf` array which tracks delta-times and rendering stats

## Development Workflow

1. **Edit** `index.html` in any text editor
2. **Save** and reload browser tab (or enable auto-refresh)
3. **Test** game mechanics: play a few rounds, check edge cases (bombs, chain resets, stage transitions, boss defeat)
4. **Debug** with DevTools; inspect game state objects or add temporary console output

## Common Tasks

### Add a new weapon type
1. Add entry to `WEAPON_TYPES` with cooldown and pierce values
2. Update `fireWeapon()` to spawn bullets with new pattern
3. Add sound effect to `sfx` object if needed

### Add a new boss
1. Add to `BOSSES` array with name, colors, HP multiplier
2. Define a new boss attack function (e.g., `bossBulletSpinAttack`)
3. Call it from the boss state machine in `update()`

### Adjust difficulty curve
1. Modify `DIFFICULTIES[difficulty]` for lives, bombs, spawn rates, speed/HP multipliers
2. Or adjust stage progression thresholds in `update()` where stage increments

### Change visual/audio style
- Audio: modify beep/noise parameters or add new `sfx` functions
- Visuals: update color constants, adjust animation timings, modify `draw()` rendering calls
- Effects: screen shake, vignette, flash overlays are controlled in `update()` and CSS keyframes

## Performance Considerations

- **Particle count**: Capped by quality setting; lower quality = fewer particles on explosion
- **Enemy limit**: No hard cap; very high enemy counts will cause lag
- **Frame rate**: Game maintains 60 FPS target; delta-time clamping prevents runaway physics
- **Audio context**: Lazily initialized; first sound triggers setup delay (typically <50ms)

## Browser Compatibility

Requires:
- Canvas 2D API
- Web Audio API (optional; game degrades gracefully if unavailable)
- Touch Events API (for mobile)
- LocalStorage (for persistence)
- Modern CSS (transforms, animations, gradients)

Tested on Chrome, Firefox, Safari, Edge. Touch controls work on iOS/Android.
