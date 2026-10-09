# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a standalone HTML5 application. The entire game logic, rendering, and audio synthesis are contained in a single `index.html` file with embedded CSS and JavaScript.

## Architecture

### Single-File Design
- **index.html** (1670 lines) contains:
  - Inline CSS styling for the game canvas, HUD, UI overlays, and animations
  - Vanilla JavaScript game engine with no external dependencies
  - Canvas 2D rendering pipeline
  - Procedural audio synthesis using Web Audio API
  - Touch and mouse input handling

### Game Loop Structure
The game runs on `requestAnimationFrame` with the main loop at line 1640:
1. **Update phase** (`update(dt)`): Game logic, collision detection, state changes
2. **Draw phase** (`draw()`): Canvas rendering with performance-aware quality scaling
3. **Input handling**: Keyboard (Arrow keys/WASD/Space/X), mouse pointer, and touch controls

### Core Game Systems

#### State Management (lines 863-869)
- Player object: position, health, weapon, cooldown
- Arrays for entities: `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Game state: `score`, `lives`, `running`, `paused`, `gameOver`, `stage`, `chain` (combo multiplier)
- Persistent storage via localStorage for best score, difficulty, mute state, and motion preferences

#### Entity System
- **Player** (line 877): Central character with weapon system, invulnerability frames, collision bounds
- **Enemies** (lines 1022-1040): Three types (grunt, interceptor, tank) with different HP/speed/scoring
  - Spawning throttled by difficulty and progression (lines 1197-1207)
  - AI: Weaving movement, shoot patterns (spread or aimed)
- **Bullets** (line 1124-1146): Two weapon types
  - Vulcan: Spread pattern increases at weapon level 2-3
  - Laser: Pierce through enemies, spreads horizontally at level 2-3
- **Boss** (lines 1043-1056, 1411-1439): Six unique boss designs with pattern-based attack AI
  - 6 distinct visual designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
  - Each has 3 unique attack patterns from `BOSS_PATTERNS` array (lines 1372-1409)
  - Health scaled by difficulty and boss type

#### Weapon System (lines 871-872, 1304-1322)
- Vulcan: Square bullets, spread increases with level, rapid fire
- Laser: Piercing bullets with reduced fire rate, wider spread at higher levels
- Power-ups spawn (6% life, 8% bomb, 20% weapon) when enemies die
- Weapon upgrades: Level 1→2→3, then toggles to alternate weapon at level 1

#### Difficulty System (lines 485-489)
- Three presets: Easy, Normal (default), Hard
- Affects: enemy spawn rate, speed, shoot chance, player lives, bomb count, boss HP
- Stored in localStorage and selectable before game start

#### Stage Progression
- Kill 18 enemies (`KILLS_PER_STAGE`) to trigger boss encounter
- Boss defeat increments stage and clears kill count
- 6 visual themes cycle through stages (lines 493-500)
- 6 boss designs cycle with per-difficulty HP scaling

#### Combo & Scoring (lines 1087-1101)
- Chain counter increments on consecutive kills (2.2s window between kills)
- Multiplier: `1 + floor(chain / 5)` capped at 10x
- All scores multiplied by current multiplier
- Bomb kills grant score bonus

#### Audio System (lines 542-676)
- Web Audio API synth with no external files
- Master gain and compressor for dynamic range
- Sound effects use oscillators (sine/square/sawtooth) with frequency sweeps
- Noise bursts for percussion/explosions
- Spatial panning for positional audio
- Mutable, stored in localStorage

#### Rendering Pipeline (lines 1520-1638)
- **Screen shake** support (lines 726-730) with reduced-motion preference
- **Quality scaling** (lines 474-483): Monitors frame time and adjusts particle count/shadows
  - Quality 0 (full): All particles, glow shadows
  - Quality 1 (medium): 60% particles
  - Quality 2 (low): 35% particles, no glow shadows
- **Drawing order**: Sky gradient → clouds → player → bullets → enemies → boss → power-ups → particles
- **Visual effects**: Glow shadows, flash animations, vignette pulse on low health, screen shake
- **Accessibility**: Color-safe palette, reduced-motion preference respected, screen reader announcements

#### Input Handling
- **Keyboard**: Arrow keys or WASD for movement, Space or left-click for fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Pointer movement steers plane, click fires; priority yields to keyboard input (lines 836-861)
- **Touch**: Virtual joystick on left, FIRE/BOMB buttons on right; touch-optimized button sizes

#### UI & Overlay System
- **HUD** (top, line 367-374): Score, best score, multiplier, weapon level/name, hearts, bombs
- **Boss bar** (lines 376-379): Appears when boss spawned, shows remaining health
- **Overlays**: Start screen, pause menu, game-over screen with stats
- **Animations**: Combo pop-ups, stage banners, flash effects, vignette pulse on critical health

## Development Guide

### Building & Running
This is a single-file HTML game. To run locally:
```bash
# Open in any modern browser
open index.html
# or serve via HTTP for full functionality
python3 -m http.server 8000
```

### Testing Gameplay
- **Normal flow**: Start → kill enemies → stage progression → boss battles → repeat until game over
- **Difficulty testing**: Test Easy/Normal/Hard before publish (affects spawn rate, enemy speed, boss HP)
- **Edge cases**:
  - Single heart (critical state): Vignette pulse should activate
  - Combo tracking: Kill 5 enemies fast to test multiplier increase
  - Bomb detonation: Clear all enemies/bullets and damage boss
  - Weapon swaps: Reach level 3, pick up weapon power-up to toggle to laser
  - Mobile: Test touch joystick and button layout on various screen sizes

### Key Logic Areas for Modifications

#### Adjusting Difficulty Parameters
Edit `DIFFICULTIES` object (line 485-489) to rebalance spawn rates, speeds, or HP.

#### Adding/Modifying Boss Patterns
- Boss designs in `BOSSES` array (lines 508-515)
- Attack patterns in `BOSS_PATTERNS` array (lines 1372-1409)
- Each pattern function receives the boss object and fires enemy bullets
- Helper functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`

