# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file (`index.html`). The game features:
- Canvas-based rendering with parallax scrolling clouds and dynamic sky themes
- Procedural WebAudio API sound generation (no external audio files)
- Multiple difficulty levels (Easy, Normal, Hard)
- 6 unique bosses with distinct attack patterns
- Weapon upgrades (Vulcan cannon, Laser), power-ups, smart bombs
- Touch and keyboard controls with adaptive joystick UI
- Responsive scaling for desktop and mobile
- Combo system and score multipliers
- Adaptive performance quality scaling (adjusts particle counts based on frame time)

## Running & Development

### Quick Start
Simply open `index.html` in a modern web browser. No build process, no dependencies, no server required.

### Testing Locally
- **Desktop**: Open in Chrome, Firefox, Safari, or Edge
- **Mobile**: Open on iOS Safari or Android Chrome; touch controls appear automatically
- **Alt input testing**: Use keyboard (Arrow keys/WASD to move, Space to fire, X/Shift for bomb) or mouse (move to aim, click to fire)

## Architecture & Key Code Sections

### High-Level Structure
The `index.html` file contains three main layers:

1. **Markup & Styling** (lines 1–359)
   - HTML structure: canvas, HUD overlays, touch controls
   - CSS: responsive layout, animations (heartbeat pulse, stage banners, combo popups), visual effects

2. **Game Logic** (lines 422–1666)
   - Initialization and input handling
   - Game state management (player, enemies, bullets, bosses)
   - Physics and collision detection
   - Difficulty scaling and performance tracking

3. **Audio System** (lines 542–676)
   - WebAudio synth-based sound generation
   - `beep()` and `noiseBurst()` primitives for creating SFX
   - Sound effects: weapon fire, explosions, boss telegraph, game over, etc.
   - Master gain and dynamic range compression

### Core Game Systems

#### Difficulty Constants (lines 485–489)
```javascript
DIFFICULTIES = {
  easy:   { spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul },
  normal: { ... },
  hard:   { ... }
}
```
Controls enemy spawn rates, speed, shooting frequency, starting resources, and boss health multipliers. Affects game progression and pacing.

#### Enemy & Boss Data (lines 502–515)
- **ENEMY_TYPES**: `grunt`, `interceptor`, `tank` — each has HP, speed multiplier, color, score value, and appearance conditions (minStage)
- **BOSSES**: 6 unique boss designs with distinct colors, core glows, and HP multipliers

#### Boss Attack Patterns (lines 1372–1409)
Each boss has 3 distinct attack patterns stored in `BOSS_PATTERNS[designIndex]`. Patterns use helper functions:
- `fireRadial()`: Circle of bullets
- `fireAimedSpread()`: Spread aimed at player
- `fireColumns()`: Vertical walls
- `fireSpiralBurst()`: Rotating spiral
- `fireHomingSeeded()`: Homing with spread scatter
- `fireDoubleRing()`: Two concentric circles

Pattern selection cycles: `patterns[b.patternIndex % patterns.length]`

#### Game Loop & State Updates (lines 1148–1302)
Main `update(dt)` function:
1. Input processing (keyboard, joystick, mouse steering)
2. Player movement with bounds clamping
3. Weapon cooldown and firing
4. Bullet & enemy spawning
5. Collision detection (bullets vs enemies, bullets vs boss, enemies vs player, etc.)
6. Score and combo tracking
7. Particle and effect lifetime management

#### Input Handling (lines 767–861)
- **Keyboard**: Arrow keys, WASD for movement; Space for fire; X/Shift for bomb; P/Esc to pause
- **Touch**: Two-zone layout — left side drag for movement with visual joystick, right side buttons for FIRE and BOMB
- **Mouse**: Move pointer over canvas to aim, click to fire (does not override keyboard/joystick)
- Each input type has priority: keyboard/joystick > mouse; joystick visual only shows during touch interaction

### Rendering Pipeline (lines 1520–1638)

1. **Background**: Sky gradient (stage-dependent theme), sun glow, parallax clouds
2. **Objects**: Player plane, enemies, bullets, enemy bullets, power-ups, particles
3. **Effects**: Screen shake (translates canvas), flash overlay (hit/bomb), vignette (critical health), combo/stage banners
4. **HUD**: Score, best score, multiplier, weapon level, hearts, bombs, boss health bar (when active)

### Performance Adaptation (lines 474–483)
Quality scaling based on frame time average (45-frame window):
- **quality = 0**: Full visual fidelity (glow effects on bullets/ships)
- **quality = 1**: Medium (reduced particles, ~60% visual effects)
- **quality = 2**: Low (minimal particles, ~35% visual effects)

