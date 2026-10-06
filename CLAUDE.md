# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built with vanilla HTML5 Canvas and Web Audio API. The entire game is contained in a single `index.html` file (~1670 lines) with inline styles and JavaScript.

## Running the Game

The game requires no build step or dependencies. To run:
- Open `index.html` directly in a modern web browser (Chrome, Firefox, Safari, Edge)
- The game is fully playable on desktop (mouse + keyboard) and mobile (touch)

No server needed — it's a pure client-side single-file application.

## Game Architecture & Key Systems

### Core Loop
The game runs on a 60 FPS `requestAnimationFrame` loop:
1. **Update Phase** (`update(dt)`): handles input, movement, collision detection, spawning
2. **Render Phase** (`draw()`): renders all entities to canvas with effects
3. **Performance Tracking** (`trackPerf(dt)`): adaptive quality system reduces particle/glow effects if FPS drops below ~40

### Game State
Located around line 863-869: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`, `score`, `lives`, `stage`, etc. All manipulated imperatively; no frameworks.

### Player & Input (lines 767-861)
- **Keyboard**: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Hover over canvas to steer (priority over keyboard when mouse is active); click to fire
- **Touch**: Left zone is joystick (drag to move), right zone has FIRE and BOMB buttons
- Mouse and keyboard input blend smoothly—keyboard always takes priority when pressed

### Weapon System (lines 871-1146)
Two weapons with level progression:
- **Vulcan** (default): Fires vertical bullets. Spreads as you level up (1→3 bullets).
- **Laser**: Fires faster, pierces enemies. Also spreads at higher levels, bouncing between lanes.
- Power-ups cycle through weapon upgrades, then swap weapon type when maxed.

### Enemy System (lines 502-1041)
**Enemy Types** (defined in `ENEMY_TYPES` object):
- **Grunt**: Basic red enemy, 1 HP, common from stage 1
- **Interceptor**: Fast orange enemy, 1 HP, appears stage 1+, weaves
- **Tank**: Slow purple heavy unit, 4 HP, stage 2+, worth 35 points

Enemies **spawn with random patterns**:
- `'spread'` pattern: fires 3 bullets in an arc
- `'aimed'` pattern: calculates trajectory toward player
- Weaving enemies (`interceptor`) use sine wave offset from their base X

Spawn rate scales with game frame count (difficulty ramp) and stage.

### Boss System (lines 508-1409)
Six unique boss designs cycle through stages (repeating). Each boss:
- Has distinct hull/wing/core colors and attack patterns
- Enters from top, moves horizontally while attacking
- Uses **telegraph animation** (~22 frames) before firing
- Has HP scaled by difficulty multiplier × boss type multiplier

**Boss Attack Patterns** (defined in `BOSS_PATTERNS`):
- Radial bursts, aimed spreads, columns, spirals, homing missiles, etc.
- Each boss pattern object contains 3 attack types cycling through a sequence
- Patterns spawn `radial` bullets that move via `dx`/`dy` (not `vx`/`speed`)

### Collision Detection (line 1069)
Simple AABB (axis-aligned bounding box) rectangle overlap:
```javascript
Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2
```
Used for: bullet-enemy, bullet-boss, enemy-player, enemyBullet-player, powerUp-player collisions.

### Scoring & Multiplier System (lines 1087-1101)
- **Chain**: Each kill extends the chain; chain breaks if you take damage or wait 2.2 seconds
- **Multiplier**: 1 + floor(chain / 5), capped at 10x
- Score increases based on enemy type × multiplier
- Boss kills award flat 500 points

### Particle System (lines 1073-1085, 1296-1297)
Particle objects spawn on explosions and damaging hits. They:
- Store position, velocity, life duration, max life, and color
- Fade out visually using `globalAlpha` scaled by remaining life ratio
- Particle count is adaptive based on quality setting

### Difficulty Modes (lines 485-489)
Three presets affecting enemy spawn rate, speed, shoot chance, boss HP:
- **Easy**: Slower spawns, weaker enemies (0.8× boss HP)
- **Normal**: Balanced (default)
- **Hard**: Aggressive spawns, tougher enemies (1.3× boss HP)

Settings persist in localStorage. Difficulty can be changed before starting or between lives.

### Performance Adaptation (lines 473-483)
Tracks the last 45 frame delta times. If average > 1/40 (below 40 FPS), quality increases (1 or 2). If average < 1/56 (above 56 FPS), quality decreases. Higher quality values:
- Reduce particle count during explosions
- Disable glow effects on projectiles and power-ups

### Canvas Sizing (lines 752-765)
Base resolution is **480×720** (portrait, phone-friendly). Canvas scales to fit window while maintaining aspect ratio, capped at 1.6× for desktop play.

### Audio System (lines 542-676)
Uses **Web Audio API with procedural synthesis**—no external audio files:
- **Oscillators** for tonal sounds (shoot, laser, boss telegraph, etc.)
- **Noise bursts** for explosions (tank, bomb)
- **Filters** (lowpass/highpass) for tone shaping
- **Panning** based on bullet X position (stereo imaging)
- **Envelope controls** for volume ramping (attack, decay)
- Mute/unmute toggle persists in localStorage

Common SFX functions: `beep()`, `noiseBurst()`, `sfx.shoot()`, `sfx.bomb()`, `sfx.explode()`, etc.

### Visual Themes (lines 493-500)
Six color themes cycle by stage:
- Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- Each defines sky gradient, cloud color, sun glow color
- Clouds scroll at different speeds to create parallax depth

### UI & HUD (lines 367-419, 680-725)
- **Top center**: Score, best score, multiplier, weapon name/level, hearts (lives), bombs
- **Top right**: Mute, reduced motion toggle, pause button
- **Boss fight**: Boss name and health bar appear when fighting
- **Combo pop**: Visual feedback at 10, 20, 30... kill milestones
- **Stage banner**: Announcement on new stage or boss arrival
- **Overlay**: Start menu, pause screen, game over screen

### Input Accessibility (lines 1491-1504)
- Pause/resume with P key or pause button
- Reduced motion toggle respects `prefers-reduced-motion` media query
- Screen reader announcements via `srLive` div (ARIA live region)

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (around line 502)
2. Set `minStage` to control when it appears
3. Update `spawnEnemy()` logic (line 1026) if needed to adjust spawn weights

### Adding a Boss Pattern
1. Create a new function (e.g., `fireNewPattern(b)`) that pushes `enemyBullets`
2. Add to a new entry in `BOSS_PATTERNS` array (line 1372)
3. Boss designs cycle by stage; pattern index maps to boss design

### Tweaking Difficulty
- Edit `DIFFICULTIES` object (line 485) to adjust spawn rates, enemy speeds, lives, bomb count, boss HP multiplier
- Changes take effect on next game start

### Adjusting Weapon Balance
- **Vulcan**: Modify spread angles and bullet speeds in `fireWeapon()` (line 1124)
- **Laser**: Adjust lane offsets, pierce values, or fire rate (`fireDelay` at line 1177)
- **Power-up rarity**: Edit thresholds in `maybeSpawnPowerUp()` (line 1058)

### Visual & Audio Changes
- **Sky/cloud colors**: Edit `THEMES` array (line 493)
- **Enemy colors**: Edit `ENEMY_TYPES` and boss designs in `BOSSES` (line 508)
- **Screen shake intensity**: Modify calls to `screenShake()` throughout update/event handlers
- **Sound design**: Edit SFX functions in `sfx` object (line 624)—tweak frequencies, durations, sweep targets

### Testing Game Balance
The game stores high score and settings in localStorage. To reset:
```javascript
localStorage.clear()
```
Then reload. Useful for testing difficulty progression without playing through full games.

## Key Files & Responsibilities

| Section | Lines | Purpose |
|---------|-------|---------|
| HTML Structure & Styles | 1–359 | DOM, CSS grid/flex layouts, animations |
| Canvas & Input Setup | 752–861 | Canvas sizing, keyboard/mouse/touch handlers |
| Game State | 863–910 | Core state variables, reset logic |
| Update Loop | 1148–1302 | Movement, collision, spawning, particle updates |
| Rendering | 1520–1638 | Canvas drawing, layering, effects |
| Main Loop | 1640–1666 | RAF loop, frame timing, event delegation |

## Design Notes

- **Single-file architecture**: Easier to deploy and version as one asset; all game logic in one scope.
- **Procedural audio**: No asset loading; reduces HTTP requests and file size.
- **Adaptive rendering**: Quality setting maintains playable framerates on lower-end devices.
- **Input priority**: Keyboard > joystick > mouse, so each input method feels responsive without conflict.
- **Boss telegraph**: Visual & audio cues before attacks give player reaction time; key to difficulty perception.
