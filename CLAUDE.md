# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: Sky Fighter

A Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The player controls a fighter jet, collects weapons, avoids enemy fire, and defeats stage bosses. All game logic, rendering, audio synthesis, and UI are contained in a single `index.html` file (~1700 lines).

## Game Mechanics

### Core Loop
- **Spawn System**: Enemies spawn at intervals that vary per difficulty, with spawn rate decreasing as player destroys enemies (tracking kill count)
- **Difficulty Tiers**: 3 difficulties (easy/normal/hard) configured via `DIFFICULTIES` constant
- **Stages**: 6 progressive stages, each with unique visual theme from `THEMES` array; player must eliminate 18 enemies (`KILLS_PER_STAGE`) to advance
- **Boss Encounters**: One boss per stage from `BOSSES` array; defeated boss triggers stage clear and advances to next stage
- **Lives/Bombs**: Configurable per difficulty; taking damage or getting hit by enemy fire costs a life; bombs provide screen-wide damage with invincibility frames

### Player Mechanics
- **Weapon System**: 3 weapon types (Vulcan/laser/missile) that spawn as power-ups; each has 3 upgrade levels; weapon selection persists during stage
- **Scoring**: Enemy kills + destruction combo multiplier (increases for consecutive hits without taking damage, resets on player hit)
- **Movement**: Constrained to bottom half of screen; collision detected via circular distance

### Enemy Types
Defined in `ENEMY_TYPES`:
- **Grunt**: Basic red enemy, 1 HP, spawns from stage 1
- **Interceptor**: Faster orange enemy, 1 HP, weaves side-to-side, spawns from stage 1  
- **Tank**: Heavy purple enemy, 4 HP, slow, spawns from stage 2+
Each type has distinct visuals, speeds, shoot chances, and score values

## Architecture

### File Structure
Single-file monolithic application:
- **HTML markup** (lines 361-420): Static UI elements—canvas, HUD display, overlay, touch controls
- **CSS** (lines 10-358): Dark sci-fi theme; responsive design with clamp() for mobile scaling; animations for combat feedback (flash, vignette, combo pop)
- **JavaScript** (lines 422+): IIFE-wrapped game code in strict mode

### Game State
Global state variables manage:
- `bestScore`, `muted`, `difficulty`, `reducedMotion` (persisted to localStorage)
- Player object: position, velocity, health/lives, bomb count, current weapon, weapon levels
- Arrays of active enemies, bullets, explosions, and particles
- Combo multiplier and stage/boss tracking

### Key Subsystems

#### Audio System (No External Files)
- WebAudio API for procedural synthesis via `beep()` (oscillators + filters) and `noiseBurst()` (random noise + envelope)
- Spatial audio: `panFor()` pans sounds left/right based on screen position
- SFX triggered per action: shoot, laser, enemy explosions (grunt/interceptor/tank variants), hit, bomb, powerup, boss telegraph, stage clear, game over
- Muted state stored in localStorage

#### Rendering
- **Canvas-based 2D rendering** drawn each frame
- **Player sprite**: Procedurally drawn triangle with dynamic rotation/color
- **Enemy sprites**: Procedurally drawn based on type (rectangles + accents for visual variety)
- **Boss sprites**: Multi-part procedural shapes (hull, wings, core) with type-specific colors
- **Particle system**: Explosions and visual feedback trails
- **Adaptive quality system**: Tracks frame timing and reduces visual fidelity (fewer particles, lower draw frequency) if frame rate drops below target

#### Input
- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move to steer, click to fire
- **Touch**: Virtual joystick on left, action buttons on right (fire/bomb); drag to move in joystick zone
- Touch controls conditionally enabled based on device detection

#### Accessibility
- Screen reader live region (`#srLive`) for game state announcements
- `prefers-reduced-motion` media query detection with toggle button
- Reduces animations, screen shake, and flashes when enabled
- Color contrast designed for dark theme

### Data Configuration

