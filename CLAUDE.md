# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file web application (`index.html`) that requires no build process or dependencies. The game runs entirely in the browser using Canvas 2D rendering and Web Audio API for sound synthesis.

## Development

### Running the Game

Open `index.html` directly in a browser. No build tools, server, or installation required. The game starts with a title screen where players can select difficulty and begin playing.

### Game Features

- **Three difficulty levels**: Easy, Normal, Hard (affects enemy spawning, speed, shoot chance, and boss HP)
- **Three enemy types**: Grunts, Interceptors, Tanks (with different stats and HP)
- **Two weapon types**: Vulcan (spread shot) and Laser (piercing, narrow beam)
- **Boss system**: 6 unique boss designs with distinct attack patterns across multiple stages
- **Power-ups**: Life restoration, bombs, weapon upgrades
- **Scoring system**: Chain multiplier that increases with rapid kills
- **Local storage persistence**: Best score, difficulty choice, mute state, reduced motion preference
- **Accessibility**: Screen reader support, reduced motion support (respects `prefers-reduced-motion`), pause system
- **Mobile controls**: Virtual joystick for movement, FIRE and BOMB buttons on touch devices
- **Desktop controls**: Mouse steering (optional), keyboard (Arrow/WASD move, Space fire, X/Shift bomb), click to fire

## Architecture

### Game State (lines 863–900)