Adjusts automatically; no user intervention needed.

### Storage & Persistence (lines 463–527)
Uses `localStorage` for:
- `skyfighter.bestScore`: High score persistence across sessions
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Accessibility preference override

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502–506) with name, w, h, hp, speedMul, color, dark, score, minStage, optional weave
2. Update `EXPLODE_SFX` (line 678) if using custom explosion sound: `{ myEnemyType: 'myExplosionSfx' }`
3. Add sound effect to `sfx` object (line 624) if needed: `myExplosionSfx: (x) => { ... }`
4. Enemy spawning respects `minStage` automatically; no other changes needed

### Tweaking Difficulty
Modify `DIFFICULTIES` object (lines 485–489):
- `spawnBase` / `spawnMin`: Smaller = spawn faster
- `enemySpeed`: Multiplier on base enemy speed
- `enemyShootChance`: Probability enemy shoots when eligible (0–1)
- `bossHpMul`: Multiplier on boss health

### Adding a Boss Attack Pattern
1. Create a new function in the pattern-helper section (after line 1358):
   ```javascript
   function myPattern(b, speed) {
     // Push enemyBullets with { x, y, w, h, radial: true, dx, dy }
   }
   ```
2. Add to appropriate boss's pattern array in `BOSS_PATTERNS` (lines 1372–1409)
3. Pattern is called with the boss object; use `b.x`, `b.y` to fire from center

### Modifying Game Balance
- **Player health**: Adjust starting `lives` in `DIFFICULTIES` or `loseLife()` threshold (line 1457)
- **Weapon power**: Modify bullet count/spread in `fireWeapon()` (lines 1124–1146)
- **Bomb effectiveness**: Adjust damage in `useBomb()` (line 1117) or area-of-effect
- **Score multiplier**: Adjust `KILLS_PER_STAGE` (line 491) for stage progression; tweak multiplier ramp in `addScore()` (line 1090)
- **Boss HP**: Tune individual boss `hpMul` in `BOSSES` (line 515) or `baseHp` formula (line 1046)

### Adding Sound Effects
WebAudio synth approach — no file dependencies:
- `beep(freq, duration, type, volume, opts)`: Oscillator-based tone with optional sweep, filter, pan
- `noiseBurst(duration, volume, opts)`: Noise with filter and pan
- Options: `{ sweepTo, filterType, filterFreq, pan, delay }`
- Example new SFX (line 624):
  ```javascript
  mySound: () => {
    beep(880, 0.1, 'sine', 0.05);
    noiseBurst(0.08, 0.03, { filterFreq: 3000 });
  }
  ```
- Call `sfx.mySound()` from game logic; audio context auto-initializes on first interaction

### Testing Specific Features
- **Difficulty balance**: Press difficulty button on start screen, play through stages
- **Boss patterns**: Reach stage 2+ to encounter boss; patterns cycle every ~1.5 seconds
- **Weapon upgrades**: Collect green power-ups; cycle between Vulcan (spread at lvl 3) and Laser (dual or triple lane)
- **Touch controls**: Open on mobile or use browser DevTools device emulation; drag left side to move, tap buttons to fire/bomb
- **Accessibility**: Toggle reduced motion (🚫 button, top-right); verifies animations disable and game remains playable

### Optimizing Rendering
Quality scaling is automatic, but if you add new visual features:
- Particle count: Scaled by `quality` variable; use in loops like: `Math.max(3, Math.round(count * (quality === 0 ? 1 : 0.6)))`
- Shadow/glow: Wrap in `if (quality === 0)` to skip on lower-end devices
- Complex shapes: Use canvas primitives (arc, ellipse) rather than multiple paths; batch transform saves

## Browser Compatibility
- **Chrome/Chromium**: Full support (canvas, WebAudio, CSS animations, touch)
- **Firefox**: Full support
- **Safari (iOS/macOS)**: Full support; note WebAudio context suspended until user interaction
- **Edge**: Full support
- Requires: ES6 JavaScript, Canvas 2D, WebAudio API, CSS Flexbox

## Performance Notes
- Single HTML file: No network latency, instant load
- No external dependencies or requests
- Canvas rendering at native resolution (480×720) scaled with CSS, not bitmap resizing
- Particle count adapts to frame rate (target: 50 FPS on average hardware)
- Audio context shares one master gain → consistent volume mix
- localStorage operations are synchronous but minimal (4 reads on load, occasional writes)
