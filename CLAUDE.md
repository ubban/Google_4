# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 vertical shoot-em-up game inspired by Raiden. It features:
- Canvas 2D rendering with dynamic visual effects
- WebAudio API-based sound synthesis (no external audio assets)
- Difficulty scaling with three levels (easy/normal/hard)
- Boss encounters with health bars
- Weapon progression system with 3 weapon types
- Touch and keyboard controls
- Full accessibility support (ARIA live regions, reduced motion, screen readers)
- Adaptive quality system for performance management
- LocalStorage for persistence (scores, settings, difficulty)

## Quick Start

The entire game is in `index.html` (1,669 lines). To run:
1. Open `index.html` in any modern browser
2. Use **Arrow Keys** or **WASD** to move, **Space** to shoot, **X** to bomb
3. On touch devices: use on-screen joystick and buttons

## Code Architecture

### Structure
Everything is in a single file with three sections:
1. **HTML/DOM** (lines 1–420): Page structure, UI elements, HUD layout
2. **CSS** (lines 10–360): Responsive styling with dark theme, animations, effects
3. **JavaScript IIFE** (lines 422–1666): Entire game logic in a single closure

### Main Game State (line 864)
```javascript
let player, bullets, enemies, enemyBullets, particles, powerUps, boss;
```
Mutable objects tracking active game entities.

### Key Systems

**Game Loop** (line 1640–1652)
- `requestAnimationFrame(loop)` drives the main loop
- Calls `update(dt)` for state changes, then `draw()` for rendering
- Time-delta capped at 50ms to prevent large jumps
- Performance tracked via `trackPerf()` for adaptive quality

**Update System** (line 1148)
- `update(dt)`: Main game logic—collision, spawning, movement
- `updateBoss(dt)`: Boss-specific AI and attacks
- Handles player input, enemy spawning based on kills and difficulty
- Manages weapon cycling, power-up collection, hit detection

**Rendering** (line 1517)
- `draw()`: Clears canvas, applies effects (vignette, flash, shake), renders all entities
- Custom drawing functions: `drawPlane()`, `drawBossShip()`, `drawBullet()`
- Conditional glow effect based on quality setting

**Audio System** (line 545–597)
- WebAudio synth—no external files, all procedural
- `beep(freq, dur, type, vol, opts)`: Synthesized tones with frequency sweep
- `noiseBurst(dur, vol, opts)`: Noise for explosions
- Stereo panning based on entity position (`panFor()`)
- Master gain + compressor for dynamic range

**Difficulty & Game Balance** (lines 485–489)
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```
Tweaked via `spawnBase/spawnMin` (enemy spawn intervals), `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`.

**Weapons** (lines 747–788)
Three types with distinct behavior:
- **VULCAN**: Standard multi-bullet spread, fast rate
- **CANNON**: Large slow projectiles, strong damage
- **LASER**: Narrow beam, maximum damage per hit
Leveled 1–3; higher levels increase spread/damage/fire rate.

**Enemies** (lines 502–506)
```javascript
const ENEMY_TYPES = {
  grunt:       { w: 34, h: 34, hp: 1, speedMul: 1,    color: '#e53935', score: 10, minStage: 1 },
  interceptor: { w: 26, h: 26, hp: 1, speedMul: 1.7,  color: '#ff7043', score: 15, weave: true },
  tank:        { w: 46, h: 46, hp: 4, speedMul: 0.55, color: '#5e35b1', score: 35, minStage: 2 }
};
```
Spawning strategy: `spawn cooldown` → `min kills to unlock` → `speed/AI variance per difficulty`.

**Bosses** (lines 508–515)
Six unique bosses, each with distinct colors and HP multiplier. Spawned after every 18 kills.

**Themes** (lines 493–500)
Six visual themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic) with sky gradients and cloud/sun colors.

## Common Development Tasks

### Adjusting Game Balance
- Enemy spawning: Tweak `DIFFICULTIES` object (spawnBase, spawnMin, enemySpeed, enemyShootChance)
- Boss HP: Modify `bossHpMul` in DIFFICULTIES or individual boss `hpMul` in BOSSES array
- Lives/bombs: Edit DIFFICULTIES
- Score values: ENEMY_TYPES and powerup constants

### Adding/Tweaking Visuals
- **Colors**: Update THEMES, ENEMY_TYPES, BOSSES, POWERUPS
- **Particle effects**: Modify `particles` array behavior in collision handlers (e.g., lines 1276–1285)
- **Animations**: CSS keyframes (heartBeat, cardIn, iconBob, etc.) or canvas animation timing

### Modifying Sounds
- Lookup function names like `beep()`, `noiseBurst()`, then adjust frequency/duration parameters
- All sounds are parametric—no audio files to manage

### Performance Tuning
- Adaptive quality: `quality` variable (0=full, 1=medium, 2=low) adjusts particle counts and glow
- If too slow: increase `spawnMin` to spawn fewer enemies, or reduce particle counts
- Frame time tracking: See `trackPerf()` (line 476)

### Accessibility
- Screen reader announcements: `announce(text)` updates `#srLive` ARIA live region
- Reduced motion: `reducedMotion` bool disables vignette/flash/shake
- Check `.reduced-motion` CSS class toggle for conditional effects

## Testing & Validation

**Manual Testing**:
1. Test all three difficulties—spawn rates and balance should feel progressive
2. Verify boss encounters at stage changes (every 18 kills)
3. Check weapon cycling with W/E or buttons (desktop/touch)
4. Verify touch controls work on mobile (joystick and fire/bomb buttons)
5. Mute toggle persists via LocalStorage
6. Reduced motion mode hides vignette/flash/shake

**Performance**:
- Open DevTools → Performance tab; target 60fps
- Monitor frame times; if drops, check enemy/particle counts or reduce quality thresholds

**Accessibility**:
- Enable a screen reader (NVDA, JAWS, VoiceOver)
- Verify critical events are announced (start, hit, stage clear)
- Test reduced motion toggle

## File & Editor Notes

- **Single file simplicity**: No build step, no dependencies, works offline
- **Responsive design**: Uses clamp() and vw/vh units; tests on desktop, tablet, mobile
- **Dark theme only**: Color scheme is hardcoded for dark mode; adjust :root if needed
- **Touch-friendly**: All buttons min 36px; safe area insets for notches
