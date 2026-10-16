# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML file. The game features multiple difficulty levels, boss encounters, weapon systems, and procedural audio synthesis using the Web Audio API.

## Running the Game

The game requires no build process. Simply open `index.html` in a web browser to run it.

- **Desktop**: Open `index.html` in Chrome, Firefox, Safari, or Edge
- **Mobile**: Responsive design works on all screen sizes; touch controls automatically enable on touch devices
- **No external assets**: All graphics are drawn with Canvas, all sounds are synthesized with Web Audio API

## Architecture

### Single HTML File Structure

All code is contained in `index.html` (~1700 lines) with three sections:

1. **HTML Markup** (lines 1-421)
   - Canvas element for game rendering
   - UI overlays (start screen, game over, pause)
   - HUD elements (score, health, weapons, boss bar)
   - Touch control elements (virtual joystick, fire button)

2. **CSS Styling** (embedded in `<style>` tags)
   - Responsive design using `clamp()` for dynamic scaling
   - Dark theme with gradient backgrounds
   - Animations for UI feedback (card entrance, button presses, combo display)
   - Safe area insets for mobile notches

3. **JavaScript Game Logic** (lines 422+, wrapped in IIFE)
   - All game systems and logic contained in a single IIFE to avoid global scope pollution

### Core Game Systems

**Game State & Lifecycle**
- `gameState`: Tracks whether game is 'idle' (menu), 'playing', or 'paused'
- `updateGame()`: Main game loop that updates all entities and physics
- `render()`: Renders canvas graphics each frame

**Player System**
- `player`: Position, health, weapon state, bomb count
- `playerShoot()`: Fires current weapon type
- `playerBomb()`: Triggers bomb (limited uses per stage)
- Weapon types: standard shot, spread shot, laser (unlocked progressively or with pickups)

**Enemy System**
- Enemy types in `ENEMY_TYPES`: grunt, interceptor, tank (each with different HP, speed, score)
- Enemy spawning based on difficulty settings with adaptive rates
- Enemy AI: moves down-screen, shoots at player, despawns at bottom
- Collision detection for player bullets vs enemies

**Boss System**
- `BOSSES` array: 6 unique boss definitions with unique colors and HP multipliers
- Boss appears after killing `KILLS_PER_STAGE` enemies
- Boss-specific behavior: complex movement patterns, special attacks
- Boss telegraph sounds before attacks

**Difficulty System** (`DIFFICULTIES` object)
- Three presets: easy, normal, hard
- Configures: enemy spawn rates, speeds, shoot accuracy, lives, bombs, boss HP scaling
- Persisted in localStorage

**Scoring & Progression**
- Score multiplier increases with consecutive hits (combo system)
- Stage progression (current stage = enemies killed / `KILLS_PER_STAGE`)
- Best score persisted in localStorage
- Stage visual feedback with `stageBanner` animation

**Audio System** (Web Audio API)
- No external audio files; all sounds synthesized with oscillators and noise buffers
- `beep()`: Sine/square/sawtooth oscillators with frequency sweeps and filters
- `noiseBurst()`: White noise filtered for explosion/impact sounds
- `sfx` object: Named sound effects (shoot, laser, explode variants, bomb, powerup, etc.)
- Spatial audio: Pan sounds based on X position for left/right positioning
- Performance: Sounds only play if not muted and audioContext is initialized

**Rendering & Visual Effects**
- Canvas-based rendering with `ctx.fillRect()`, `ctx.drawImage()`, bezier paths for complex shapes
- Themes system: 6 visual themes with different sky gradients, cloud colors, sun positions
- Visual feedback: flash effects on hit/bomb, vignette pulse when critical health, combo pop animation
- Adaptive quality: Performance monitoring adjusts enemy/particle counts to maintain 60 FPS

**Input Handling**
- Keyboard: Arrow keys for movement, spacebar for shoot, Z for bomb, P for pause
- Mouse: Position tracked for aiming feedback
- Touch: Auto-activates joystick UI, virtual fire/bomb buttons
- `touchControls` element conditionally shown on touch devices

**UI & Menus**
- Overlay system: Start screen, pause screen, game-over screen
- Difficulty selection buttons
- Controls hint display with platform detection
- Motion reduction toggle for accessibility
- Screen reader live region (`srLive`) for announcements

**Storage Keys**
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Accessibility preference

## Key Constants & Configurations

- `KILLS_PER_STAGE`: Number of enemies to kill before boss spawns (18)
- `THEMES`: Array of 6 visual environments with sky gradients
- `DIFFICULTIES`: Object with settings per difficulty level
- `BOSSES`: Array of 6 boss types with colors and HP scaling
- `ENEMY_TYPES`: Enemy type definitions (grunt, interceptor, tank)

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update spawn logic in `spawnEnemy()` if needed
3. Add explosion SFX variant if unique: add to `sfx` object and `EXPLODE_SFX` map

### Adding a New Boss
1. Add object to `BOSSES` array with: `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Boss rendering is generic (uses hull/wing/core colors), no special AI needed unless significant behavior desired
3. Adjust `hpMul` to balance difficulty relative to other bosses

### Adding a Sound Effect
1. Create function in `sfx` object using `beep()` and/or `noiseBurst()`
2. `beep()` params: frequency, duration, wave type, volume, options object
3. Options: `sweepTo` (end frequency), `filterType`/`filterFreq` (lowpass/highpass/bandpass), `pan`, `delay`
4. Call via `sfx.effectName()` throughout game code

### Adjusting Difficulty Balance
1. Modify values in `DIFFICULTIES` object:
   - Lower `spawnBase`/`spawnMin` = faster enemy spawning
   - Higher `enemySpeed` = faster enemy movement
   - Higher `enemyShootChance` = more enemy bullets
   - Reduce `lives`/`bombs` for harder difficulty
   - Adjust `bossHpMul` to scale boss health per difficulty

### Adding a Visual Theme
1. Add object to `THEMES` array with: `sky` (3-color gradient array), `cloud`, `sun`, `name`
2. Colors are either hex strings or RGB values as comma-separated strings
3. Theme selected randomly each stage via `themes[~~(Math.random() * themes.length)]`

## Performance Considerations

- **Adaptive Quality**: Game monitors frame times and adjusts `quality` level (0-2) to maintain ~60 FPS
  - Higher quality = more particles/effects
  - Lower quality = fewer particles/effects
- **Canvas Context**: Reused `ctx` object throughout to avoid repeated `getContext()` calls
- **Entity Pooling**: Could be implemented for bullets/enemies if needed for optimization
- **Audio**: Web Audio synthesis can be CPU-intensive; mute option always available

## Accessibility Features

- **Keyboard Navigation**: Full game playable with keyboard controls
- **Touch Support**: Virtual joystick and buttons for mobile
- **Motion Reduction**: Accessible toggle to disable animations/flash effects
- **Screen Reader**: Live region (`srLive`) announces important events
- **High Contrast**: Dark theme with bright accent colors

## Browser Compatibility

- Requires: Canvas, Web Audio API, localStorage
- Tested on: Modern Chrome, Firefox, Safari, Edge
- Mobile: Responsive design works on all screen sizes
- Touch: Joystick UI activates automatically on touch devices
