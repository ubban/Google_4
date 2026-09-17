# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML/CSS/JavaScript web application. The game features:
- Player-controlled aircraft that scrolls vertically through stages
- Multiple enemy types with distinct behaviors
- Boss encounters at the end of each stage
- Weapon upgrade system (Vulcan/Laser with power levels)
- Smart bomb mechanics
- Dynamic difficulty levels (Easy/Normal/Hard)
- Synthesized sound effects (no external audio assets)
- Touch and keyboard controls with responsive canvas scaling
- Local storage persistence for high scores, settings, and preferences

## Code Structure (All in `index.html`)

The codebase is organized into logical sections within a single IIFE:

### Game Constants
- **DIFFICULTIES**: Defines spawn rates, enemy speed, health multipliers, and starting resources for each difficulty
- **KILLS_PER_STAGE**: Number of enemy kills needed to trigger boss fight (18)
- **THEMES**: Sky gradient palettes for visual variety (6 themes rotate through stages)
- **ENEMY_TYPES**: grunt, interceptor, tank — each with HP, speed multiplier, color, score value, and minimum stage
- **BOSSES**: 6 unique boss designs (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) with hull colors and HP multipliers

### Core Systems
1. **Canvas & Rendering**: BASE_W=480, BASE_H=720. Scales to fit viewport while maintaining aspect ratio
2. **Audio (WebAudio API)**: Synthesized SFX via `beep()` (oscillator) and `noiseBurst()` functions. Includes frequency sweeps, filtering, and spatial panning
3. **Input**: 
   - Keyboard: arrows/WASD for movement, space to fire, X/Shift for bombs, P/Esc to pause
   - Mouse: move to steer, click to fire
   - Touch: left 55% of screen is joystick zone, right 45% has fire/bomb buttons
4. **Game State**: Tracked in globals (player, bullets, enemies, score, lives, stage, chain)
5. **Rendering Pipeline**: HUD (stats, hearts, bombs), boss bar, stage banners, combo popups, particles, main game canvas

### Key Game Mechanics
- **Spawn System**: Enemies spawn based on stage, with spawn rate controlled by difficulty
- **Combat**: Player fires bullets; enemies counter-fire. Collisions with bullets deal damage; collisions with enemy bullets hurt player
- **Weapons**: Vulcan (spread shot, 3 rounds) vs Laser (straight shot, single). Power ups rotate weapons and increase level
- **Bombs**: Smart bombs damage all on-screen enemies; limited stock, refillable via powerups
- **Chain/Multiplier**: Score multiplier increases with consecutive kills without taking damage (resets on hit)
- **Bosses**: Triggered after 18 kills per stage. Boss moves in attack patterns, telegraphs shots, and is immune to small-arms fire

## Development Commands

Since this is a static HTML file with no build process:

- **Open in browser**: Open `index.html` directly in any modern browser
- **Test locally**: Use a local HTTP server (e.g., `python -m http.server`, `npx http-server`)
- **Development server** (if you need hot reload): Set up a simple watch script or use an IDE's preview feature

No linting, testing, or build tools are configured.

## Architecture Tips for Future Work

### Adding Content (Enemies/Bosses)
1. Add entry to `ENEMY_TYPES` (or `BOSSES`) with visual and behavioral parameters
2. Update `KILLS_PER_STAGE` or boss appearance logic if adding new spawn rules
3. Enemies are rendered as simple rectangles with dark/light shading; bosses are hand-drawn 2D shapes

### Modifying Gameplay Balance
- **Difficulty curves**: Adjust `DIFFICULTIES` object (spawn rates, enemy speed, lives, bombs)
- **Stage progression**: Modify enemy spawn conditions in `spawnEnemy()` logic and `minStage` in enemy definitions
- **Score/Rewards**: Adjust `score` values in enemy definitions or multiplier logic in collision handlers

### Audio System Notes
- All SFX are synthesized in real-time; no audio files to manage
- `beep()` creates tonal sounds; `noiseBurst()` creates percussion/explosions
- Uses `AudioContext` with dynamic compressor for volume control and spatial panning for stereo effects
- Respects user's reduced-motion and mute preferences

### Input & Controls
- Mouse steering is always active on desktop but yields to keyboard/joystick
- Touch joystick spawns only when dragging; stays hidden when not in use
- Pause toggle works from keyboard, pause button, or settings (not during overlays)

### Performance Optimization
- Adaptive quality system (`quality` variable) throttles rendering based on frame time
- Particles and projectiles are pooled (bullets, particles reuse from arrays)
- Canvas pixel rendering only; no DOM-heavy UI during gameplay (HUD uses CSS overlays)

### Storage Keys
- `skyfighter.bestScore`: High score persistence
- `skyfighter.muted`: Audio mute preference
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference for motion reduction

## Design Patterns

- **State machine**: `running`/`paused`/`gameOver` flags control main loop flow
- **Object pools**: Bullets, particles reuse from arrays to reduce GC pressure
- **Entity model**: Player, enemies, bullets, particles are plain objects with x, y, w, h properties
- **Direct canvas rendering**: No sprite sheets; shapes drawn programmatically with gradients and shadows
- **Synthesized audio**: No external assets; all SFX generated via WebAudio API

## Common Tasks

### Debug Gameplay
- Open browser DevTools Console
- Pause the game (P key), inspect `player`, `enemies`, `boss` in console
- Adjust `DIFFICULTIES` or enemy stats, reload to test

### Add New Stage Theme
- Add object to `THEMES` array with `sky` (gradient stops), `cloud`, and `sun` color values
- Theme is selected as `THEMES[stage % THEMES.length]`

### Tweak Boss Behavior
- Boss AI is simple pattern-based movement and firing; modify in boss update/render logic
- Attack patterns defined by elapsed time and phase transitions
- Boss telegraph warnings play before major attacks

## Testing Notes

- Game works on desktop (mouse/keyboard) and mobile (touch)
- Test responsive scaling by resizing browser window or rotating device
- Verify audio works across browsers (WebAudio context may need user gesture to initialize)
- Check reduced-motion preferences (system setting or in-game toggle) to ensure animations respect user accessibility needs
