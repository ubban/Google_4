# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built entirely in vanilla HTML5/Canvas with no external dependencies. The entire game is implemented in a single `index.html` file (~1700 lines).

## Quick Start

Since this is a static HTML file with no build system:

```bash
# Open the game in a browser
open index.html
# or from terminal:
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

**Development**: Simply edit `index.html` and refresh the browser to see changes. No build step required.

## Architecture & Code Organization

The game is structured as a single monolithic script (~1300 lines of JavaScript) organized into these conceptual sections:

### Canvas & Rendering
- **Canvas setup** (line 752-765): 480×720 base resolution, responsive scaling
- **Drawing functions**: 
  - `drawPlane()` - renders player and enemy ships (parametric design based on color/shape)
  - `drawBossShip()` - boss-specific rendering with animated core
  - `drawClouds()` - parallax cloud background layers
- **Effects**: screen shake, flash effects, vignette for critical health, particle system
- **Render order**: sky gradient → sun glow → clouds → player → bullets → enemies → boss → power-ups → particles

### Game State
- **Player**: position, velocity, weapon (vulcan/laser), ammo level, invulnerability timer
- **Entities**: bullets (player), enemyBullets (enemies), enemies array, powerUps, boss, particles
- **Score system**: kills chain, multiplier (1–10x based on consecutive kills), best score persisted to localStorage
- **Lives system**: configurable per difficulty (3–4 lives), bombs for screen-clear
- **Stages**: increase procedurally; every 18 kills triggers a boss fight

### Input Handling
- **Desktop**: Mouse steering (smooth glide to pointer), keyboard (arrows/WASD for movement, Space to fire, X/Shift for bomb)
- **Mobile**: Joystick in left zone, FIRE/BOMB buttons in right zone; touch events drive full pointer emulation
- **Pause**: P or Escape key
- Keyboard and joystick take priority over mouse to avoid fighting inputs

### Difficulty & Progression
- **Difficulties** (line 485–489): Easy/Normal/Hard with tuned spawn rates, enemy speed, shoot chance, boss HP scaling
- **Enemy types** (line 502–506): Grunt (1 HP), Interceptor (fast, 1 HP), Tank (slow, 4 HP); minStage gates spawning
- **Boss system** (line 508–515): 6 unique designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with stage-specific attacks
- **Boss attack patterns** (line 1372–1409): Each boss has 3 distinct patterns: radial bursts, aimed spreads, homing barrages, etc.

### Audio (WebAudio Synthesis, No Samples)
- **Setup** (line 542–595): Lazy-init AudioContext, dynamics compressor (brick-wall limiter effect), mute toggle
- **Sound functions**:
  - `beep()` - sine/square/sawtooth oscillator with sweep, filter, pan, gain envelope
  - `noiseBurst()` - white noise with filter for explosions/impact
- **Sound effects** (line 624–676): shoot, laser, explode variants, hit, bomb, powerup, boss telegraph, stage clear, game over
- **Spatial audio**: Pan based on X position (`panFor()`)

### Collision Detection
- **Simple AABB** (line 1069–1071): `rectsOverlap()` checks all entity hitboxes
- **Hits tracked**: enemy ↔ bullet, boss ↔ bullet, player ↔ enemy, player ↔ enemyBullet, player ↔ boss, player ↔ powerUp

### Animation & Visual Effects
- **Screen shake**: `screenShake(magnitude, duration)` adds random jitter to canvas translation
- **Flash effects**: white flash on hit/bomb detonation
- **Combo display**: large floating text near screen center when kill milestones hit
- **Stage banner**: large text overlay on stage transitions
- **Boss health bar**: top-center bar with boss name, real-time HP tracking

### Performance Adaptation
- **Adaptive quality** (line 473–483): Tracks frame time over 45 frames; downscales particle counts and shadow effects (quality levels 0/1/2) if frame rate dips below ~40 FPS or improves above ~56 FPS
- **Reduced motion**: CSS animation opt-out for accessibility; controlled via UI toggle

### Persistence
- **localStorage keys**:
  - `skyfighter.bestScore` - high score
  - `skyfighter.muted` - audio toggle
  - `skyfighter.difficulty` - last selected difficulty
  - `skyfighter.reducedMotion` - motion preferences

## Key Technical Details

### Weapon System
- **Vulcan** (line 1126–1134): Spread-shot machine gun
  - Level 1: single center shot
  - Level 2: dual offset shots (±0.12 rad spread)
  - Level 3: triple spread (±0.24 rad)
  - Fire rate: 0.15s cooldown
- **Laser** (line 1135–1145): hitscan-style piercing beam
  - Level 1: single center lane
  - Level 2: dual side lanes
  - Level 3: triple lanes (dual side + center)
  - Fire rate: 0.1s cooldown (faster)
  - Pierces enemies up to pierce level count

### Enemy Behavior
- **Movement**: constant downward velocity (speed adjusted by type × difficulty multiplier) + optional weave (sine wave left/right)
- **Shooting**: random delay (60–150 frames), then either spread pattern (3-way fan) or aimed (homing at player)
- **Death**: spawns 3–22 particles (quality-scaled), occasional power-up drop (6% life, 8% bomb, 20% weapon)

### Boss Behavior
- **Entry**: slides in from top over ~2 seconds, then settles at y=120
- **Movement**: bounces left/right at canvas edges at 55 px/s
- **Attack cycle**: 
  1. Telegraph phase: 22 frames of blinking core + sound
  2. Fire: execute attack pattern (radial, aimed, homing, or geometric)
  3. Cool: 85 frames before next telegraph
- **Pattern selection**: rotates through 3 patterns per boss design; each boss has unique bullet geometries

### Scoring & Multiplier
- **Base scores**: Grunt 10, Interceptor 15, Tank 35, Boss 500
- **Chain system**: Each kill increments chain; resets on death or 2.2s no-kills timeout
- **Multiplier**: 1 + floor(chain / 5), capped at 10x (at 50+ chain)

## Performance Considerations

- **No library overhead**: Pure Canvas 2D API, minimal allocations per frame
- **Object pooling**: Bullets, enemies, particles created/destroyed as needed; no pre-allocation
- **Spatial tricks**: 
  - Enemies auto-cull when y > canvas.height + 30
  - Bullets auto-cull when off-canvas
  - Boss only active during boss stage (no extra update cost)
- **Quality scaling**: Particle counts, shadow effects, animation intensity drop when CPU-bound
- **Audio caching**: SFX functions are pure, no sample loading; synthesis latency ~5ms

## Browser Compatibility

- **Canvas 2D API**: All modern browsers (Chrome, Firefox, Safari, Edge)
- **WebAudio API**: Required for sound (falls back silently if unavailable)
- **localStorage**: Optional (graceful degrade if disabled)
- **Touch events**: Full support; includes pointer emulation for mobile
- **Reduced motion media query**: Respects `prefers-reduced-motion: reduce` by default

## Future Improvement Areas (Guidance for Contributors)

1. **High scores persistence**: Currently localStorage-only; could integrate cloud sync
2. **Difficulty balance**: Boss HP multipliers and spawn rates are tuned via `DIFFICULTIES` constant; test extensively before adjusting
3. **Visuals**: Themes system (`THEMES` array) is extensible; add more sky/cloud/sun color schemes
4. **Enemy variety**: New enemy types can be added to `ENEMY_TYPES` with custom rendering in main draw loop
5. **Boss variety**: New bosses go in `BOSSES` array with new patterns in `BOSS_PATTERNS` (keep patterns aligned by index)
