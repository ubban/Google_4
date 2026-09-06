# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single self-contained HTML file. It runs entirely in-browser with no external dependencies, using Canvas 2D for graphics and WebAudio for procedurally generated sound effects. The game features a player-controlled fighter plane, multiple enemy types, boss encounters, and progression through stages.

## Quick Start

- **To run**: Open `index.html` in any modern browser. No build process or server needed.
- **No build required**: The entire game is self-contained in a single file (~58KB).
- **No tests**: This is a game without a test suite. Manual testing in-browser is the verification method.

## Game Architecture

The code is structured as a single IIFE (Immediately Invoked Function Expression) with these main sections:

### Game Loop
```
requestAnimationFrame(loop) → update(dt) → draw() → render canvas
```
- `loop()`: Main game loop; manages delta time and calls update/draw
- `update(dt)`: Game logic (collision detection, movement, spawning, scoring)
- `draw()`: Canvas rendering (backgrounds, sprites, particles, HUD)
- Performance is dynamically tracked and quality is adjusted (quality 0=full, 1=medium, 2=low)

### Player & Combat
- **Player**: Fixed position; controlled via keyboard (WASD/Arrows), mouse (move towards pointer), or touch (left zone for movement)
- **Weapons**: Two types with 3 levels each
  - Vulcan: Spread shot; fires on cooldown (~150ms)
  - Laser: Focused lanes; fires faster (~100ms)
  - Power-up system cycles weapon type and upgrades levels
- **Firing**: Bullets travel upward; laser pierces (levels affect pierce count); regular bullets one-hit kills
- **Bombs**: Destroys all enemies on screen + damage to boss; limited count; recharged via power-ups

### Enemy & Boss Systems
- **Enemy types** (ENEMY_TYPES): grunt, interceptor, tank
  - Each has unique speed, HP, color, spawn weight, and minimum stage
  - Spawn rate increases as stages progress; can weave/dodge
  - Two shoot patterns: aimed (player-tracking) or spread
- **Boss encounters**: 
  - Trigger after KILLS_PER_STAGE (18) enemies killed
  - 6 boss designs cycle based on stage; each has unique visual and attack patterns
  - Boss patterns defined in BOSS_PATTERNS array
  - Bosses have telegraph phase and multiple attack phases

### Difficulty System
Three difficulties scale enemy stats, spawn rates, lives, and boss HP:
- `DIFFICULTIES[difficulty]`: spawnBase/Min, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
- Stored in localStorage; player can change mid-menu

### Score & Progression
- **Chain multiplier**: Increments per kill; resets if 2.2 seconds pass without a kill (breakChain)
- **Multiplier value**: 1 + floor(chain/5), capped at 10x
- **Stages**: Progress when KILLS_PER_STAGE enemies defeated; resets stage counter; increases difficulty ramp
- **Best score**: Persisted in localStorage; displayed at game over

### Audio System
- **No external audio files**: All sounds synthesized via WebAudio
- **Oscillators & noise**: beep() for tonal sounds, noiseBurst() for percussion/impact
- **Spatial audio**: Pan sounds left/right based on x-coordinate via panFor()
- **Mute toggle**: Stored in localStorage

## Key Game Constants

