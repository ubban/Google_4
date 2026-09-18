# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:
- Multiple difficulty levels (Easy, Normal, Hard)
- Progressive stage system with boss encounters
- Dynamic weapon system with upgrade progression
- Adaptive performance quality scaling
- WebAudio synthesis for all sound effects
- Full mobile and touch control support
- Canvas-based 2D rendering with responsive scaling

## Architecture

### Single-File Structure
The entire game (gameplay logic, rendering, audio, UI) is contained in `index.html`. The code is organized in functional sections:

1. **Performance & Quality Management** (lines 473-482)
   - Adaptive frame-time tracking to adjust particle/shadow quality
   - Quality levels: 0 (full) → 1 (medium) → 2 (low)
   - Targets smooth 40-60 FPS based on device capability

2. **Game State & Configuration** (lines 485-527)
   - `DIFFICULTIES` object: spawn rates, enemy speed, lives, bombs, boss HP multipliers
   - `THEMES` array: 6 background themes with sky gradients and colors
   - `ENEMY_TYPES`: grunt, interceptor, tank with different properties
   - `BOSSES`: 6 boss variants with unique visual designs and attack patterns
   - LocalStorage persistence for: best score, muted state, difficulty, reduced motion preference

3. **Audio System** (lines 542-676)
   - WebAudio API synthesis (no external audio files)
   - `beep()`: parametric oscillator with frequency sweep, filtering, panning, volume envelope
   - `noiseBurst()`: procedurally generated noise with filtering
   - `sfx` object: predefined sound effects for all game events (shoot, explode, bomb, powerup, boss, etc.)
   - Master gain and compressor for dynamic range control

4. **Input Handling** (lines 768-861)
   - Keyboard: Arrow keys/WASD (movement), Space (fire), X/Shift (bomb), P/Esc (pause)
   - Mouse: pointer position for steering, click to fire
   - Touch: two-zone interface with virtual joystick (left) and action buttons (right)
   - Joystick implementation with circular constraint

5. **Game Loop & Rendering** (lines 1148-1666)
   - Main update loop: player movement, weapon fire, collision detection
   - Enemies: spawning, movement, weaving patterns, shooting logic
   - Boss state machine: entering phase, telegraphing phase, attack patterns
   - Particles: explosion effects with decay
   - Canvas rendering with screen shake, flash effects, vignette
   - Cloud parallax with theme-specific colors

### Key Game Systems

#### Weapon System
- **Vulcan** (default): spread shot pattern that widens with level (1→2→3 bullets)
- **Laser**: narrow, piercing shots; switch to laser after level 3 Vulcan
- Fire rate: Vulcan 0.15s, Laser 0.1s
- Bullets pierce enemies or bosses based on weapon type

#### Enemy AI
- **Spawn system**: difficulty-based spawn timer (Math.max of min/base - ramp)
- **Movement**: base vertical speed + type multiplier; interceptors weave horizontally
- **Shooting patterns**: 
  - "spread": 3-bullet spray
  - "aimed": tracking shot toward player
  - Shoot chance varies by difficulty

#### Boss System
- Six boss types (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each stage (mod 6) spawns corresponding boss
- State machine: entering → telegraphing → attacking
- **Attack patterns** (lines 1372-1409): 6 unique patterns per boss type
  - Radial bursts, aimed spreads, columns, spirals, homing, cross-fire, etc.
- HP scales by: base (70 + stage*35) × difficulty multiplier × boss design multiplier
- Visual feedback: core glows during telegraph, damage feedback on hit

#### Combo & Score Multiplier
- Chain counter increments on each kill
- Multiplier = 1 + floor(chain/5), max 10x
- Chain resets after 2.2 seconds of no kills
- Breaks on player damage

#### Difficulty & Performance
- **Spawn rates**: Easy (40-75), Normal (26-58), Hard (18-40) frames
- **Enemy stats**: speed, shoot chance, health
- **Boss HP**: Easy 0.8x, Normal 1.0x, Hard 1.3x
- **Quality scaling**: particle count, shadow blur dynamically adjust based on frame time

#### Power-ups
- Life (6% drop): restore 1 HP (max to max lives)
- Bomb (8% drop): add 1 bomb (max to max bombs)
- Weapon (20% drop): upgrade weapon level or switch weapon type

## Development Notes

### To Run Locally
Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge). No build step or server required.

