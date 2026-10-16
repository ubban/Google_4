# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:
- Procedurally spawned enemies with increasing difficulty across 6 stages
- 6 unique boss encounters with distinct visual and attack patterns
- Weapon progression (Vulcan guns → Laser) with level upgrades
- Smart bomb mechanic with screen-clearing effect
- Combo/multiplier system for chained kills
- Responsive design supporting keyboard, mouse, and touch controls
- Web Audio API-based procedural sound effects
- Adaptive performance scaling for smooth gameplay across devices
- Persistent best score tracking via localStorage

## Architecture

### Single-File Structure
All code lives in `index.html` (1669 lines). The game is entirely self-contained with no external dependencies:
- **Lines 1-400**: HTML structure and CSS styling
- **Lines 422-1669**: JavaScript game engine wrapped in an IIFE (Immediately Invoked Function Expression)

### Core Game Systems

#### 1. **Entity System**
Game state is managed through simple object properties (no classes):
- `player`: Player ship with position, velocity, weapon state, health, invulnerability frames
- `enemies`: Array of enemy objects with position, type, health, shoot cooldown
- `bullets`: Player projectiles (position, velocity, type: 'vulcan'/'laser')
- `enemyBullets`: Enemy projectiles
- `particles`: Visual effects (explosions, hits, etc.)
- `powerUps`: Weapon upgrades and health pickups
- `boss`: Active boss (null when inactive)

#### 2. **Enemy System**
Three main enemy types defined in `ENEMY_TYPES` constant:
- `grunt`: Basic red enemy (1 HP, score 10)
- `interceptor`: Fast orange enemy (1 HP, weaving pattern, score 15)
- `tank`: Purple armored enemy (4 HP, score 35, appears stage 2+)

Enemies spawn based on difficulty settings with adaptive spawn rates. Each type has its own speed multiplier and color for visual distinction.

#### 3. **Boss System**
Six procedurally designed bosses in `BOSSES` array:
- VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each has unique color scheme and health multiplier
- Boss appears after player kills `KILLS_PER_STAGE` (18) enemies
- Boss bar displays at top of screen when active

#### 4. **Weapon System**
Two weapons with level progression:
- **Vulcan**: Rapid-fire bullets with spread pattern
- **Laser**: Concentrated beam with higher damage per shot
- Weapons upgrade by collecting power-ups (visual difference in projectile size/color)
- Current weapon and level displayed in HUD

#### 5. **Scoring System**
- Base kill value: 10-35 points depending on enemy type
- Multiplier system: Chained kills (within 3 seconds) increase multiplier (1x → 6x max)
- Combo timer resets when chain breaks
- Best score persisted in localStorage

#### 6. **Audio System**
Web Audio API synthesis (no audio files):
- `beep()`: Generates sine/square/triangle waves for tones
- `noiseBurst()`: Creates noise-based sound effects
- Panning for stereo effects based on screen position
- Mute toggle persisted in localStorage
- Reduced motion mode disables visual effects on player preference

#### 7. **Rendering Pipeline**
Canvas-based 2D rendering:
- Sky gradient background with theme colors
- Enemy and player sprite rendering (geometric shapes with gradients)
- Particle system for visual feedback
- HUD layer (score, hearts, bombs, weapon info)
- Screen flash and vignette effects for hit/bomb feedback
- Performance-adaptive quality (3 levels) adjusts particle and effect complexity

#### 8. **Input Handling**
Multiple input methods with priority:
- **Keyboard**: Arrow keys (movement), Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse**: Crosshair steering + click to fire
- **Touch**: Virtual joystick for movement, fire/bomb buttons
- Input priority: Keyboard/Joystick > Mouse (prevents fighting)

### Game Constants & Configuration

**Difficulty Settings** (`DIFFICULTIES`):
```javascript
{
  spawnBase: number,        // Base spawn delay (frames)
  spawnMin: number,         // Minimum spawn delay
  enemySpeed: number,       // Enemy movement speed multiplier
  enemyShootChance: number, // Probability enemy fires
  lives: number,            // Starting lives
  bombs: number,            // Starting bombs
  bossHpMul: number         // Boss health multiplier
}
```

**Visual Themes** (`THEMES`): 6 procedurally-selected themes with sky gradients, cloud colors, and sun colors

**Responsive Canvas**: Base resolution 480×720, scales responsively up to 1.6x with aspect ratio preservation

### Performance Optimization

**Adaptive Quality System**:
- Tracks frame times over 45-frame window
- If avg > 25ms (40 FPS) → reduce quality (fewer particles)
- If avg < 17.8ms (56 FPS) → increase quality
- Quality levels: 0 (full) → 1 (medium) → 2 (low)

**Collision Detection**: Uses bounding boxes (AABB). Check proximity before expensive pixel-perfect tests.

## Development

### Making Changes

The game is a single HTML file. When editing:

1. **Keep the IIFE scope**: All game code must remain within the `(function () { ... })()` wrapper to avoid global pollution
2. **Reference constants early**: THEMES, DIFFICULTIES, ENEMY_TYPES, BOSSES are defined early and referenced throughout
3. **Maintain entity structure**: Don't introduce classes; keep entity objects simple (data + methods as needed)
4. **Update HUD carefully**: Many DOM elements cache references at the top. If changing HTML structure, update the corresponding selectors

### Common Tasks

#### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` constant with: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update spawn logic in `update()` to recognize the new type
3. Add drawing code in `drawEnemy()`

#### Modifying Boss Difficulty
Change `BOSSES` array color scheme or adjust `hpMul` values. Difficulty modifiers apply during boss creation.

#### Adjusting Difficulty Settings
Edit the `DIFFICULTIES` object constants. Changes affect spawn rate, enemy speed, and player health.

#### Adding Visual Effects
Use the `particles` array. Each particle needs `x`, `y`, `vx`, `vy`, `life`, `draw` function. Example: explosions, hit sparks.

#### Sound Design
- Weapon fire: Adjust `beep()` calls in weapon firing code (frequency, duration, volume)
- Hit feedback: Modify `noiseBurst()` calls for enemy/boss damage

### Testing Tips

1. **Performance**: Open DevTools (F12) and check frame rate. Test on mobile with `isTouchDevice` detection.
2. **Difficulty scaling**: Change `difficulty` variable and restart to test Easy/Normal/Hard balance
3. **Screen sizes**: Resize browser window to test responsive scaling (works up to 1.6x)
4. **Touch controls**: Use Chrome DevTools Device Emulation for joystick testing

### Storage Keys

Game state is persisted using localStorage with keys:
- `skyfighter.bestScore`: Best score across sessions
- `skyfighter.muted`: Audio mute state (1 = muted)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference (1 = enabled)

## Deployment

The entire game is in `index.html`. Simply serve it over HTTP/HTTPS. No build step required.
- Recommended: Use HTTP/2 or gzip compression for the ~59KB file
- Browser support: All modern browsers with Canvas 2D and Web Audio API support

## Future Expansion Ideas

- Additional weapon types with unique fire patterns
- Enemy special abilities (shields, split patterns)
- Boss phase transitions with pattern changes
- Leaderboard integration (currently local storage only)
- Game settings persistence for all preferences
- Accessibility: Keyboard-only alternative to mouse steering
