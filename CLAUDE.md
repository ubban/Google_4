# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. It features:
- Player-controlled aircraft with weapon upgrades
- Multiple enemy types with varying behaviors
- Boss encounters with complex attack patterns
- Scoring system with combo multipliers and difficulty settings
- Responsive design supporting desktop (keyboard/mouse) and mobile (touch controls)
- Procedurally-generated audio using Web Audio API

## Architecture

### Single-File Structure
The entire game is implemented in `index.html` (1,669 lines) with three integrated sections:
1. **CSS** (lines 10–359): Inline styles for all UI, HUD, and animations
2. **HTML** (lines 360–420): Minimal DOM structure with canvas and overlay elements
3. **JavaScript** (lines 422–1669): Complete game logic and rendering

### Core Systems

#### Game State (lines 863–869)
Global variables track:
- **Player**: Position, weapon (vulcan/laser), level, invuln frames
- **Entities**: `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Progression**: `score`, `lives`, `stage`, `kills`, `stageKills`, `multiplier`
- **UI**: `paused`, `gameOver`, `running`

#### Main Loop (lines 1640–1669)
- `loop(now)`: Called via `requestAnimationFrame`
  - Calculates delta time (capped at 50ms)
  - Calls `update(dt)` if running and not paused
  - Calls `draw()` each frame
  - Tracks performance for adaptive quality (lines 473–483)

#### Update Logic (lines 1070–1302)
Player movement and input drive the loop:
- **Player input** (lines 1070–1100): Keyboard (arrows/WASD), mouse targeting, joystick (touch), firing/bomb controls
- **Enemy spawning** (lines 1130–1206): Time-based with progressive difficulty ramp per stage
- **Collision detection** (lines 1232–1294): Player bullets vs enemies, player bullets vs boss, enemyBullets vs player, boss vs player, pickups
- **Boss battles** (lines 1230, 1255–1266): If boss exists, apply damage and check defeat

#### Rendering (lines 1507–1638)
- `draw()`: Renders each frame with canvas transforms for screen shake
- **Theme system** (lines 493–500, 1529–1544): Six themed skies with gradient backgrounds and parallax clouds
- **Drawing functions**: `drawPlane()`, `drawBossShip()`, `drawClouds()`
- **Adaptive rendering**: Reduces shadow/glow on low-end devices via `quality` variable

### Enemy & Boss Systems

#### Enemy Types (lines 502–506)
Defined in `ENEMY_TYPES` with properties: width, height, HP, speed multiplier, color, score, minimum stage
- **Grunt** (stage 1+): Slow, low HP, shoots straight
- **Interceptor** (stage 1+): Fast, weaving pattern, shoots straight
- **Tank** (stage 2+): Slow, high HP, displays health bar

#### Boss Fights (lines 508–515, 1372–1439)
- **Six bosses** (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) appear one per stage
- **Pattern-based attacks** (BOSS_PATTERNS, lines 1372–1409):
  - Each boss has 3 attack patterns that rotate
  - Patterns include: radial bursts, aimed spreads, columns, spirals, cross fire, homing missiles, teleport+attack
- **Telegraph system** (lines 1421–1434): Boss flashes white 22 frames before attacking

#### Power-Ups (lines 1304–1322)
- **Life** (red square): Restore a life
- **Bomb** (yellow square): Restore a bomb charge
- **Weapon** (green square): Upgrade weapon level or toggle between vulcan/laser

### Audio System (lines 542–676)

Uses **Web Audio API with no external files**—all sound is synthesized:
- **Oscillators** (square, sine, sawtooth waves) for melodic sounds
- **Noise bursts** (random buffer + filters) for explosions
- **Gain envelopes** for attack/release dynamics
- **Filters** (lowpass/highpass) for tone shaping
- **Panning** based on world position (left/right stereo)

Key SFX functions: `beep()`, `noiseBurst()`. Individual effects are in the `sfx` object (lines 624–676).

### Input Systems (lines 767–862)

#### Keyboard
- Arrow keys / WASD: Move
- Space: Fire
- X / Shift: Drop bomb
- P / Esc: Pause

#### Mouse
- Move over canvas: Steer toward cursor
- Click: Fire
- Keyboard/joystick input overrides mouse to prevent fighting

#### Touch (Mobile)
- Left 55% (moveZone): Joystick movement with visual feedback
- Right 45% (actionZone): Fire and bomb buttons
- Joystick has max distance (50px) with stick visual

### UI & HUD (lines 34–358)

#### On-Screen Display
- **Top-center HUD**: Score, best score, weapon, hearts, bomb indicators, multiplier
- **Top-right buttons**: Mute (🔊), reduced motion toggle (🎆), pause (⏸)
- **Overlay**: Menu (start game, difficulty selection), pause screen, game-over screen
- **Boss bar**: Appears when boss is active, shows remaining HP

#### Persistent State (localStorage, lines 517–527)
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute toggle
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced-motion preference (respects system `prefers-reduced-motion`)

### Difficulty & Progression

#### Difficulties (lines 485–489)
Each difficulty changes spawn rates, enemy behavior, lives, bombs, and boss HP multiplier.

#### Stage Progression
- **Goal**: Kill 18 enemies (`KILLS_PER_STAGE = 18`)
- **Boss spawn**: 1.2s after 18th kill
- **Advance**: Boss defeated → stage++, theme changes, kill counter resets
- **Theme cycling**: Six themes repeat via `(stage - 1) % THEMES.length`

#### Combo System (lines 896–898, 1248, 1339–1350)
- **Multiplier**: Increases from kills (every 10th kill shows combo pop-up)
- **Score boost**: Each kill awards `score * multiplier`
- **Reset**: Losing a life breaks the chain (multiplier → 1)

## Development & Debugging

### Canvas Size & Scaling
- **Base resolution**: 480×720 (portrait)
- **Scaling**: Maintains aspect ratio, max 1.6× scale (lines 753–765)
- **Responsive**: Re-sizes on window/orientation changes

### Performance Tracking (lines 473–483)
- Adaptive quality: Reduces shadow/glow at 40 FPS threshold, re-enables above 56 FPS
- `quality` variable: 0 (full), 1 (medium), 2 (low)

### Entity Management
- Enemies and projectiles maintain lists; off-screen/dead entities are filtered out
- Particle system for visual feedback (explosions, hits)

## Modification Guide

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with stats
2. Update `spawnEnemy()` to include spawn condition (minStage check)
3. Update `EXPLODE_SFX` (line 678) if using custom explosion sound

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508) with name and colors
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372)
3. Pattern functions use helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.

### Adding a New Theme
1. Add entry to `THEMES` array (line 493) with sky gradient colors, cloud color, sun color, and name
2. Themes automatically cycle per stage

### Modifying Gameplay Numbers
Key constants cluster near the top of the game state section:
- `BASE_W`, `BASE_H` (line 753): Canvas resolution
- `KILLS_PER_STAGE` (line 491): Enemies to kill before boss
- `DIFFICULTIES` (line 485): Spawn rates, enemy speed, live/bomb counts
- Weapon properties, power-up spawn rates: Embedded in `update()` and respective functions

## Common Tasks

### Run/Test
The game is immediately playable in any browser:
```bash
open index.html
# or
firefox index.html
```

### Debug Rendering
- Uncomment `drawPlane()` call (line 1558) to visualize weapon radius
- Add `console.log()` in `update()` or `draw()` to track state

### Tune Difficulty
Edit `DIFFICULTIES[difficulty]` entries (lines 485–489) or per-stage numbers in `update()` (line 1201: `difficultyRamp`).

### Audio Tuning
Modify `beep()` parameters (frequency, duration, volume) or add new SFX to the `sfx` object (line 624).

### Reduced Motion
The CSS respects `prefers-reduced-motion: reduce` and disables animations (line 349–358). Animations also reduced via `reducedMotion` variable (line 521).
