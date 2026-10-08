# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-page HTML application. It features procedural enemy waves, multiple boss battles, power-ups, two weapon types, and difficulty scaling.

### Key Characteristics

- **Single-file architecture**: All code (HTML, CSS, JavaScript) is in `index.html` (~1670 lines)
- **No external dependencies**: Uses only browser APIs (Canvas 2D, Web Audio API, localStorage)
- **Responsive design**: Scales to any screen size; supports both desktop (mouse/keyboard) and mobile (touch) controls
- **Adaptive performance**: Dynamically reduces particle effects and glow on low-end devices
- **Synthesized audio**: All sound effects generated in real-time with Web Audio API, no audio files

## Architecture Overview

### Game Loop Structure
```
requestAnimationFrame(loop)
  → trackPerf(dt)
  → update(dt)          // Game logic, physics, collision
  → draw()              // Canvas rendering
```

The game runs at ~60 FPS with frame delta-time (`dt`) capped at 0.05s to prevent logic issues on slow frames.

### Core Game State
- **Player**: Position, speed, weapon type/level, lives, invulnerability frames, cooldown
- **Bullets**: Player shots with optional pierce and laser variants
- **Enemies**: Position, type (grunt/interceptor/tank), HP, shooting patterns
- **Enemy Bullets**: Radial or aimed projectiles from enemies/bosses
- **Power-ups**: Life, bomb count, weapon upgrades (floating down-screen)
- **Particles**: Explosion effects (auto-culled after maxLife)
- **Boss**: Health bar, telegraph phase, attack patterns

