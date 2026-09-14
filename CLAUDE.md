# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:

- Vertical scrolling gameplay with procedural enemy spawning
- Weapon upgrade system (Vulcan → Laser)
- Smart bomb mechanic with limited charges per stage
- Multi-stage progression with unique boss encounters
- Three difficulty levels (Easy, Normal, Hard) with persistent settings
- Touch and keyboard controls optimized for mobile and desktop
- Procedurally generated audio using WebAudio API (no external sound assets)
- Adaptive performance optimization based on frame time
- Score multiplier system based on kill chains

## Architecture

The entire game is contained in `index.html` (~1670 lines) split into these functional blocks:

### Rendering & Canvas
- Fixed 480×720 canvas base resolution (responsive scaling up to 1.6x)
- 2D canvas context for all drawing (planes, enemies, particles, effects)
- Separate HUD layer (html overlay) for score, weapon info, health
- Flash overlay for hit/bomb feedback
- Vignette effect for critical health warning
- Stage banners and combo pop notifications

### Audio System
- WebAudio synth-based sound effects (no external files)
- `ensureAudio()` initializes AudioContext on first user interaction
- Compressor chain on master gain for dynamic range
- `beep(freq, dur, type, vol, opts)` generates tones with optional sweep/filter/pan
- `noiseBurst(dur, vol, opts)` generates noise for explosions
- Pan position calculated from screen X coordinate for spatial audio
- Mute state persisted to localStorage

### Input Handling
- **Keyboard**: Arrow keys/WASD move, Space/Click fire, X/Shift bomb, P/Esc pause
- **Mouse**: Plane glides toward pointer (non-intrusive steering)
- **Touch**: Virtual joystick (left side) + FIRE/BOMB buttons (bottom right)
- Joystick constrained to 50px radius with smooth movement
- Mouse input takes priority only when joystick inactive
- Input state tracked in `keys` object and `joyVec` for movement

### Game State
Core variables initialized in `resetGame()`:
- `player`: x, y, w, h, speed, cooldown, weapon, level, invulnerability
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: entity arrays
- `boss`: single boss object (null when inactive)
- `score`, `lives`, `stage`, `kills`, `chain`, `multiplier`
- Difficulty settings cached in DIFFICULTIES object

### Collision & Physics
- AABB collision detection used throughout (rectangular bounding boxes)
- No external physics library—all velocity/position updates manual
- Particles fade over time (`life` / `maxLife` ratio)
- Enemy movement applies `speedMul` based on type
- Player movement clamped to canvas bounds

### Enemy System
- Three regular enemy types: `grunt`, `interceptor`, `tank` (stats in ENEMY_TYPES)
- Each type has color, dark shade, HP, score value, min stage
- Interceptors have `weave: true` flag for sine-wave movement
- Spawning rate scales with difficulty (spawnBase, spawnMin)
- Enemy AI: random vertical movement, shoot on timer with difficulty-based chance
- Stage advancement: kill 18 enemies per stage (`KILLS_PER_STAGE`)

### Boss System
- Six unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each boss has distinct hull colors and core color
- Boss HP scales with stage and difficulty (hpMul factor)
- Telegraphing mechanic: boss flashes white before attack
- Attacks spawn radial bullet patterns
- Boss defeated at stage boundaries (automatic progression)

### Game Loop
- `loop(now)` called via requestAnimationFrame
- Delta time capped at 50ms to prevent large jumps
- Frame time tracked in `frameTimes` array (45-frame rolling average)
- Adaptive quality: adjusts based on avg frame time vs target (40fps low/56fps high)
- Calls `update(dt)` then `draw()` when running

### Difficulty Multipliers
Three preset profiles in DIFFICULTIES object:
- **Easy**: 4 lives, 3 bombs, slower enemy spawning, lower enemy shoot rate
- **Normal**: 3 lives, 2 bombs, balanced (default)
- **Hard**: 3 lives, 2 bombs, rapid enemy spawning, aggressive shooting, tougher bosses

## Key Mechanics

### Weapon System
- Starts with Vulcan (stream of bullets)
- Laser power-up upgrades to Laser (faster, different SFX)
- Can downgrade back to Vulcan (only one weapon type active)
- Level cap 3 (each pickup increments level)

### Chain & Multiplier
- Killing enemies without getting hit extends chain
- Chain resets on player death
- Multiplier: 1x base + 0.1x per chain kill (max ~2x)
- `chainTimer` counts down; hitting enemies resets it

### Smart Bombs
- Limited count per stage (restored on stage clear)
- Clears all enemy bullets and some enemies
- Full-screen damage with radial effect
- Bomb animation and SFX separate from explosions

### Screen Shake
- `screenShake(mag, dur)` triggers camera shake
- Reduced to 15% intensity if reduced motion enabled
- Duration capped to 0.1s in reduced motion mode

### Power-Ups
- Three types: weapon upgrade, bomb, health
- Spawn on random enemy deaths
- Float downward; collected on contact
- Brief particle burst on collection

## Common Tasks

### Adjust Difficulty Balance
Edit `DIFFICULTIES` object (line ~485) to tweak spawn rates, enemy behavior, lives/bombs, boss scaling.

### Add New Enemy Type
1. Add entry to `ENEMY_TYPES` with stats (color, hp, speed, score, minStage)
2. Add drawing code to `drawEnemies()` (around line ~1100)
3. Reference in `spawnEnemy()` (line ~1200)

### Modify Weapon Behavior
- Vulcan bullet spray: adjust spread angle and count in update loop
- Laser: change sweep frequency and bullet size
- Both use cooldown timer on player object

### Tweak Visual Effects
- Boss colors: update BOSSES array designs
- Enemy colors: edit ENEMY_TYPES color/dark values
- Screen themes: modify THEMES array (sky gradients, cloud/sun colors)

### Adjust Performance Settings
- Target frame rates in `trackPerf()` (1/40 for low quality, 1/56 for high)
- Particle glow rendering: `if (glow)` checks quality level (line ~1650)
- Boss drawing complexity: reduce draw calls or skip details at low quality

## Input & Mobile
- Joystick radius: 50px (adjust `maxDist` in `joystickMove`)
- Touch controls auto-enable if `'ontouchstart'` detected
- Mouse always active when over canvas (falls back to keyboard if no mouse movement)
- Buttons use `touchstart`/`touchend` with `passive: false` for preventDefault
- Screen shake disabled in reduced motion mode (accessibility)

## Audio & Performance
- All audio is synthesized (no file loading overhead)
- Compressor prevents audio clipping on rapid SFX
- Pan value clamped to [-1, 1] based on screen X
- Master gain set to 0.9 to leave headroom
- Mute button persisted to localStorage

## Responsive Design
- Base canvas: 480×720
- Scaling applied in CSS via `max-width: 100%` and inline `width`/`height`
- Safe area insets applied for notch devices
- HUD elements use clamp() for responsive font sizes
- Touch control zones sized relative to viewport
