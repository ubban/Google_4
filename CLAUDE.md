# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up (shmup) game built as a single-file HTML5 Canvas application. It features multiple difficulty levels, stage-based progression, boss battles, weapon upgrades, and synthesized audio using the Web Audio API.

## How to Run

1. **Development**: Open `index.html` directly in a web browser. No build step, no dependencies.
2. **Live server** (optional): Use any local HTTP server: `python -m http.server 8000` or `npx http-server`

The game runs entirely client-side in a single HTML file (~1600 lines).

## Architecture Overview

The entire game logic is contained in an IIFE (Immediately Invoked Function Expression) starting at line 423. The structure follows a classic game loop pattern:

### Core Systems

**Game State** (lines 863-869)
- `player`: Position, speed, weapons, invulnerability frames
- `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles`: Game entities
- `boss`: Current boss state during stage bosses
- Game flags: `running`, `paused`, `gameOver`, `stage`, `score`, `multiplier` (combo chain)

**Input Handling** (lines 768-861)
- Keyboard: Arrow keys / WASD + Space (fire) + X/Shift (bomb) + P/Esc (pause)
- Mouse: Move cursor to steer; click to fire (continuous while held)
- Touch: Virtual joystick on left side; FIRE/BOMB buttons on right
- Priority: Keyboard/joystick input overrides mouse steering

**Rendering Pipeline** (lines 1520-1638)
- Canvas is 480x720px (scaled to fit viewport, max 1.6x)
- Draws: Sky gradient, clouds, player, enemies, bullets, boss, particles, UI HUD
- `shakeTime` and `shakeMag` handle camera shake on hits/bombs
- Screen flash effects (`#flash`) for impact feedback

**Game Loop** (lines 1640-1652)
- `requestAnimationFrame(loop)` drives 60fps updates
- Clamps dt to 0.05 max per frame
- Calls `update(dt)` then `draw()` when `running && !paused`
- Adaptive quality system (lines 474-483) reduces particles/glow effects if FPS drops

### Key Game Systems

**Difficulty Tuning** (line 485-489)
- Three presets: Easy / Normal (default) / Hard
- Controls: spawn rates, enemy speed, shoot frequency, lives, bomb count, boss HP multiplier
- Difficulty is persistent in localStorage

**Combat**
- **Player weapons** (lines 1124-1146): Vulcan (spread shots) or Laser (piercing shots with wider lanes at higher levels)
- **Enemy types** (lines 502-506): Grunt, Interceptor, Tank — each with different HP, speed, score
- **Boss patterns** (lines 1372-1409): 6 unique boss designs with 3 attack patterns each (radial, aimed spread, cross, homing, teleport, etc.)
- **Damage model**: Bullets deal 1 HP to enemies, 3 HP to bosses. Tanks have 4 HP. Bomb deals 24 HP to bosses, destroys all on-screen enemies.

**Progression**
- 18 kills per stage triggers boss warning → boss spawns after 1.2s delay
- Defeating boss advances stage, resets kill counter, changes theme
- 6 themes cycle: Daylight → Sunset → Night → Storm → Arctic → Volcanic
- 6 bosses cycle with stage-based difficulty scaling

**Scoring & Combo System** (lines 1087-1101)
- Base score per enemy type (Grunt: 10, Interceptor: 15, Tank: 35, Boss: 500)
- Combo multiplier increases by 1 for every 5 consecutive kills (max 10x)
- Combo chain resets if player takes damage or 2.2s passes without a kill
- Bombs always award 5 points per destroyed enemy

**Power-ups** (lines 1058-1067, 1304-1322)
- Life (6% spawn chance): +1 hp (max to starting lives)
- Bomb (8% spawn chance): +1 bomb (max to difficulty max)
- Weapon (20% spawn chance): Upgrade weapon level (1-3) or switch Vulcan ↔ Laser at level 3

**Audio** (lines 542-676)
- WebAudio synth: All sounds generated procedurally (beep, noise bursts, frequency sweeps)
- `beep()`: Oscillator-based tones with optional filter/panning
- `noiseBurst()`: Generated white noise filtered
- `sfx` object: Named sound effects for each game event (shoot, explode, bomb, hit, boss telegraph, etc.)
- Master compressor for consistent volume; panned spatially by screen position

**Collision Detection**
- Simple AABB (axis-aligned bounding box) via `rectsOverlap()` (line 1069-1071)
- Bullet-enemy, bullet-boss, enemy-player, bomb-all collisions in `update()` (lines 1232-1294)
- Collision response: Damage, score, particle spawn, sound effect