### Difficulties
Located at lines 485-489:
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... }
};
const KILLS_PER_STAGE = 18; // enemies to kill before boss spawn
```

### Themes (Visual)
Six visual themes that cycle per stage (sky gradient, cloud color, sun rays). Located at lines 493-500.

### Enemy Types
Located at lines 502-506: grunt, interceptor, tank. Each defines w/h/hp/color/score/minStage.

### Boss Designs
Located at lines 508-515: BOSSES array with 6 designs (VANGUARD, BEHEMOTH, etc.). Each has hull colors, core color, and hpMul.

### Boss Attack Patterns
Located at lines 1372-1407: BOSS_PATTERNS indexed by (stage-1) % 6. Each boss has 3 patterns that cycle during combat.

## Common Modifications

### Add a new enemy type
1. Add entry to ENEMY_TYPES object (line 502+)
2. Add spawn probability check in spawnEnemy() (line 1028+)
3. Add explosion SFX mapping in EXPLODE_SFX if needed (line 678)
4. Ensure minStage prevents early spawns

### Adjust difficulty parameters
Edit DIFFICULTIES object (line 485). Changes apply immediately; persisted in localStorage.

### Create new boss attack pattern
1. Add pattern function in BOSS_PATTERNS (line 1372+)
2. Use helper functions: fireRadial(), fireAimedSpread(), fireColumns(), fireSpiralBurst(), fireCross(), fireHomingSeeded(), fireDoubleRing()
3. Each function pushes to enemyBullets array; radial:true means use dx/dy instead of speed/vx

### Change visual appearance
- **Player plane**: drawPlane() at line 913
- **Boss ship**: drawBossShip() at line 970
- **Backgrounds & clouds**: Theme colors in THEMES (line 493); drawClouds() at line 1507
- **Particles**: spawnExplosion() at line 1073; render at line 1628

### Add new sound effect
Use beep() or noiseBurst() helper functions (lines 566-623) to synthesize. Add to sfx object (line 624+). Parameters:
- beep: frequency, duration, type (square/sine/sawtooth), volume, opts (sweepTo, filterFreq, pan, delay)
- noiseBurst: duration, volume, opts

### Adjust weapon properties
Edit fireWeapon() (line 1124+):
- Vulcan spread angles and bullet velocity
- Laser number of lanes and pierce levels
- Fire rate: player.cooldown = fireDelay (line 1177)

## Input Handling

- **Keyboard**: Arrow keys / WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Movement steers player toward cursor; click to fire
- **Touch**: Left half of screen is joystick (move), right half has Fire/Bomb buttons
- **Priority**: Keyboard/joystick > mouse steering (mouse doesn't override active input)

## UI Elements & State Persistence

localStorage keys:
- skyfighter.bestScore
- skyfighter.muted
- skyfighter.difficulty
- skyfighter.reducedMotion (respects prefers-reduced-motion media query)

HUD elements updated in real-time: score, multiplier, weapon name/level, hearts (lives), bombs, boss health bar.

## Performance & Quality Settings

Adaptive quality system (lines 474-483) tracks frame time and adjusts:
- quality 0 = full effects (shadow blur, glow, particle count)
- quality 1 = medium (reduces particle count by ~40%)
- quality 2 = low (reduces particle count by ~65%)

Can be toggled manually via "reduced motion" button, which also disables screen shake/flash animations.

## File Organization

Single file, but logically separated by comments:
- Lines 425-461: DOM element cache
- Lines 463-466: Storage key constants
- Lines 473-483: Performance tracking
- Lines 485-527: Game constants & initialization
- Lines 542-676: Audio system
- Lines 700-750: UI rendering & input setup
- Lines 752-862: Canvas sizing & input handlers
- Lines 863-911: Game state initialization
- Lines 1148-1302: Update loop (collision, movement, spawning)
- Lines 1372-1407: Boss attack patterns
- Lines 1507-1638: Rendering functions
- Lines 1640-1666: Main loop & startup

## Mobile Considerations

- Touch controls automatically enabled on touch devices (line 779)
- Joystick visual feedback with draggable stick
- Responsive canvas scaling with MAX_SCALE clamping (line 757)
- Safe area insets respected for notches/status bars
- Reduced motion preference honored for accessibility

## Browser Compatibility

Requires:
- Canvas 2D context
- WebAudio API (optional; game runs without sound if unavailable)
- localStorage
- requestAnimationFrame
- ES6 support (arrow functions, const/let, template literals)

Tested on modern Chrome, Firefox, Safari, and mobile browsers. No polyfills needed.