| Constant | Purpose |
|----------|---------|
| `DIFFICULTIES` | Spawn rates, enemy speed/accuracy, lives, bombs, boss HP scaling |
| `THEMES` | 6 stage backgrounds (sky gradients, cloud/sun colors, names) |
| `ENEMY_TYPES` | Enemy definitions: dimensions, HP, speed multiplier, shoot chance, score |
| `BOSSES` | 6 boss types with visual colors (hull/wing/core), name, HP multiplier |

## Development

### Running the Game
Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge). No build step required.

**Supported Platforms:**
- Desktop (keyboard + mouse)
- Mobile/tablet (touch controls + accelerometer optional for tilt aim)
- All modern browsers with:
  - HTML5 Canvas
  - WebAudio API
  - ES6 JavaScript
  - LocalStorage

### Testing Changes
1. Edit `index.html` directly
2. Refresh browser tab
3. Test the specific feature/level:
   - Use difficulty buttons in main menu to test different spawn rates
   - Play through multiple stages to verify stage advancement logic
   - Manually adjust `KILLS_PER_STAGE` constant to speed up testing
   - Check browser DevTools console for any errors during gameplay

### Common Edits

#### Tweaking Difficulty/Balance
- **Enemy spawn rate**: Edit `DIFFICULTIES.{easy,normal,hard}.{spawnBase,spawnMin}`
- **Enemy behavior**: Edit `ENEMY_TYPES.{type}.{speedMul,shootChance}`
- **Boss health**: Edit `BOSSES[n].hpMul` or `DIFFICULTIES.{diff}.bossHpMul`
- **Player health/bombs**: Edit `DIFFICULTIES.{diff}.{lives,bombs}`

#### Modifying Visuals
- **Stage backgrounds**: Edit `THEMES` array (sky gradient colors, cloud/sun RGB)
- **Player/enemy colors**: Edit `ENEMY_TYPES.{type}.{color,dark}` or procedural draw functions
- **Boss sprites**: Edit boss drawing code in rendering loop or boss color constants in `BOSSES`

#### Adjusting Audio
- **Sound volumes**: Edit `vol` parameter in `sfx` object entries
- **Sound pitch/tone**: Tweak `freq` or `sweepTo` values in beep calls
- **Disable audio**: Set `muted = true` or `audioCtx = null`

#### Adding New Features
- **New weapon type**: Add entry to weapon array in state, update damage calculation, add sfx
- **New enemy type**: Add to `ENEMY_TYPES`, implement draw/collision logic, spawn in enemy spawn function
- **New stage mechanic**: Modify stage advancement logic or add conditional difficulty multipliers per stage

### Performance Notes
- Adaptive quality system is built-in; it automatically reduces particle density and draw frequency if frame rate dips
- Monitor `quality` variable (0=full, 1=medium, 2=low) for performance state
- Particle count grows with explosions; large bomb explosions may cause temporary frame drops
- WebAudio synthesis is CPU-light; audio context resumed on user interaction

### Browser Compatibility
- Requires ES6 support (arrow functions, const/let, template literals, destructuring)
- WebAudio API required for sound
- LocalStorage required for persistence (falls back to defaults if unavailable)
- Touch events tested on iOS/Android
- Graceful degradation: game runs without sound if WebAudio unavailable; touch controls auto-disable on desktop

## Known Patterns & Idioms

1. **State Persistence**: User preferences (best score, mute state, difficulty, reduced-motion) stored in localStorage with keys prefixed `skyfighter.`
2. **Event Flow**: Game paused on blur/resume on focus to avoid jarring re-entry
3. **Combo Mechanics**: Combo multiplier increments on hit, resets on player damage; displayed in UI and used in score calculation
4. **Procedural Rendering**: All graphics drawn via Canvas 2D API (no images/sprites); enables lightweight distribution and dynamic theming
5. **Virtual Joystick**: Touch movement mapped to player x-velocity; action zone buttons for fire/bomb discrete actions