**Performance Optimization**
- Adaptive quality mode (lines 474-483): Tracks frame times; reduces particle count, disables glow/shadows if FPS < 40
- Reduced motion preference (lines 520-523): Disables animations if OS/browser prefers reduced motion
- Lazy WebAudio initialization (line 545-561): Only creates context on first input

### Data Structures

**DIFFICULTIES** (line 485)
```javascript
{ spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul }
```

**BOSS_PATTERNS** (line 1372)
- Array of 6 patterns (one per boss)
- Each pattern is an array of 3 attack functions
- Functions manipulate `enemyBullets` array

**Player Object** (line 877)
```javascript
{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
```

**Enemy Object** (line 1032)
```javascript
{ x, y, w, h, speed, type, hp, color, dark, score, pattern, weave, weaveSeed, baseX }
```

**Boss Object** (line 1047)
```javascript
{ x, y, w, h, hp, maxHp, entering, phaseTimer, attackTimer, patternIndex, vx, design, telegraph }
```

## Development Notes

### Modifying Game Balance

- **Difficulty curves**: Edit `DIFFICULTIES` constants (line 485)
- **Enemy stats**: `ENEMY_TYPES` (line 502)
- **Boss difficulty**: Adjust `hpMul` in `BOSSES` array (line 508) or base HP formula in `spawnBoss()` (line 1046)
- **Weapon power**: Edit `fireWeapon()` spread/count or bullet `speed`/`pierce`
- **Spawn rate**: Adjust `spawnBase` and `spawnMin` in difficulty settings

### Adding New Features

**New Enemy Type**: 
1. Add to `ENEMY_TYPES` with `minStage` constraint
2. Add explosion SFX key to `EXPLODE_SFX` (line 678)
3. Modify spawn logic in `spawnEnemy()` (line 1022) probability checks

**New Boss Attack Pattern**:
1. Create pattern function: `fireRadial(b, count, speed)`, `fireAimedSpread()`, etc. (lines 1324-1370 are templates)
2. Add 3-pattern array to `BOSS_PATTERNS`
3. Boss automatically uses patterns cyclically based on design index

**New Theme**:
1. Add to `THEMES` array (line 493) with sky gradient colors, cloud color, sun color
2. Themes cycle per stage: `THEMES[(stage - 1) % THEMES.length]`

### Audio Synthesis

WebAudio parameters are tuned per sound effect in the `sfx` object:
- `beep(freq, dur, type, vol, opts)`: Sine/square/sawtooth oscillator with frequency ramp
  - `opts.sweepTo`: Target frequency (exponential ramp)
  - `opts.filterFreq`: Biquad filter cutoff
  - `opts.pan`: Stereo panning (-1 to 1)
- `noiseBurst(dur, vol, opts)`: Procedurally generated white noise with envelope
- Compression (line 549) flattens dynamics; master gain at 0.9

Adjust:
- Volume levels: Change `vol` parameter or master gain
- Timbre: Modify filter type/freq or oscillator sweep targets
- Timing: Adjust `dur` and `delay` offsets for attack/release

### Persistence & Local Storage

- Best score: `STORAGE_BEST` (line 463)
- Mute state: `STORAGE_MUTE`
- Difficulty setting: `STORAGE_DIFF`
- Reduced motion preference: `STORAGE_REDUCED`

All stored as strings in `localStorage`; read on startup (lines 517-523).

### Accessibility

- Reduced motion mode: Disables animations, screen shake, flash effects (checked at line 349)
- Screen reader announcements: `announce()` function (line 468) updates `#srLive` aria-live region
- Semantic HTML: Buttons, labels, ARIA attributes for game state
- Keyboard shortcuts all clearly labeled in UI

## File Structure

```
Google_4/
├── index.html          # Single-file game (HTML + CSS + JavaScript)
├── README.md           # Basic info
└── CLAUDE.md           # This file
```

## Common Modifications Checklist

- [ ] Adjust difficulty balance → `DIFFICULTIES`
- [ ] Change enemy spawn frequency → `spawnBase` / `spawnMin`
- [ ] Modify weapon behavior → `fireWeapon()`
- [ ] Add visual polish → Canvas draw functions (`drawPlane`, `drawBossShip`)
- [ ] Tune audio → `beep()`, `noiseBurst()` parameters or master gain
- [ ] Change color scheme → `THEMES` or hardcoded `fillStyle` colors
- [ ] Adjust screen shake intensity → `screenShake()` calls and `shakeMag` multiplier
