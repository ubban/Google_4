# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application (index.html). The game features canvas-based rendering, procedurally generated sound effects, multiple difficulty levels, boss battles, and a weapon upgrade system.

## Quick Start

- **Play the game**: Open `index.html` in a modern browser
- **No build step required**: The game runs directly from the HTML file
- **Development**: All code is in `index.html` (CSS in `<style>` block, JavaScript in `<script>` block)

## Project Structure

The entire game is in a single `index.html` file organized as:

1. **HTML Markup** (lines 1-420): DOM structure with canvas and overlay UI elements
2. **CSS Styling** (lines 10-358): Responsive design, animations, HUD elements, overlay screens
3. **JavaScript Game Code** (lines 422+): Core game engine and mechanics

## High-Level Architecture

### Game Loop & State

- **Canvas dimensions**: 480x720 (BASE_W × BASE_H), responsive scaling up to 1.6x
- **Frame-based update**: Uses `requestAnimationFrame` for timing
- **Game states**: Menu (overlay visible), Playing, Paused, Game Over
- **FPS target**: 60 FPS with adaptive quality scaling (quality: 0=full, 1=medium, 2=low)

### Core Systems

#### 1. **Rendering Pipeline**
- Canvas 2D context (`ctx`) for all visuals
- Background: Gradient sky (changes per stage theme)
- Layers rendered in order: sky → enemies → player → projectiles → effects → UI
- Theme system (6 sky gradients): Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge

#### 2. **Input Handling**
- **Keyboard**: Arrow keys / WASD for movement, Space/X/Shift for firing, X/Shift for bombs, P/Esc to pause
- **Mouse**: Move to steer, click to fire
- **Touch**: Virtual joystick on left, FIRE/BOMB buttons on right (only shown on touch devices)
- Dual-input support: keyboard and mouse can be used simultaneously

#### 3. **Audio System**
- **Web Audio API** for all sound: no external audio files
- Procedurally generated SFX: beep() for tones, noiseBurst() for noise
- Each event has unique sound: shooting, explosions (3 types), hits, bombs, power-ups, boss events, UI clicks
- Master gain/compressor chain for mixing
- Mutable via UI button, setting persisted to localStorage

#### 4. **Game Entities**
- **Player**: Single controllable craft, 48×48 px, position (px, py)
- **Enemies**: Three types (ENEMY_TYPES):
  - `grunt` (34×34): Basic red enemy, hp=1
  - `interceptor` (26×26): Fast weaving red enemy, hp=1
  - `tank` (46×46): Slow purple heavy unit, hp=4, appears stage 2+
- **Bosses**: 6 named bosses (BOSSES array) with custom colors, hp multipliers, and attack patterns
- **Projectiles**: Player bullets and enemy bullets, simple position/velocity

#### 5. **Game Progression**
- **Stages**: Increase enemy density and boss difficulty (KILLS_PER_STAGE = 18 kills advance stage)
- **Difficulty levels** (DIFFICULTIES object):
  - Easy: lower spawn rate, slower enemies, 4 lives, 3 bombs
  - Normal: balanced, 3 lives, 2 bombs
  - Hard: dense enemies, fast, 3 lives, 2 bombs
- **Score system**: Base score + multiplier (x) based on consecutive kills without being hit
- **Weapons**: Vulcan (default), Laser, Plasma, Missiles (cycle through power-ups)
- **Weapon levels**: 1-5 (higher = faster fire rate, more projectiles)

#### 6. **Persistent State**
- localStorage keys (STORAGE_* constants):
  - `skyfighter.bestScore`: Best score achieved
  - `skyfighter.muted`: Audio mute state (0 or 1)
  - `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
  - `skyfighter.reducedMotion`: Accessibility preference (0 or 1)
- Best score displayed on title screen, compared to final score

#### 7. **Accessibility Features**
- **Reduced motion toggle**: Disables screen shake, flash effects, pulsing animations
- **Screen reader support**: `#srLive` element for announcements (aria-live="polite")
- **Prefers-reduced-motion detection**: Auto-enables reduced motion if OS preference set
- **Inclusive color/visual design**: Distinct enemy colors, clear UI

#### 8. **Performance Optimization**
- **Adaptive quality**: Tracks frame times (45-frame rolling average) and reduces quality if avg > 1/40s
- **Quality modes**:
  - 0 (full): All effects, precise rendering
  - 1 (medium): Reduce particle count or effect complexity
  - 2 (low): Minimal effects, simplest rendering