### Canvas Sizing
- Base resolution: 480×720 (portrait)
- Scales up to 1.6x based on viewport while maintaining aspect ratio
- Responsive to window resize and orientation change

### Audio
- Uses WebAudio context for real-time synthesis
- Safe initialization with user interaction (click/key to ensure context is not suspended)
- Mute toggle persists to localStorage
- Filter types: lowpass, highpass, bandpass applied per-sound

### Collision Detection
- AABB (axis-aligned bounding box) via `rectsOverlap()` function
- Applies to: enemy↔bullet, boss↔bullet, enemy↔player, bullet↔player, powerup↔player

### Screen Effects
- **Screen shake**: magnitude and duration; reduced motion preference disables or dampens
- **Flash**: white screen on hit, bomb; larger flash on bomb
- **Vignette**: red pulse when health critical (1 HP)
- All effects respect prefers-reduced-motion media query

### Mobile / Touch Behavior
- Touch controls auto-enable on devices with touch support
- Joystick base appears on touch start in move zone, constrains to 50px radius
- Fire button continuous while held, bomb button triggers once per tap
- Safe area insets respected for notched devices

### Accessibility
- Screen reader announcements via `srLive` element with aria-live
- Emoji icons (❤️, 💣, ✈️) for visual indicators, fallback text in announcements
- Reduced motion option for animation-sensitive users
- Keyboard navigation for all controls

### Performance Considerations
- Particle count scales with quality setting (full→medium→low)
- Shadow effects (glow) disabled at low quality
- Enemy/bullet counts can reach ~50-100 during intense moments
- Frame delta clamped to 50ms to prevent spiral on frame drops
- Theme and cloud rendering uses modulo arithmetic for seamless looping

### Browser Compatibility
- Requires: ES6+ JavaScript, Canvas 2D, WebAudio API
- Tested on: Chrome, Firefox, Safari 12+, Edge
- Mobile: iOS Safari 13+, Android Chrome/Firefox

## Common Editing Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502) with: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` logic to include new type in random selection (line ~1027)
3. Add explosion SFX mapping in `EXPLODE_SFX` if unique sound needed (line 678)

### Adding a New Boss Pattern
1. Add pattern function to `BOSS_PATTERNS` array (line 1372)
   - Each function receives boss object and fires bullets
   - Use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
2. Boss selects pattern by index mod array length
3. Pattern fires on telegraph timeout (~22 frames after visual telegraph)

### Tweaking Difficulty
Edit `DIFFICULTIES` object (line 485) values:
- `spawnBase` / `spawnMin`: frames between spawns (higher = fewer enemies)
- `enemySpeed`: multiplier on enemy velocity
- `enemyShootChance`: 0-1 probability each enemy shoots
- `lives`, `bombs`: starting values
- `bossHpMul`: multiplier on base boss HP

### Adjusting Weapon Balance
- Weapon fire rate: `fireDelay` variable (line 1177)
- Vulcan spread pattern: array of angles (line 1127)
- Laser pierce value: 1 or 2 (line 1141)
- Bullet speed: `speed` property in fireWeapon() (lines 1130, 1140)

### Theme / Visual Tweaking
- Sky gradients: edit `THEMES` colors (line 493)
- Boss colors: edit `BOSSES` object (line 508)
- Enemy colors: edit `ENEMY_TYPES` (line 502)
- Cloud rendering: `drawClouds()` function (line 1507) speed and scale parameters

## Code Style Notes
- Self-contained IIFE to avoid global namespace pollution
- Heavy use of shorthand for performance: `en`, `b`, `dt`, `ctx`
- Inline canvas drawing (no sprite atlases) for art
- Deterministic RNG seeding via frame counter for weave patterns
- Magic numbers hardcoded where domain-specific (e.g., 0.15s fire delay is game feel)
