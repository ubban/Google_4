# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 file with no external dependencies. The entire game (HTML, CSS, JavaScript) is contained in `index.html`.

## Quick Start

- **View the game**: Open `index.html` in any modern browser
- **No build system**: This is a standalone HTML file with embedded styles and scripts
- **No dependencies**: No npm, build tools, or external libraries required

## Architecture

The project is a single-file canvas-based game with the following structure:

### Game State (`global scope`)
- **Player**: Position, health, weapon, cooldown
- **Entities**: Bullets (player and enemy), enemies, power-ups, particles, boss
- **Score/Progression**: Score, lives, multiplier, stage counter, difficulty
- **Input**: Keyboard keys, mouse position, joystick state for touch

### Core Game Loop
The `loop(now)` function runs on `requestAnimationFrame` and is the heartbeat:
1. **Track performance** (`trackPerf`) — adaptive quality scaling based on frame time
2. **Update** (`update(dt)`) — entity physics, collisions, spawning
3. **Draw** (`draw()`) — render scene and all entities
4. **Screen shake** — applied during canvas transform (performance optimization)

### Entity Systems

**Player** (`player` object)
- Steerable with keyboard (arrow keys/WASD), mouse (click-to-aim), or touch (joystick)
- Two weapon types: Vulcan (spread shots) and Laser (piercing beams), each upgradable to level 3
- Invulnerability period after hit; collision boxes aligned with rendered plane

**Enemies** (`enemies` array)
- Three types: Grunt, Interceptor (weaving), Tank (multi-HP)
- Each type has difficulty-gated spawn (tank only appears stage 2+)
- Movement: straight descent + optional weave pattern
- Shooting: either spread pattern or aimed-at-player pattern
- Despawn when off-screen

**Boss** (`boss` object)
- Spawns after 18 enemy kills per stage; 6 unique designs cycle by stage
- Enters screen, bounces horizontally, telegraphs attacks before firing
- 6 unique bullet patterns per boss (radial, aimed spreads, teleport + barrage, etc.)
- Destroys all enemy bullets on defeat; advances to next stage

**Bullets**
- Player bullets: Track pierce count; lasers pierce up to 2 enemies
- Enemy bullets: Either tracked velocity or radial (for boss patterns)
- Collisions are rect-based; player bullets deal 1 hp to regular enemies, 3 hp to boss

**Power-ups** (random drops from enemy kills)
- Life: Restore health
- Bomb: Add to bomb count (destructive 1-frame-per-game-frame clear)
- Weapon: Upgrade weapon level or switch type

### Rendering Pipeline

**Sky gradient & parallax clouds** (`drawClouds`, theme-based)
- 6 themes cycle by stage (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- Two cloud layers at different speeds create depth

**Entities**
- Planes drawn procedurally (`drawPlane`) with color/dark variant
- Boss drawn with rotation glow on core (telegraph phase)
- Bullets with optional glow (quality-dependent)
- Particles (explosion debris) with alpha fade
- Power-ups rotate and glow

**HUD** (DOM overlays, never cleared)
- Top: Score, best, multiplier, weapon, hearts, bombs
- Boss bar: Name and health (only shown when boss active)
- Status: Combo pop, stage banner, vignette flash on critical health
- Touch controls (on mobile): Joystick + fire/bomb buttons

### Input Handling

**Keyboard/Mouse**
- WASD/Arrows: Move player
- Space: Fire
- X/Shift: Drop bomb
- P/Esc: Toggle pause
- Mouse: Click to fire, move to steer (lower priority than keyboard)

**Touch**
- Left 55% of screen: Joystick movement zone
- Right 45% of screen: Fire and bomb buttons
- Auto-detect with `ontouchstart` check

**Audio Context**
- WebAudio synth (no audio files)
- Lazy-init on first input (`ensureAudio`)
- Mute state persists in localStorage

### Difficulty Tuning

Three difficulty presets affect:
- `spawnBase` / `spawnMin`: Enemy spawn interval
- `enemySpeed`, `enemyShootChance`: Combat intensity
- `lives`, `bombs`, `bossHpMul`: Resource allocation

Selected difficulty persists in localStorage; applied at game start.

### Performance Optimization

**Adaptive quality** (`quality` variable: 0 = full, 2 = low)
- Tracks average frame time over 45 frames
- Reduces particle count, glow effects at lower quality levels
- Screen shake reduced in reduced-motion mode

**Reduced motion support**
- Detects `prefers-reduced-motion` media query
- Disables animations (screen shake, flash, pulse vignette, bobbing icons)
- Toggle persists in localStorage

### Storage Keys

All persisted to localStorage:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Motion preference override

## Making Changes

**Common edits:**

- **Difficulty balance**: Adjust `DIFFICULTIES` constants (spawnBase, enemySpeed, etc.)
- **Enemy variety**: Edit `ENEMY_TYPES` or add new types to `spawnEnemy()`
- **Boss patterns**: Add/modify functions in `BOSS_PATTERNS` array (e.g., `fireRadial`, `fireAimedSpread`)
- **Visuals**: Adjust colors in `THEMES` or plane rendering in `drawPlane()`, `drawBossShip()`
- **Weapon balance**: Modify spread patterns in `fireWeapon()` or pierce logic
- **Audio**: Edit SFX definitions in `sfx` object (frequency, duration, filter type)

**Important invariants:**
- Canvas is always 480×720 (scaled in CSS); scale down gracefully with quality setting
- Collision detection uses rect-based overlap; adjust entity `w`/`h` to tune
- Boss entrance checks `b.entering` flag; don't modify position during this phase
- Particle count scales with quality; never hard-code particle spawns
- Screen shake is no-op in reduced-motion mode; always check before applying visual effects

**Testing:**
- Open in browser and play through all difficulties
- Verify audio (mute/unmute), motion effects, and touch controls on mobile
- Check that bosses spawn at stage transitions and behave as designed
- Confirm high score persists across page reloads