- Frame time tracking: `frameTimes` array, `trackPerf(dt)` updates quality

### Key Constants & Configuration

- `KILLS_PER_STAGE`: 18 (enemies killed to advance stage)
- `ENEMY_TYPES`: Object defining each enemy class (size, hp, speed, score, spawning rule)
- `BOSSES`: Array of 6 boss definitions with visual/difficulty properties
- `DIFFICULTIES`: Object with spawn rates, enemy stats, starting lives/bombs per difficulty
- `THEMES`: 6 sky gradient themes with color palettes and names
- Base canvas: 480×720; scales up to 1.6× max based on viewport
- Touch device auto-detection: `isTouchDevice` boolean

## Important Implementation Notes

### Canvas Rendering
- All drawing uses 2D context directly (no libraries)
- Player and enemies drawn procedurally with `fillRect` / `fillStyle` / shapes
- Effects (vignette, flash, combo text) use CSS overlays where possible, canvas for game objects
- Clip regions and z-index managed via layer rendering order

### Game Loop Flow
```
requestAnimationFrame → update(dt) → render() → repeat
```
- `dt` is delta time; adaptive quality affects update/render complexity
- After render, pause check and input processing occur

### Enemy Spawning
- Spawn timer decreases each frame; when it reaches 0, spawn random enemy and reset timer
- Spawn interval ranges from `spawnBase` to `spawnMin` based on difficulty and stage
- Enemy type unlocking: grunts always, interceptors stage 1+, tanks stage 2+

### Boss Battles
- Trigger when `killCount >= KILLS_PER_STAGE` (stage completion)
- Boss health scales by `bossHpMul` and difficulty multiplier
- Boss pattern: telegraphed attacks, projectile spray
- Victory: score bonus, clear sound, stage banner, next stage begins

### Weapon Mechanics
- Power-up drops from killed grunts/interceptors increase level
- Each level (1-5) increases fire rate and sometimes projectile count/spread
- Weapon cycling: 4 weapon types (Vulcan, Laser, Plasma, Missiles) rotate on each weapon power-up

### Combo System
- Multiplier (x) increases on consecutive kills without taking damage
- Resets to 1 when player is hit
- Visual feedback: combo pop text in center, HUD displays current multiplier
- Affects final score: `score = baseScore * multiplier`

## Development Notes

### Adding New Features
- **New weapon**: Add to weapon array/cycle logic; adjust fire patterns
- **New enemy type**: Add to ENEMY_TYPES and BOSSES with color/hp/behavior
- **New SFX**: Add function to `sfx` object using `beep()` or `noiseBurst()`
- **Visual effects**: Add CSS animations or canvas drawing in render loop
- **Difficulty balance**: Adjust DIFFICULTIES object spawn rates and multipliers

### Testing in Browser
- Open `index.html` in Chrome, Firefox, Safari, or Edge
- Open DevTools (F12) and check Console for any errors
- Test at multiple viewport sizes (mobile, tablet, desktop)
- Test all three difficulty levels
- Listen for audio cues and check mute functionality

### Performance Profiling
- Frame time tracking runs continuously; check `quality` variable to see adaptive mode
- Disable effects (reduced motion) to isolate rendering vs. input cost
- Use DevTools Performance tab to profile frame timing

### Accessibility Testing
- Enable "Reduced Motion" via button; verify animations stop/minimize
- Test keyboard-only gameplay (arrow keys + space + x)
- Test touch controls on mobile
- Verify screen reader (VoiceOver/NVDA) announces score updates via `srLive`

## Known Limitations & Future Improvements

- Single-file structure: No modular split; all code in one HTML file
- No persistent gameplay data: Only best score saved, not full game history
- No multiplayer: Single-player only
- Boss patterns: Hard-coded; no procedural generation
- Mobile controls: Basic touch; no advanced gesture recognition
- Audio: Procedural only; no imported music or sampled sounds

## Browser Compatibility

- Modern browsers (Chrome 80+, Firefox 75+, Safari 13+, Edge 80+)
- Requires: Canvas 2D, Web Audio API, localStorage, requestAnimationFrame
- Mobile: iOS Safari, Android Chrome/Firefox
- No polyfills included; game uses native APIs

