# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The game features player-controlled aircraft combat, multiple enemy types, challenging boss fights, and a scoring system with progressive difficulty.

- **Language**: JavaScript (ES6+) within HTML5
- **Architecture**: Single-file self-contained application (~1,700 lines)
- **Key Technologies**: HTML5 Canvas, WebAudio API, Gamepad API, localStorage

## Running the Game

**To play:** Open `index.html` in a web browser. No server or build system required.

**Supported input methods:**
- Keyboard: Arrow keys or WASD to move, Space to fire, P to pause
- Mouse: Click and move to position the player, click to fire
- Gamepad: Analog stick to move, A/Cross button or trigger to fire
- Touch: Tap and drag on screen to move/aim (mobile devices)

## Code Architecture

The entire game is wrapped in an IIFE (Immediately Invoked Function Expression) to protect the global namespace. The structure follows this pattern:

### 1. **Constants & Configuration** (lines ~350–530)
- `DIFFICULTY` levels and their parameters (enemy speed, shoot chance, boss HP multiplier)
- `ENEMY_TYPES`: grunt, interceptor, tank definitions with stats
- `BOSSES`: six unique boss designs with color schemes and HP multipliers
- `BOSS_PATTERNS`: attack pattern functions for each boss (fire patterns at indices 0–5)
- `THEMES`: background color gradients, sun glow, cloud colors for each stage
- Storage keys for persistence (best score, muted state, difficulty, reduced motion preference)

### 2. **Game State** (lines ~530–560)
Global mutable state managing:
- Player: position, weapon, level, health, invulnerability, cooldown
- Bullets and enemy bullets (arrays of projectile objects)
- Enemies and boss (dynamic arrays)
- Power-ups (life, bomb, weapon upgrades)
- Particles (explosion debris)
- Game flags: running, paused, gameOver
- Score, chain multiplier, stage counter

### 3. **Input Handling** (lines ~570–750)
- Keyboard event listeners for arrow keys, WASD, Space, P, and UI buttons
- Mouse tracking for aim-and-click control mode
- Gamepad/joystick polling with dead-zone handling
- Touch event listeners for mobile support
- Focus/blur handling to prevent unexpected input when window loses focus

### 4. **Audio System** (lines ~750–850)
Procedurally generated sounds using WebAudio API with no external assets:
- `beep(freq, duration, type, volume, opts)`: oscillator-based tone synthesis with frequency sweeps and filters
- `noiseBurst(duration, volume, opts)`: white noise with envelope and filtering
- `sfx` object: named sound effects (shoot, laser, explode, hit, bomb, power-up, boss telegraph, game over, etc.)
- Spatial panning based on event x-position for stereo effect
- Master gain and compressor for dynamic range control

### 5. **Rendering** (lines ~850–1100)
Canvas drawing functions:
- `draw()`: main render function (clears, draws backgrounds, players, enemies, bullets, UI)
- `drawPlane()`: draws aircraft (player, enemies, boss) with 3D perspective
- `drawBossShip()`: boss-specific rendering with animated core glow and wings
- Screen shake effect applied via canvas translate
- Vignette effect for critical health state
- Parallax scrolling clouds at different speeds
- Particle system for explosion effects

### 6. **Collision & Damage** (lines ~1100–1250)
- `rectsOverlap()`: AABB collision detection for rectangles
- Enemy-bullet collision: damage tracking, HP system for tanks
- Player-bullet collision: life loss, invulnerability frames
- Player-power-up collision: weapon upgrade, health/bomb gain
- Enemy-power-up collision: prevents pickup on enemy collision
- `screenShake()` and `flashHit()` for feedback on damage

### 7. **Game Loop & Update** (lines ~1250–1550)
- `update(dt)`: primary game state update function
  - Player movement from keyboard, mouse, or gamepad input
  - Boundary checking to keep player on screen
  - Weapon firing with cooldown and weapon-type logic
  - Enemy spawning with difficulty scaling
  - Boss AI: movement, attack telegraphing, pattern execution
  - Collision resolution for all object types
  - Chain/multiplier decay
  - Win/lose condition checks
