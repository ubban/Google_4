# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla JavaScript and HTML5 Canvas. It's a single-file, self-contained web application with no external dependencies or build tools required.

## How to Run

Simply open `index.html` in a web browser (Chrome, Firefox, Safari, or Edge all work). The game runs entirely client-side with no server requirements.

**Development:** No build step, no dev server, no npm install needed. Edit the HTML file directly and refresh the browser to see changes.

## Architecture & High-Level Design

### Game Loop & State Management

The entire game runs on a single `requestAnimationFrame` loop (line 1640-1652) that:
1. Calculates delta time (dt)
2. Calls `update(dt)` to simulate game logic
3. Calls `draw()` to render to canvas
4. Handles performance adaptive quality scaling (quality variable: 0=full, 1=medium, 2=low)

Core game state is tracked with global variables (lines 864-869):
- `player`: position, weapon, health, invulnerability frames
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: game objects
- `score`, `lives`, `stage`, `chain`, `multiplier`: progression tracking

### Canvas & Rendering

- **Base resolution**: 480×720 (landscape mobile optimized)
- **Canvas sizing**: Responsive with max 1.6x scale (line 754-764)
- **Rendering flow** (line 1520):
  1. Clear canvas
  2. Apply screen shake offset if active
  3. Draw sky gradient with theme colors
  4. Draw parallax clouds (2 layers)
  5. Draw player plane, enemies, bosses, bullets, effects

### Input System

Three input methods coexist without conflict:
- **Keyboard**: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
- **Mouse**: Move cursor to steer, click to fire (only when cursor is over canvas)
- **Touch**: Left 55% of screen = joystick movement, right buttons = fire/bomb

Keyboard and joystick take priority over mouse steering (line 1161-1169).

### Difficulty & Game Progression

Three difficulty levels (line 485-489) scale:
- Enemy spawn rate
- Enemy movement speed & shoot frequency
- Player lives (3-4) and bomb count (2-3)
- Boss health multipliers

**Stage progression**: Each stage requires 18 kills (KILLS_PER_STAGE, line 491) before boss spawns. After defeating a boss, stage counter increments and visual theme cycles through 6 themes.

### Weapon System & Power-ups

**Weapons**: Vulcan (spread shot) and Laser (piercing beam)

**Weapon levels**: Each type has 3 levels affecting spread/pierce. Level progression (line 1311-1320):
- Levels 1-2: Increase spread/pierce
- Level 3: Switch to alternate weapon, reset to level 1

**Power-up drops** (line 1058-1067): On enemy death, 6% chance for life, 8% for bomb, 20% for weapon power-up

### Boss System

**Boss introduction** (line 1043-1056):
- Spawns after KILLS_PER_STAGE kills
- 12 second warning delay with stage banner
- Boss health scales with difficulty and unique per-boss multiplier

**Boss attack patterns** (line 1372-1409):
Each of 6 bosses has 3 unique attack patterns executed in sequence:
1. VANGUARD: radial burst, aimed fan, wide V-fan
2. BEHEMOTH: heavy columns, slow homing, dense radial
3. SPECTER: teleport dash, spiral, double aimed
4. DREADNOUGHT: 8-way cross, homing missiles, dense radial
5. WRAITH: teleport barrage, double rings, cross-spread
6. TITAN: heavy columns, full ring, relentless homing

Pattern switching occurs during telegraph phase (20-22 frames) with audio/visual cue.

### Audio System

**Web Audio Synth** (no external audio files):
- Uses oscillators (sine, triangle, square, sawtooth) with envelope shaping
- Noise bursts generated via buffer with random data
- Supports filtering (lowpass/highpass) and stereo panning based on screen position
- Master gain/compressor for dynamic range control

Key SFX functions (line 624-676): shoot, laser, explosions (grunt/interceptor/tank), hit, bomb, powerup, boss telegraph/hit/defeat, stage clear, UI click, gameover

### Accessibility Features

- **Screen reader support**: Announcements via aria-live region (line 468-471)
- **Reduced motion support**: Respects prefers-reduced-motion media query, disables animations/shake/flash (line 520-540, 349-358)
- **Keyboard navigation**: All controls available via keyboard
- **Color contrast**: High contrast UI with text shadows for readability

### Performance Optimization

**Adaptive quality** (line 474-483):
- Tracks frame times over 45 frames
- Reduces quality (particle counts, shadow blur) when avg frame time > 25ms
- Increases quality when avg frame time < 17.9ms

**Render optimizations**:
- Only spawn/update active game objects
- Particle counts scale with quality setting
- Canvas shadow/blur disabled at lower quality levels

## Common Development Tasks

### Adding a New Enemy Type

Define in `ENEMY_TYPES` (line 502-506):
```javascript
newType: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#...',
           dark: '#...', score: 25, minStage: 3 }
```

Add explosion sound variant in `EXPLODE_SFX` (line 678) and `sfx` object (line 624+).

Update `spawnEnemy()` (line 1022-1040) to include the new type in random selection.

### Adding a New Boss Pattern

1. Add pattern function to `BOSS_PATTERNS` array (line 1372-1409)
   - Use existing pattern helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
2. Boss design colors defined in `BOSSES` array (line 508-515)
3. Boss will automatically cycle through its 3 patterns in sequence

### Modifying Game Difficulty

Edit `DIFFICULTIES` object (line 485-489). Changes affect:
- `spawnBase`/`spawnMin`: Enemy spawn frequency (frames)
- `enemySpeed`: Enemy movement multiplier
- `enemyShootChance`: Probability enemy will fire
- `lives`/`bombs`: Starting resources
- `bossHpMul`: Difficulty multiplier for boss health

### Changing Visual Themes

`THEMES` array (line 493-500) defines sky gradients and cloud colors. Each stage cycles through available themes. Modify colors or add new themes here.

## Key Code Locations

| Feature | Location |
|---------|----------|
| Game loop | 1640-1652 |
| Game state & player | 864-911 |
| Input handling | 768-861 |
| Update logic | 1148-1302 |
| Collision detection | 1069-1294 |
| Draw/render | 1520-1637 |
| Boss AI | 1411-1439 |
| Audio synthesis | 542-676 |
| UI state/overlay | 429-467, 1469-1504 |

## Performance Notes

- The adaptive quality system (quality 0/1/2) scales particle counts and visual effects
- Frame time tracking helps identify bottlenecks
- Canvas size is fixed at 480×720 and scaled via CSS for responsive display
- No external image/audio assets, everything is procedurally generated

## Storage Keys

Game state persisted to localStorage:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (0/1)

## Browser Compatibility

Requires: ES6 JavaScript, Web Audio API, HTML5 Canvas, touch events
Tested on: Chrome, Firefox, Safari, Edge (desktop and mobile)