### Stage Progression
- **KILLS_PER_STAGE = 18**: Number of enemies to defeat before boss spawns
- **6 themes**: Visual stage variety (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- **6 boss designs**: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN (cycle repeats)
- **Difficulty scaling**: Spawn rate increases, enemy speed scales, boss HP multiplier changes

### Difficulty Settings
Three presets stored in localStorage:
```javascript
easy:   spawnBase=75,  enemySpeed=1.0,  lives=4,  bombs=3,  bossHpMul=0.8
normal: spawnBase=58,  enemySpeed=1.4,  lives=3,  bombs=2,  bossHpMul=1.0  (default)
hard:   spawnBase=40,  enemySpeed=1.9,  lives=3,  bombs=2,  bossHpMul=1.3
```

### Input Handling

**Desktop:**
- Mouse: Move to steer, click to fire
- Arrows/WASD: Move
- Space: Fire
- X/Shift: Bomb
- P/Esc: Pause

**Mobile:**
- Left side drag: Move (visual joystick)
- FIRE button: Tap to fire
- BOMB button: Tap to bomb

Priority: Keyboard/joystick > mouse steering (mouse only active if no keys pressed).

## Key Game Systems

### Weapons
**Vulcan** (default):
- Level 1: Single center shot
- Level 2: Two-shot spread
- Level 3: Three-shot spread
- 0.15s fire rate, 500 px/s bullet speed

**Laser**:
- Level 1: Single center shot
- Level 2: Two-lane shot
- Level 3: Three-lane shot
- Pierce through up to 3 enemies
- 0.1s fire rate, 640 px/s bullet speed

### Combo System
- **Chain**: Incremented on every enemy kill, reset if 2.2s passes without a kill
- **Multiplier**: x1 baseline, up to x10 (1 per 5-kill interval)
- Score = base points × multiplier
- Visual feedback: "N KILLS!" popup every 10 kills

### Boss Attack Patterns
Each boss has 3 unique attack patterns (cycles continuously):
- **Radial**: Bullets spread in all directions
- **Aimed spread**: Bullets fan toward player with randomness
- **Specialized**: Teleport, homing, cross-fire, spirals, etc.

Telegraph phase (22 frames) shows core glowing bright white before attack fires.

### Collision Detection
Simple AABB (axis-aligned bounding box) with tolerance:
```javascript
rectsOverlap(a, b) => abs(a.x - b.x) < (a.w + b.w)/2 && abs(a.y - b.y) < (a.h + b.h)/2
```

### Audio Synthesis
All sounds generated on-the-fly; no audio files. Web Audio API features used:
- **Oscillators**: Sine, square, sawtooth (for tonal SFX)
- **Noise**: BufferSource with random data (for explosions, impacts)
- **Envelopes**: Linear/exponential gain ramps (ADSR-style)
- **Filters**: Low-pass, high-pass frequency filtering
- **Stereo panning**: `createStereoPanner` for spatial effects
- **Compression**: Master compressor to prevent clipping

Key SFX categories:
- `beep()`: Tonal effects (shoot, boss telegraph, power-up)
- `noiseBurst()`: Percussive effects (explode, bomb)
- `sfx` object: Named combinations (e.g., `explodeTank()` = noise + low-freq burst)

## Common Development Tasks

### Running the Game
1. Open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
2. Or use a local HTTP server: `python -m http.server 8000` then visit `http://localhost:8000`
3. Game starts with a "Start Game" overlay; select difficulty and click "Start Game"

### Testing Specific Features

**Test a new enemy type:**
- Add to `ENEMY_TYPES` object with `w, h, hp, speedMul, color, dark, score, minStage` properties
- Optionally add `weave: true` for sinusoidal movement
- Update spawn logic in `spawnEnemy()` to include the new type based on stage or probability

**Test a new weapon:**
- Add weapon name to `fireWeapon()` conditional
- Define behavior (spread angles, pierce count, fire rate, projectile speed)
- Update `weaponLabel()` if the weapon name differs from the key

**Test boss patterns:**
- Add 3 new functions to the file (e.g., `fireNewPattern()`)
- Add them as an array to `BOSS_PATTERNS` in the correct index
- Boss selection cycles: `BOSSES[(stage - 1) % BOSSES.length]`

**Test difficulty balancing:**
- Modify `DIFFICULTIES` object values
- Restart game and cycle difficulty in the overlay
- Observe spawn rate, enemy speed, boss health changes

### Adding a Stage Theme
Add object to `THEMES` array with:
```javascript
{ sky: ['top', 'mid', 'bottom'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
```

Sky is a linear gradient top-to-bottom. Cloud and sun are RGB for rendering clouds and glow. Themes cycle: stage N uses `THEMES[(N-1) % THEMES.length]`.

### Performance Tuning

Quality levels (0=full, 1=medium, 2=low) are auto-scaled based on frame time:
- Average frame time > 1/40s: increase quality reduction
- Average frame time < 1/56s: reduce quality reduction

Quality affects:
- Particle spawn counts: `Math.max(3, Math.round(count * (quality === 0 ? 1 : quality === 1 ? 0.6 : 0.35)))`
- Glow effects: Only rendered when `quality === 0`
- Shadow blur: Reduced or disabled at higher quality levels

### Debugging Tips

**Console logging:**
- Game state is intentionally minimal; add `console.log(player, boss, enemies)` during update/draw
- Frame counter: `frame` variable increments every update; useful for timing events

**Visual debugging:**
- Toggle reduced motion: Click 🎆 button to disable screen shake/flash
- Boss health bar: Always visible during boss fight; shows remaining HP
- Combo display: "N KILLS!" appears every 10 kills

**Storage inspection:**
- Best score: `localStorage.getItem('skyfighter.bestScore')`
- Difficulty: `localStorage.getItem('skyfighter.difficulty')`
- Muted: `localStorage.getItem('skyfighter.muted')`
- Reduced motion: `localStorage.getItem('skyfighter.reducedMotion')`
- Clear all: Run `Object.keys(localStorage).filter(k => k.startsWith('skyfighter')).forEach(k => localStorage.removeItem(k))` in console

## Code Style & Conventions

- **No build step**: Code runs directly in browser; no transpilation
- **No external libraries**: Only vanilla JavaScript and browser APIs
- **IIFE wrapper**: All code in `(function() { 'use strict'; ... })()` for scope isolation
- **camelCase**: Variables, functions, object keys
- **Constants**: UPPERCASE (e.g., `DIFFICULTIES`, `BOSSES`, `THEMES`, `KILLS_PER_STAGE`)
- **Comments**: Minimal; section markers like `// ---------- Audio ----------` separate major systems
- **Magic numbers**: Balancing values (speeds, cooldowns, spawn rates) are hardcoded; adjust via object configs when adding difficulty/stage/enemy variety

## Local Storage Keys

| Key | Value | Purpose |
|-----|-------|---------|
| `skyfighter.bestScore` | Number | Highest score achieved |
| `skyfighter.difficulty` | String (easy/normal/hard) | Selected difficulty |
| `skyfighter.muted` | '0' or '1' | Sound muted state |
| `skyfighter.reducedMotion` | '0' or '1' | Reduced motion enabled |

## Accessibility Features

- **Screen reader support**: `aria-live` region (`#srLive`) announces game events (stage, boss defeat, lives remaining, etc.)
- **Reduced motion**: Respects `prefers-reduced-motion` media query; button toggles override
- **Keyboard support**: All controls accessible via keyboard
- **Color contrast**: UI text uses high-contrast white on dark backgrounds
- **No seizure risk**: Screen shake and flash respect reduced motion setting; animated text uses safe pulse rates

## Testing Checklist for Changes

- [ ] Game starts without console errors
- [ ] Player movement works (keyboard, mouse, touch)
- [ ] Firing works; bullets visible and collision kills enemies
- [ ] Bombs work; clear all bullets/enemies on screen
- [ ] Power-ups drop, are collected, weapon/life/bomb updates visible
- [ ] Boss spawns after 18 kills; health bar appears and decreases on hits
- [ ] Boss attack patterns fire correctly; player can dodge
- [ ] Difficulty selection works; restart applies new settings
- [ ] Score and best score update correctly with multiplier
- [ ] Game over screen shows final score and stage
- [ ] Mute button toggles audio
- [ ] Reduced motion button toggles effects (screen shake, flash, vignette pulse)
- [ ] All 6 stage themes appear as stage progresses
- [ ] Mobile touch controls work (joystick moves, FIRE/BOMB buttons)
- [ ] localStorage saves/loads correctly after page reload
