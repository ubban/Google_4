# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game (CSS, JavaScript, and HTML) is contained in `index.html` (~1669 lines). It features canvas-based 2D rendering, WebAudio-synthesized sound effects, multiple difficulty levels, 6 distinct boss encounters, enemy AI patterns, and both keyboard and touch input support.

## Getting Started

### Running the Game

Open `index.html` directly in a web browser. No build step or server is required. The game:
- Runs fullscreen and responsive on desktop, tablet, and mobile
- Uses localStorage for persistent score and preference storage
- Implements WebAudio synthesis (no external audio files)
- Adapts rendering quality based on frame rate

### Development Environment

No special setup required. The file can be edited in any text editor and viewed in any modern browser. Changes are immediately visible on reload.

## Code Architecture

### Major Sections (by line ranges)

1. **Initialization & Constants (lines 422-522)**
   - DOM element references
   - localStorage keys
   - Difficulty configurations (easy/normal/hard)
   - Enemy types and boss definitions
   - Theme/color palettes (6 themes with distinct sky gradients)
   - IIFE wrapper enclosing entire game

2. **Audio System (lines 543-676)**
   - WebAudio synth engine with frequency modulation, filtering, and panning
   - Procedural sound effects: beep(), noiseBurst() 
   - SFX object: weapon fire, explosions, boss telegraph, game-over, etc.
   - Mute toggle stored in localStorage
   - Audio initialized on first user interaction (autoplay policy)

3. **UI & Input Handling (lines 680-839)**
   - HUD rendering: hearts, bombs, score, weapon level, multiplier
   - Screen effects: vignette on critical health, flash on hit/bomb
   - Touch joystick implementation for mobile (moveZone, joystick tracking)
   - Mouse-based aiming on desktop
   - Button callbacks (mute, motion, pause, difficulty select)

4. **Game State & Entities (lines 864-871)**
   - Global game variables: player, bullets, enemies, enemyBullets, particles, powerUps, boss
   - Score, lives, bombs, stage tracking
   - Combo chain and multiplier system
   - Spawn timer and frame counter

5. **Core Game Loop (lines 1148-1302)**
   - Single update(dt) function handling all game logic
   - Player movement and boundaries
   - Enemy spawning and movement (difficulty-based)
   - Bullet updates (player and enemy)
   - Collision detection (player↔enemy, player↔bullet, enemy↔player-bullet, bomb)
   - Particle updates (explosions, effects)
   - Power-up collection
   - Score and chain calculations
   - Boss health bar updates

6. **Boss System (lines 1372-1439)**
   - 6 named bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
   - BOSS_PATTERNS array: each boss has 3 unique attack patterns
   - Pattern types: fireRadial, fireAimedSpread, fireColumns, fireSpiralBurst, fireCross, fireHomingSeeded, fireDoubleRing
   - Telegraph warning before each pattern fire
   - Homing and aimed attack variations
   - updateBoss() handles boss movement and pattern timing

7. **Rendering & Animation (lines 1507-1639)**
   - drawClouds() for parallax scrolling background
   - drawPlane() for player and enemy sprite (reusable with colors)
   - drawBossShip() for boss sprite (complex multi-part design)
   - Canvas clear and redraw each frame
   - Particle rendering (explosions)
   - HUD overlay rendering

8. **Main Loop (lines 1640-1668)**
   - requestAnimationFrame-based game loop
   - Delta time calculation
   - Performance tracking for adaptive quality
   - Pause state handling

### Key Design Patterns

**Entity Management:** Entities (enemies, bullets, particles) are stored in arrays updated each frame. No object pooling—allocation/deallocation on demand.

**Collision Detection:** Simple axis-aligned bounding box (AABB) overlap via rectsOverlap(). Checked each frame for player, enemies, bullets, and particles.

**Procedural Audio:** All sounds generated via WebAudio oscillators and noise buffers; no pre-recorded assets. Allows spatial panning and frequency modulation.

**Persistent Storage:** Difficulty, mute state, best score, and motion-preference settings stored in localStorage. Keys: `skyfighter.*`.

**Difficulty Scaling:** DIFFICULTIES object parameterizes spawn rate, enemy speed, and boss HP multiplier per difficulty.

**Theme System:** THEMES array defines sky gradient, cloud color, and sun color. Cycles per stage.

**Adaptive Quality:** Monitors frame time over 45-frame rolling window. Adjusts quality level (0=full, 1=medium, 2=low) if avg > 1/40s or < 1/56s.

