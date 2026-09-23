# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a vertical shoot-'em-up game (Raiden-style) implemented as a single HTML5 file with embedded JavaScript. No build process is needed—the game runs directly in any modern web browser.

### Quick Start

- **Run**: Open `index.html` directly in a web browser or serve it via HTTP
- **Develop**: Edit `index.html` directly; changes are live on refresh (with server reload)
- **Deploy**: Upload `index.html` to a web server

## Architecture & Code Organization

The game is organized as a single-file application (1669 lines) within the `<script>` block of `index.html`, starting at line 422. Below is the high-level structure:

### 1. **Core Systems** (State & Setup)
- **Storage Keys** (lines ~463–466): Best score, mute state, difficulty, reduced motion preference
- **Adaptive Performance** (lines ~473–483): Quality level (0=full, 1=medium, 2=low) that dynamically adjusts based on frame timing
- **Game Configuration**:
  - `DIFFICULTIES` (lines ~485–489): Easy, Normal, Hard presets with spawn rates, enemy speed, lives, bombs
  - `THEMES` (lines ~493–500): 6 visual themes with sky gradients and cloud/sun colors
  - `ENEMY_TYPES` (lines ~502–506): grunt, interceptor, tank with stats (HP, speed, score, appearance)
  - `BOSSES` (lines ~508–515): 6 boss types with unique color schemes and HP multipliers
  - `KILLS_PER_STAGE` (line ~491): 18 enemies defeat to advance stage

### 2. **Game State Variables** (Initial setup around lines ~517–527)
These are declared in module scope, not passed as objects. Key ones include:
- `bestScore`, `muted`, `difficulty`, `reducedMotion` — player preferences
- `quality` — adaptive performance level
- `player`, `enemies`, `bullets`, `explosions`, `effects`, `powerUps`, `combo` — active entities
- `gameState` — "menu" | "playing" | "paused" | "gameOver"
- `currentStage`, `stageKills` — progression tracking

### 3. **Audio System** (lines ~542–610)
Uses **Web Audio API** with synthesized sound (no external files):
- `ensureAudio()` — lazily initialize AudioContext on first user interaction
- `beep(freq, dur, type, vol, opts)` — generate tones with optional frequency sweep, filter, pan, delay
- `noiseBurst(dur, vol, opts)` — white noise for explosions
- All sounds auto-mute if `muted` is true

### 4. **Input Handling** (lines ~785–839 for touch joystick, keyboard in event listeners)
- **Touch Controls**: Virtual joystick with `joystickStart()`, `joystickMove()`, `joystickEnd()`
- **Keyboard**: WASD or arrow keys for movement, Space/Z for fire, X for bomb
- **Canvas Points**: `canvasPointFromEvent()` converts event coordinates to canvas space, handling DPI scaling

### 5. **Rendering** (Immediate mode, no retained state)
- **HUD** (lines ~680–725): Hearts, bombs, score, weapon info, combo pop, stage banner
- **Plane** (lines ~913–969): `drawPlane()` with animated flame trails based on power-up level
- **Boss Ship** (lines ~970–1020): `drawBossShip()` with rotating wings, core glow
- **Clouds** (lines ~1507–1519): Parallax scrolling background
- **Main Render** (lines ~1520–1639): `draw()` clears canvas, draws all entities

### 6. **Entity Management**
- **Spawning**: `spawnEnemy()` (line ~1022), `spawnBoss()` (line ~1043), `maybeSpawnPowerUp()` (line ~1058)
- **Movement & Collision** (`update()`, lines ~1148–1303):
  - Bullets move, enemies move (downward, may weave), destructible blocks move
  - `rectsOverlap()` (line ~1069) — axis-aligned AABB for all collisions
  - Player-bullet, enemy-bullet, bomb-enemy, power-up pickup all checked here
- **Effects**: `spawnExplosion()` (line ~1073) for particle clouds, screen shake on hit

### 7. **Weapon System** (lines ~1124–1400)
- `fireWeapon()` (line ~1124) — player fires based on current `weaponType`
- `applyPowerUp(type)` (line ~1304) — changes weapon; levels 0–3 are standard (bullet spreads), higher custom patterns
- Boss-specific firing functions:
  - `fireRadial()` — even arc around boss
  - `fireAimedSpread()` — cone toward player
  - `fireColumns()` — vertical columns
  - `fireSpiralBurst()` — rotating spiral
  - `fireHomingSeeded()` — homing bullets with slight tracking
  - `fireDoubleRing()` — two concentric rings at different speeds