- `loop(now)`: requestAnimationFrame callback that:
  - Calculates delta time with 50ms cap to prevent spiral-of-death
  - Tracks performance stats for quality auto-adjustment
  - Calls update() and draw() conditionally based on running/paused state

### 8. **Enemy & Boss AI**
- **Enemy spawning**: random types, positions, and attack patterns (spread vs. aimed shots)
- **Enemy movement**: weaving patterns for interceptors, straight descent for others
- **Weapon levels**: player weapon (vulcan or laser) has 1–3 upgrade levels affecting spread/pierce
- **Boss patterns** (lines ~1300–1410): indexed arrays of attack functions per boss
  - Column fire (multiple vertical bullet streams)
  - Radial/full-ring fire
  - Homing seeded projectiles (arc toward player)
  - Double-ring patterns
  - Aimed spreads and combinations
- Boss telegraph: 22-frame warning flash before attack launches

### 9. **Difficulty & Progression**
- Difficulty levels (easy/normal/hard) scale enemy speed, boss HP, and spawn rate
- Progressive stages with cycling themes and boss designs
- Score multiplier (1–10x) based on kill chain
- Stage advancement triggers boss spawn and theme change

## Key Implementation Details

### Collision System
Uses simple AABB (axis-aligned bounding box) with center-based overlap test:
```javascript
Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2
```

### Performance Scaling
Auto-adjusts rendering quality (0–2: high to low) based on frame time tracking:
- Quality 0: Full glow effects and shadows
- Quality 1: Reduced shadow blur and particle count
- Quality 2: Minimal effects and fewer particles

### Persistence
Uses localStorage for:
- Best score
- Muted state
- Selected difficulty
- Reduced motion preference

### Audio Initialization
WebAudio context is lazily initialized on first user interaction (required by browser autoplay policy).

## Common Development Tasks

### Adjusting Game Balance
1. **Enemy difficulty**: Modify `DIFFICULTIES` object (lines ~475–495) — adjust `enemySpeed`, `spawnRate`, `enemyShootChance`, `bossHpMul`
2. **Boss HP**: Adjust `hpMul` in each boss definition (lines ~508–515) or `baseHp` multiplier in `spawnBoss()` (line ~1046)
3. **Weapon balance**: Modify damage/spread in `fireWeapon()` (lines ~1124–1146) or bullet properties (speed, pierce)
4. **Enemy types**: Add/modify entries in `ENEMY_TYPES` (lines ~502–506)

### Adding New Bosses
1. Add entry to `BOSSES` array with color scheme and HP multiplier
2. Create attack pattern function array in `BOSS_PATTERNS` (lines ~1300–1409) at matching index
3. Add theme to `THEMES` array if desired (line ~475)

### Adding Boss Attack Patterns
Boss patterns are functions called with the boss object:
- `fireRadial(boss, bulletCount, speed)`: circle of bullets
- `fireColumns(boss, speed)`: vertical bullet streams
- `fireAimedSpread(boss, bulletCount, speed, angle)`: angled spread
- `fireHomingSeeded(boss, bulletCount, speed)`: homing projectiles
Combine multiple patterns in a stage array for phase-based behavior.

### Sound Effects
New sounds can be added to the `sfx` object by composing `beep()` and `noiseBurst()` calls:
```javascript
mySound: () => {
  beep(440, 0.1, 'sine', 0.1);
  noiseBurst(0.05, 0.05);
}
```

### Visual Themes
Modify `THEMES` array (line ~475) to change backgrounds:
```javascript
{ name: 'Stage Name', sky: ['#color1', '#color2', '#color3'], sun: 'R,G,B', cloud: 'R,G,B' }
```

## No Build System or Testing Framework

This is a self-contained game file. No npm, webpack, build scripts, or unit tests. The game is tested by playing in a browser.

## Browser Compatibility

Requires:
- HTML5 Canvas API
- WebAudio API (for sound; game runs without audio if unavailable)
- Gamepad API (optional; game works with keyboard/mouse)
- ES6 (arrow functions, const/let, template literals)
- localStorage (for persistence; game runs without it)

Tested on modern browsers (Chrome, Firefox, Safari, Edge).
