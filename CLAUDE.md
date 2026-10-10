# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single self-contained HTML5 file. The game features:
- Canvas-based 2D graphics rendering
- Procedurally generated Web Audio API sound effects (no audio files)
- Three difficulty levels with scaling enemy behavior
- Multiple weapon types and power-up system
- Boss encounters with distinct attack patterns
- Mobile touch controls and desktop keyboard/mouse input
- LocalStorage persistence for scores and settings
- Adaptive performance quality scaling

## Architecture

The entire game is in `index.html` with three main sections:

### HTML & DOM (lines 1-420)
- Canvas element for rendering
- HUD overlay showing score, lives, bombs, weapon status
- Boss health bar
- Screen flash and vignette effects for feedback
- Pause/game-over overlay
- Touch control zone for mobile

### CSS Styling (lines 10-359)
- Responsive design using viewport meta tags and clamp()
- Dark theme with neon colors (#2196f3, #ffd54f, #ff5252)
- Canvas centered with aspect ratio preservation
- Animations for critical states: heartbeat (low health), combo popups, stage banners, flash effects
- Reduced motion support via `prefers-reduced-motion` media query
- Touch action handling to prevent browser defaults

### JavaScript Game Logic (lines 422-1666)

**Game State & Loop (lines 863-1652)**
- Single `loop()` function runs at ~60fps via requestAnimationFrame
- `update(dt)` handles physics, collisions, spawning
- `draw()` renders all game objects with adaptive quality
- Performance tracking adjusts particle count/shadow quality dynamically

**Key Game Objects**
- `player`: Position, health, weapon level, cooldown timers, invulnerability frames
- `enemies`: Array with type-specific properties (grunt, interceptor, tank), weaving patterns, shooting behavior
- `boss`: Multi-phase encounter with telegraph warning before attacks, distinct patterns per boss design
- `bullets`, `enemyBullets`, `particles`, `powerUps`: All cleared each frame when off-screen

**Gameplay Systems**
- **Difficulty Scaling**: Three sets of constants (DIFFICULTIES) control spawn rate, enemy speed, shoot chance, starting lives/bombs, boss HP multiplier
- **Weapon System**: Two weapons (Vulcan spread-shot, Laser piercing beam) with 3 upgrade levels each that expand coverage/pierce count
- **Scoring & Combo**: Chain counter multiplies points for consecutive kills; resets on hit or after 2.2s of no kills
- **Boss Patterns** (BOSS_PATTERNS array): Each boss has 3 distinct attack sequences cycling through radial spreads, aimed shots, homing fire, teleport barrages, etc.
- **Power-ups**: 6% weapon, 8% bomb, 6% life; spawn randomly on enemy death

**Audio (lines 542-676)**
- Web Audio synth using OscillatorNode for tonal SFX and noise bursts
- Procedural sound design: shoot beeps, laser sweeps, explosion frequency sweeps, boss telegraph
- Pan positioning based on event location for stereo field effect
- Muted toggle stored in localStorage

**Input Handling**
- Keyboard: Arrow keys/WASD for movement, Space for fire, X/Shift/P for bomb/pause
- Mouse: Hover to aim player aircraft toward pointer, click to fire
- Touch: Left half of screen is joystick, right buttons for FIRE/BOMB
- Joystick input priority: keyboard > joystick > mouse (mouse only moves if no keyboard/joystick active)

**Rendering Pipeline**
- Sky gradient with parallax clouds and sun glow
- Enemy and player planes drawn with computed fill/dark colors
- Boss ship with animated glow core
- Bullets with shadow glow (quality-dependent)
- Particle system for explosions (count scales with quality setting)
- Screen shake (reduced in reduced-motion mode)

**Theme System (THEMES array)**
- 6 environment themes cycling per stage with unique sky gradients and cloud colors
- BOSSES array defines 6 unique boss designs with hull color gradients and core glow

## Development Workflow

### Running the Game
Simply open `index.html` in a browser (Firefox, Chrome, Safari all supported). Game runs at fullscreen with responsive canvas scaling.

### Testing Locally
- Use Firefox's canvas/rendering inspector to debug draw calls
- Browser DevTools console for sound object testing: `sfx.shoot()`, `sfx.bomb()`, etc.
- Throttle CPU in DevTools to test low-quality mode (quality var should ramp up when avg frame time > 1/40s)

### Common Development Tasks

**Add a New Boss**
1. Add entry to `BOSSES` array with `{ name, hullTop, hullBottom, wing, core, hpMul }`
2. Add attack pattern array to `BOSS_PATTERNS` (3 function patterns each firing enemy bullets in various formations)
3. Boss is auto-selected by index: `BOSSES[(stage - 1) % BOSSES.length]`

**Adjust Game Balance**
- `DIFFICULTIES` object controls spawn rates, enemy speed, bullet patterns, lives/bombs
- `KILLS_PER_STAGE` (18) determines when boss appears
- Individual enemy type stats in `ENEMY_TYPES`: hp, speedMul, shootChance, score
- Power-up spawn rates in `maybeSpawnPowerUp()`: 6% weapon, 8% bomb, 6% life

**Add Sound Effects**
- New beep: `beep(frequency, duration, type, volume, opts)` with optional sweepTo, filterFreq, pan, delay
- New noise: `noiseBurst(duration, volume, opts)` with filterType, filterFreq, pan
- Add to `sfx` object as a named function

**Modify Difficulty/Leaderboard**
- High scores stored in localStorage under `skyfighter.bestScore`
- Difficulty persisted in `skyfighter.difficulty`
- Change `STORAGE_*` constants if renaming keys

**Optimize Performance**
- quality variable (0=full, 1=medium, 2=low) automatically scales particle count and shadow effects
- Reduce `frameTimes` window size (currently 45 frames) if needing faster adaptation
- Adjust particle spawn count in `spawnExplosion()` based on quality

## Key Implementation Details

**Collision Detection**: Simple AABB (axis-aligned bounding box) via `rectsOverlap()` — checks absolute distance between centers against summed half-widths.

**Fixed Frame Time**: `update(dt)` receives clamped delta (max 0.05s per frame) so high-frame-rate systems don't cause overflow.

**Invulnerability System**: Player gains 1.5s of invuln after hit, blinks every 4 frames while active (reduced-motion disables blink animation).

**Canvas Scaling**: Fixed 480×720 internal resolution scaled to fit window up to 1.6x while maintaining aspect ratio.

**Mobile Accessibility**: Full touch joystick on left half, FIRE/BOMB buttons bottom-right; all touch zones use `pointer-events: auto` to avoid page scroll.

**Reduced Motion**: Three code paths check `reducedMotion` flag — particle effects scale down, animations become instant, vignette/flash severity reduces to 15% of normal.

## Browser Compatibility

Requires:
- Canvas 2D context
- Web Audio API (gracefully degrades sound if unavailable)
- localStorage
- requestAnimationFrame
- Touch Events API (for mobile)

Tested on: Chrome, Firefox, Safari (desktop and iOS), Android Chrome.