- Bombs: `useBomb()` (line ~1103) — clears all bullets & enemies in radius, grants score

### 8. **Game Loop** (lines ~1640–end)
- `loop(now)` — called via `requestAnimationFrame()`
- Measures delta time (`dt`), calls `trackPerf()`, `update()`, `draw()`
- Calls `loop()` recursively

### 9. **Game States & Transitions**
- **Menu**: Difficulty selection, start button
- **Playing**: Active gameplay, pause available
- **Paused**: `togglePause()` (line ~1491)
- **GameOver**: Display final score, best score, stage reached

## Key Design Patterns

1. **No Classes / OOP**: Uses objects with properties (e.g., `player = { x, y, w, h, ... }`); enemies/bullets/effects stored in arrays
2. **Immediate-Mode Rendering**: Every frame, canvas is cleared and all entities drawn fresh (no scene graph)
3. **Delta-Time Physics**: `update(dt)` moves entities by `velocity * dt`, accounting for variable frame rate
4. **Collision Detection**: Simple AABB overlap, checked every frame in `update()`
5. **Difficulty Scaling**: Enemy spawn rate, speed, and fire rate increase based on `difficulty` setting
6. **Progressive Difficulty**: Wave of 18 kills per stage, boss encounter every few stages, stage count affects enemy health/speed
7. **Adaptive Performance**: `trackPerf()` monitors avg frame time; if frame time > 1/40s, reduce quality level (fewer particles, simpler effects)

## How to Modify Common Areas

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line ~502), set `w, h, hp, speedMul, color, dark, score, minStage`
2. In `update()`, add logic to `spawnEnemy()` to pick the new type randomly or by stage
3. Draw logic is generic (`drawPlane()`), but can be customized if needed

### Add a New Weapon/Power-up
1. Increment weapon level in `player.weaponLvl`
2. In `applyPowerUp()` (line ~1304), add case for new level
3. In `fireWeapon()` or via boss fire functions, add bullet spawn logic
4. Update `weaponLabel()` (line ~871) to display weapon name

### Adjust Difficulty
Edit `DIFFICULTIES` object (line ~485). Parameters:
- `spawnBase` — frames between enemy waves
- `spawnMin` — minimum spawn interval at high stress
- `enemySpeed` — speed multiplier
- `enemyShootChance` — per-frame probability of firing
- `lives` — starting player lives
- `bombs` — starting bombs
- `bossHpMul` — multiplier on boss health

### Add Visual Theme
Add object to `THEMES` array (line ~493):
```javascript
{ sky: ['#color1', '#color2', '#color3'], cloud: 'r,g,b', sun: 'r,g,b', name: 'Theme Name' }
```

### Adjust Audio
- Modify `beep()` and `noiseBurst()` calls throughout code to change pitch, duration, or volume
- Audio is synth-based; no file loading needed

## Performance & Optimization

- **Adaptive Quality**: `quality` variable (0–2) affects particle count, flash intensity, and cloud density
- **Frame Time Tracking**: `frameTimes[]` stores last 45 frame durations; averages every 45 frames
- **No Image Assets**: All graphics drawn via Canvas API; all audio synthesized—minimal memory/bandwidth
- **LocalStorage**: Best score, mute, difficulty persisted locally

## Browser & Platform Support

- Modern browsers with **Canvas 2D** and **Web Audio API**
- **Touch**: Full virtual joystick support for mobile/tablet
- **Keyboard**: WASD + Space/Z/X for desktop
- **Reduced Motion**: Respects `prefers-reduced-motion` media query; also toggleable via button

## Testing & Debugging

- Open browser DevTools (F12) and check `console` for errors
- `muted` variable controls audio on/off; set `localStorage.skyfighter.muted = '1'` to mute
- `difficulty` variable can be changed mid-play via `setDifficulty('easy'/'normal'/'hard')`
- Frame rate visible in HUD (if FPS counter is enabled); adaptive quality changes when frame time drops
- Screen shake and flash can be toggled via reduced-motion button

## File Structure

```
Google_4/
├── index.html          (entire game: HTML + CSS + JavaScript)
├── README.md           (basic project info)
└── CLAUDE.md           (this file)
```

No build step, no external dependencies, no separate source files.
