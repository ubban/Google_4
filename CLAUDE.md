# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a self-contained HTML5 application. The entire game runs in a single `index.html` file (~1.67KB minified structure) with integrated HTML, CSS, and JavaScript.

### Key Technologies
- **Canvas 2D API**: For all game rendering (backgrounds, sprites, effects)
- **WebAudio API**: Procedural sound synthesis (no external audio assets)
- **LocalStorage**: Persistent storage for best score and user preferences
- **Responsive Design**: Adaptive canvas scaling, touch controls, and accessibility features

## Architecture & Code Organization

The game is structured as a single self-executing function containing:

### Game State
- Player object with position, speed, cooldown, weapon state, invulnerability
- Arrays for bullets, enemies, enemy bullets, particles, power-ups
- Score, lives, stage progression, combo chain tracking
- Boss state (when active)

### Core Systems

#### 1. **Rendering** (`draw()` function, line 1520)
- Sky gradient with thematic colors
- Cloud parallax layers
- Player plane with flame effect
- Enemy planes with HP bars (tanks)
- Boss ships with custom designs
- Bullets with glow effects
- Particles (explosions)
- Power-up pickups

#### 2. **Update Loop** (`update()` function, line 1148)
- Player movement (keyboard, joystick, mouse tracking)
- Weapon firing and cooldown
- Enemy spawning with difficulty ramping
- Bullet-enemy collision detection
- Bullet-boss collision detection
- Enemy bullet-player collision detection
- Boss attacks and pattern cycling
- Combo chain decay
- Performance tracking (adaptive quality)

#### 3. **Boss System** (`updateBoss()` function, line 1411)
- 6 distinct boss designs with unique attack patterns
- Telegraph (visual warning) before attacks
- Pattern cycling based on design
- 8 different attack pattern functions:
  - Radial burst, aimed spread, columns
  - Spiral burst, cross fire, homing volleys
  - Double rings, teleport mechanics

#### 4. **Audio Synthesis** (lines 542-676)
- Master gain, compressor, and Web Audio context
- Two core synth functions: `beep()` for tones, `noiseBurst()` for noise
- Stereo panning based on screen position
- SFX library: shoot, laser, explosions (grunt/interceptor/tank specific), hit, bomb, powerup, boss telegraph/death, UI click, gameover
- All effects are synthesized in real-time—no audio files

#### 5. **Input System** (lines 767-862)
- **Keyboard**: Arrow keys/WASD for movement, Space/Shift/X for fire/bomb, P/Esc for pause
- **Mouse**: Move to steer, click to fire
- **Touch**: Left side zone for joystick, right side for action buttons
- Touch joystick has visual feedback (base and stick markers)
- All input modes can coexist (keyboard priority > joystick > mouse)

#### 6. **Difficulty System** (line 485)
Three difficulty tiers affecting:
- Enemy spawn rate (base and minimum)
- Enemy speed and shooting chance
- Starting lives and bombs
- Boss HP multiplier

#### 7. **Stage & Enemy Progression**
- 6 visual themes cycling by stage
- 3 enemy types: grunt (basic), interceptor (fast, weaving), tank (slow, high HP)
- Enemy types unlock at specific stages
- Boss spawns after 18 kills per stage
- Stage-specific theme updates

### Key Constants & Data Structures

**DIFFICULTIES** (line 485): Tuning tables for Easy/Normal/Hard
**THEMES** (line 493): 6 sky gradient palettes with cloud and sun colors
**ENEMY_TYPES** (line 502): Grunt, Interceptor, Tank specs (dimensions, HP, speed, score)
**BOSSES** (line 508): 6 boss designs with hull colors, core colors, HP multipliers
**BOSS_PATTERNS** (line 1372): Attack pattern arrays for each boss

### Performance Adaptive Quality (lines 473–482)
- Tracks frame times over 45-frame window
- Reduces particle count, shadow blur, and effect intensity on lower-end devices
- 3 quality levels: full (0), medium (1), low (2)

### Accessibility Features
- Reduced motion preference detection and toggle (stored in localStorage)
- Screen reader live region (`#srLive`) for announcements
- All flash/shake effects disabled when reduced motion is enabled
- High contrast UI with text shadows

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502) with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Optionally add `weave: true` for serpentine movement
3. Update `spawnEnemy()` logic (line 1022) to control spawn probability
4. Add sound effect in `sfx` object if unique explosion needed
5. Map sound effect in `EXPLODE_SFX` lookup (line 678)

### Adding a New Boss
1. Add design object to `BOSSES` array (line 508) with colors and `hpMul`
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372)
3. Boss names cycle through stages; they'll appear automatically on the next progression

### Adding a New Attack Pattern
1. Write pattern function in the `update()` scope (e.g., `fireRadial()`, line 1324)
2. Add to appropriate boss entry in `BOSS_PATTERNS`
3. Pattern receives boss `b` and fires into `enemyBullets` array
4. Use `radial: true` flag for non-linear bullet motion

### Tweaking Game Balance
- **Difficulty**: Modify `DIFFICULTIES` constants (line 485)
- **Weapon**: Adjust spread/lanes in `fireWeapon()` (line 1124) or fire rate via `fireDelay`
- **Enemy spawning**: Change `spawnBase` / `spawnMin` or difficulty ramping in `update()` (line 1201)
- **Boss HP**: Adjust base HP calculation (line 1046) or `hpMul` in BOSSES

## File Manifest
- **index.html**: Complete game (HTML + embedded CSS + embedded JavaScript)
- **README.md**: Minimal project header
- **CLAUDE.md**: This file

## Running & Testing

### Local Testing
1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Game starts with overlay menu; difficulty can be selected before play
3. Use browser DevTools for performance monitoring and breakpoints
4. LocalStorage is sandbox per origin; clear browser storage to reset best scores

### Mobile Testing
- Use browser mobile emulation or real device
- Touch controls enable automatically on touch-capable devices
- Canvas scales responsively; base dimensions are 480×720px

### Performance Notes
- WebAudio context requires user gesture to resume (clicks/taps activate)
- Particle count and shadow effects adapt based on frame time
- Large explosion particles (40 count for boss death) can spike GPU usage on low-end devices
- Reduce `spawnBase` or disable cloud layers to improve FPS on mobile

## Code Style & Conventions

- **Variables**: Single-letter loop vars common (i, j, a, b); longer names for game objects
- **Functions**: Utility functions (spawn*, draw*, fire*) prefixed with verb; state variables module-scoped
- **Canvas context methods**: Save/restore used extensively to manage transforms and blend modes
- **Magic numbers**: Mostly inline with comments for effect timings and collision thresholds
- **No external dependencies**: Pure vanilla JS—no libraries or build tools required

## Future Enhancement Ideas

- **Weapon variety**: Spread shot, missile, homing drones
- **Stage-specific gimmicks**: Screen-shake events, meteor spawns, temporary shields
- **Combo visuals**: Scaling damage numbers, screen-clearing effects at high multipliers
- **Leaderboard**: IndexedDB for multi-run statistics
- **Mobile-specific**: Haptic feedback on hit/bomb via Vibration API
- **Sound options**: Master volume slider, individual SFX toggles
- **Pause menu**: Settings panel in the pause overlay

## Debugging Tips

- `window.quality` can be set to 0/1/2 to force adaptive quality
- `window.frame` tracks elapsed frames for animation/pattern timing
- Set `window.reducedMotion = false` to force-enable effects
- Canvas size: `canvas.width` and `canvas.height` are the 480×720 internal resolution
- Display scaling: Controlled by `resize()` function; aspect ratio is maintained