#### Tweaking Enemy Types
Edit `ENEMY_TYPES` object (lines 502-506) for new enemy stats or types. Update `spawnEnemy()` (line 1022) selection logic.

#### Changing Game Constants
- Canvas size: `BASE_W`/`BASE_H` (line 753)
- Enemies per stage: `KILLS_PER_STAGE` (line 491)
- Stage themes: `THEMES` array (lines 493-500)

#### Visual Tuning
- Colors: Scattered throughout CSS and JS (e.g., player color #2196f3 at line 1558)
- Animations: CSS keyframes (lines 71-358) or JS timing (e.g., screen shake duration)
- Screen shake: `screenShake(magnitude, duration)` calls scattered in game logic

#### Audio Tuning
- Sound effects in `sfx` object (lines 624-676)
- Frequency, duration, and volume tweakable in each effect function
- Master volume: `masterGain.gain.value` at line 556

### Performance Considerations
- Particle count scales with quality level to maintain 60 FPS on low-end devices
- Canvas size clamped to maintain aspect ratio on mobile
- Shadow blur disabled on quality level 2
- Touch action prevented to avoid browser zoom/scroll interference

### Accessibility Features
- Reduced motion preference detected at startup (line 520)
- All animations disable when `body.reduced-motion` class is set
- Screen reader announcements for game events (lines 468-471, `srLive` element)
- Proper heading hierarchy and ARIA labels

## Common Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with w, h, hp, speedMul, color, dark, score, minStage
2. Update `spawnEnemy()` selection logic to include your type's probability (line 1026)
3. Add explosion sound effect mapping to `EXPLODE_SFX` if needed (line 678)

### Extending Stage Count
- Boss designs and themes cycle indefinitely via modulo (stages wrap to 6)
- To add new bosses, extend `BOSSES` and `BOSS_PATTERNS` arrays
- New themes: add to `THEMES` array with sky gradient colors and sun properties

### Customizing Mobile Controls
- Virtual joystick: `#moveZone`, `#joystickBase`, `#joystickStick` (lines 291-341)
- Fire/bomb buttons: `#fireBtn`, `#bombBtn` (lines 310-326)
- Adjust touch zone sizes and button positioning in CSS

### Debugging Game State
Add breakpoints or console logs in the `update()` or `draw()` functions. Check:
- `player.x / player.y`: Player position
- `enemies.length`: Active enemy count
- `boss`: Current boss object (null when none)
- `score`, `multiplier`, `chain`: Score state
- `lives`, `bombCount`: Player resources

## Storage & Persistence
Keys in localStorage (prefixed `skyfighter.`):
- `bestScore`: High score (number)
- `muted`: Audio state (0/1)
- `difficulty`: Selected difficulty (easy/normal/hard)
- `reducedMotion`: Motion preference override (0/1)