## Common Development Tasks

### Adding a New Enemy Type

1. Add entry to ENEMY_TYPES object (around line 502):
   ```javascript
   newType: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#ffaa00', dark: '#ff6600', score: 25, minStage: 3 }
   ```
2. Update spawnEnemy() to include the new type in the pool (after line 1022).
3. Add drawing logic in drawPlane() if the visual differs; otherwise reuse the default.
4. Add SFX callback to EXPLODE_SFX if unique explosion sound desired.

### Adding a New Boss

1. Add to BOSSES array (line 508-515):
   ```javascript
   { name: 'NEWBOSS', hullTop: '#123456', hullBottom: '#654321', wing: '#abcdef', core: '#fedcba', hpMul: 1.2 }
   ```
2. Add attack pattern array to BOSS_PATTERNS (line 1372+):
   ```javascript
   [
     b => fireRadial(b, 12, 150),
     b => fireAimedSpread(b, 4, 280, 0.12),
     b => fireCross(b, 170)
   ]
   ```
3. The index in BOSSES determines which pattern set fires (via designIndex).

### Modifying Difficulty

Edit DIFFICULTIES object (lines 485-489). Parameters:
- **spawnBase**: Frame interval at which to spawn enemies (lower = more frequent).
- **spawnMin**: Minimum interval as difficulty increases in-stage.
- **enemySpeed**: Multiplier on enemy velocity.
- **enemyShootChance**: Probability per frame enemy fires.
- **lives / bombs**: Starting counts.
- **bossHpMul**: Multiplier on boss.maxHp.

### Changing Audio Volume or Synthesis

- Master volume: masterGain.gain.value (default 0.9, line 556).
- Individual SFX volume passed to beep() or noiseBurst().
- Oscillator type: 'sine', 'square', 'sawtooth', 'triangle' (line 571).
- Sweep: osc.frequency.exponentialRampToValueAtTime() (line 573).
- Filter: audioCtx.createBiquadFilter() with type 'lowpass'/'highpass' (line 576).

### Adjusting Visual Effects

- Screen shake magnitude and duration: screenShake(magnitude, duration) (line 727).
- Vignette critical health warning: toggled on/off at line 687.
- Flash effect on hit: flashHit() and flashBomb() animations (lines 698, 704).
- Combo pop text animation: showComboPop() (line 712).

## Performance Notes

- Canvas is drawn every frame; no dirty-rect optimization.
- Particles are allocated on-demand; no pre-pool.
- Enemy count scales with difficulty; no hard cap (can create lag on wave peaks).
- Touch joystick uses pointer events for multi-touch support (iOS, Android).
- Reduced motion preference (prefers-reduced-motion) skips screen shake and flash on devices requesting it.

## Testing & Validation

- No unit tests or build tools.
- Manually test in browser after changes:
  - Difficulty ramp (verify spawn rates and enemy behavior change).
  - Boss patterns (confirm all 3 patterns per boss fire and don't overlap).
  - Touch input (joystick responsiveness on mobile).
  - Audio (ensure all SFX play without distortion, especially bomb explosion).
  - Collision (player↔enemy, bullet↔enemy, bomb radius).
  - Score calculation and multiplier chain.
- Check performance on low-end devices; quality auto-scale should engage if FPS drops.

## Repository Structure

```
Google_4/
├── index.html      (entire game: 1669 lines CSS + JS + HTML)
├── README.md       (minimal metadata)
├── .git/           (version control)
└── CLAUDE.md       (this file)
```

## Browser Compatibility

- Requires ES6+ (arrow functions, const/let, template literals, Math.hypot, Object.getChannelData).
- WebAudio API for sound synthesis.
- Canvas 2D context.
- localStorage for persistence.
- Pointer Events API for touch input (fallback: mouse events).
- Linear/radial gradients for theming.
- CSS backdrop-filter for HUD card blur (graceful degradation if unsupported).

Tested on modern Chrome, Firefox, Safari, and mobile browsers.

## Future Enhancement Ideas

- Multiple weapon types beyond Vulcan/Laser (missile, spread, etc.).
- Persistent progression system (stage selection, unlockables).
- Leaderboard (via server or IndexedDB).
- More bosses (expand BOSSES and BOSS_PATTERNS).
- Replay recording/playback (input log).
- Custom difficulty slider instead of 3 presets.
- Accessibility: keyboard-only controls, high-contrast mode.