All active game state is stored in module-scope variables:
- **Player**: position, health, weapon type, level, cooldown timers
- **Entities**: `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Score tracking**: `score`, `kills`, `stage`, `chain`, `multiplier`
- **Runtime flags**: `running`, `paused`, `gameOver`

### Core Game Loop (lines 1640–1665)

The `loop()` function runs via `requestAnimationFrame`:
1. Calculate delta time (clamped to 50ms max to prevent large jumps)
2. Track performance metrics for adaptive quality reduction
3. If running and not paused, call `update(dt)` then `draw()`

### Update System (lines 1148–1302)

The `update(dt)` function handles all game logic:

- **Input processing** (lines 1152–1170): Keyboard (Arrow/WASD), gamepad-style joystick, mouse target
- **Player update** (lines 1174–1181): Movement clamping, invulnerability, firing cooldown
- **Weapon firing** (lines 1124–1146): Vulcan (spread) and Laser (pierce) weapon variants scaled by weapon level
- **Entity updates**:
  - Bullets move upward
  - Enemy bullets move downward (with optional horizontal velocity)
  - Enemies move down with optional weaving pattern
  - Power-ups fall downward
- **Collision detection** (lines 1232–1294):
  - Player bullets vs. enemies
  - Player bullets vs. boss
  - Enemy bullets vs. player
  - Enemies vs. player
  - Power-ups vs. player
- **Boss AI** (line 1230): Delegates to `updateBoss(dt)`
- **Chain/multiplier decay** (lines 1183–1186): Timeout on score chain

### Collision System (line 1069–1071)

Simple AABB (axis-aligned bounding box) overlap using `rectsOverlap()`:
```javascript
Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2
```

### Rendering System (lines 1520–1638)

The `draw()` function renders the game world:
- **Screen shake**: Applies random jitter via canvas translate (if `shakeTime > 0`)
- **Sky**: Gradient background with sun glow (theme-based)
- **Clouds**: Parallax scrolling clouds for visual depth
- **Player ship**: Custom drawn polygon with flame effect
- **Bullets**: Rectangles (Vulcan) or circles (Laser) with shadow glow
- **Enemy ships**: Drawn as polygons (same style as player but different colors)
- **Boss ships**: Complex polygon shapes with animated core glow (line 970–1020)
- **Particles**: Explosion debris that fades out
- **Power-ups**: Rotating squares with color-coded glow
- **HUD**: Updated via DOM (not canvas) — score, best, multiplier, hearts, bombs, boss health bar

Rendering quality (glow, particle count) is adaptive based on frame time performance (lines 473–483).

### Audio System (lines 542–676)

**WebAudio synth** generates all sound effects dynamically:
- Uses oscillators (square, sine, sawtooth) and noise generation
- Supports frequency sweeps, filters, and stereo panning
- Each sound effect is a function in the `sfx` object (shoot, laser, explode, bomb, powerup, etc.)
- `ensureAudio()` creates the audio context on first user interaction (to comply with browser autoplay policy)
- `muted` flag silences all audio; stored in localStorage

Key patterns:
- **Beep**: Frequency sweep with optional filter and pan
- **Noise burst**: White noise with frequency filter
- **Combined**: Multiple beeps/bursts layered with delays for complex sounds

### Input Handling

**Keyboard** (lines 768–776):
- Arrow keys or WASD for movement
- Space bar for fire
- X or Shift for bomb
- P or Escape to pause

**Mouse** (lines 836–861):
- Move over canvas to set target steering position
- Click to fire (begins firing, held until mouse leaves canvas)
- Target position stored in `mouseTarget`; player glides toward it via easing

**Touch** (lines 778–834):
- Virtual joystick on left 55% of screen (movement)
- FIRE and BOMB buttons on right 45% (action zone)
- Joystick base appears only while touching

**Keyboard/Joystick/Mouse priority**: Keyboard and joystick take priority over mouse steering when active, preventing conflicts.

### Enemy Spawning (lines 1022–1041)

- Spawn interval decreases as game progresses (difficulty ramp at line 1201)
- Enemy type distribution changes based on stage (tanks appear at stage 2+)
- Interceptors "weave" side-to-side using sine wave
- Enemies shoot in either "aimed" (homing toward player) or "spread" (fixed angles) patterns
- When 18 kills (`KILLS_PER_STAGE`) reached, boss spawns after 1.2s warning

### Boss System (lines 1043–1056, 1411–1439)

- 6 unique boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each design has distinct hull colors, core color, and attack pattern set
- Boss enters from top, then moves left-right across screen
- Attack cycle: telegraph phase (core glows white) → attack execution → delay
- Each boss has 3 distinct attack patterns cycled in sequence (see `BOSS_PATTERNS` lines 1372–1409)
- Attack patterns include: radial bursts, aimed spreads, columns, spirals, crosses, homing volleys, double rings

### Difficulty System (lines 485–489)

Three presets affecting:
- Enemy spawn rate
- Enemy movement speed
- Enemy shoot chance
- Starting lives and bombs
- Boss HP multiplier

Difficulty persists in localStorage and can be changed from the menu only when not actively playing.

### Scoring & Multiplier (lines 1087–1101)

- Base kill score varies by enemy type (Grunt: 10, Interceptor: 15, Tank: 35, Boss: 500)
- Chain counter increases on each kill
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Chain timer: 2.2 seconds without a kill resets the chain
- Boss kills worth 500 × multiplier

### Performance Adaptation (lines 473–483)

Tracks last 45 frame times and adjusts quality level:
- **Quality 0** (full): All shadows, particle counts, glow effects enabled
- **Quality 1** (medium): Reduced particles, no glow shadows
- **Quality 2** (low): Minimal particles, no effects

Adjusts based on average FPS target (40–56 FPS range).

### Visual Themes (lines 493–500)

6 rotating themes change with each stage:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- Each has sky gradient colors, cloud color, and sun color
- Determines visual atmosphere as stages progress

### Local Storage Keys

- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion toggle state

## Key Functions

| Function | Purpose |
|----------|---------|
| `update(dt)` | Main game update loop (input, movement, collisions, state changes) |
| `draw()` | Render all game entities to canvas |
| `fireWeapon()` | Create player bullets based on weapon type and level |
| `useBomb()` | Detonate bomb: clear enemy bullets, kill all enemies, damage boss |
| `addScore(base)` | Increment score with multiplier, update chain timer |
| `breakChain()` | Reset combo multiplier to 1x |
| `spawnEnemy()` | Create new enemy entity at random spawn position |
| `spawnBoss()` | Create boss entity for current stage |
| `maybeSpawnPowerUp(x, y)` | 6% chance life, 8% bomb, 20% weapon (weighted random) |
| `updateBoss(dt)` | Boss movement, attack telegraphing, pattern execution |
| `killBoss()` | Destroy boss, advance stage, reset spawn rates |
| `loseLife()` | Lose one life, apply invulnerability, check game over |
| `endGame()` | Display game over screen, save best score, enable replay |
| `togglePause()` | Pause/resume game, show/hide pause overlay |
| `resetGame()` | Initialize all game state for new game |

## Editing Tips

- **Game balance**: Adjust `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, `KILLS_PER_STAGE`, weapon spread patterns, spawn rates
- **Visual polish**: Modify theme colors, particle effects, animation timings, screen shake magnitudes
- **Audio**: Edit `sfx` function parameters (frequencies, durations, filter types)
- **Controls**: Adjust movement speed, firing cooldown, bomb cost/effect
- **Boss patterns**: Modify `BOSS_PATTERNS` array functions for new attack types
- **Canvas sizing**: Default 480×720 (line 753), adapts to fit screen with max 1.6× scale (line 757)

## Browser Compatibility

Requires:
- Canvas 2D API
- Web Audio API (for audio synthesis)
- LocalStorage API (for persistence)
- `requestAnimationFrame` (for smooth game loop)
- Optional: Touch Events API (for mobile controls)

All modern browsers support these APIs.
