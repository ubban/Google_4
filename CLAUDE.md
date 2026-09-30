# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game written as a single-file HTML5 application. The entire game (HTML, CSS, and JavaScript) is contained in `index.html` (~1670 lines). This is a client-side only project with no build system, package manager, or external dependencies.

## How to Run & Test

1. **Open in browser**: Simply open `index.html` in any modern web browser. The game runs immediately.
   - Local development: `open index.html` or drag into browser
   - On web: Can be hosted on any static file server

2. **Testing gameplay**:
   - **Desktop**: Arrow keys or WASD to move, Space to fire, Shift+X to use bomb, P/Esc to pause
   - **Mobile**: On-screen joystick (left) and fire/bomb buttons (right)
   - **Audio**: Click the mute button (🔊) in the top-right; Shift+M or access via settings
   - **Accessibility**: Toggle reduced motion (🎆) to disable screen shake, flashes, vignette effects

3. **Development workflow**:
   - Edit `index.html` directly
   - Refresh browser to see changes
   - Use browser DevTools console to check for errors
   - Check localStorage: `localStorage.getItem('skyfighter.bestScore')`, etc.

## High-Level Architecture

The game follows a classic arcade game structure with a fixed main loop:

### Game Loop (Frame-based)
- `loop(now)`: Main RAF loop, calculates delta time (`dt`), calls `update()` then `draw()`
- Runs at 60 FPS, with frame skipping for low-end devices
- Frame counter at 1.25x speed for animation purposes

### Core Game State
Defined at line ~865:
- **Player**: Single player aircraft object with `{x, y, w, h, hp, level, fireMode}` (weapon level controls firing pattern)
- **Entities** (arrays):
  - `bullets`: Player projectiles
  - `enemies`: Spawned enemy ships
  - `enemyBullets`: Enemy projectiles  
  - `particles`: Explosion effects and visual feedback
  - `powerUps`: Weapon upgrades, health, bombs
  - `boss`: Boss ship object (null when not active)
- **Game flags**: `score`, `lives`, `running`, `paused`, `gameOver`, `stage`, `kills`, `chain` (combo multiplier)
- **Persistent data** (localStorage):
  - `skyfighter.bestScore`: High score
  - `skyfighter.muted`: Audio mute state
  - `skyfighter.difficulty`: Last selected difficulty
  - `skyfighter.reducedMotion`: Accessibility preference

### Update Phase (`update(dt)`)
Located at line ~1148. Key responsibilities:
1. **Player movement**: Reads from keyboard (`keys`) or touch joystick (`joyVec`)
2. **Player firing**: Weapon fires based on `firing` flag; pattern depends on `player.level` (1-5)
3. **Entity updates**: Move bullets, enemies, particles; apply physics
4. **Collision detection**: Player-enemy bullet, player-enemy, enemy-powerup collisions
5. **Enemy spawning**: `spawnEnemy()` every few frames; difficulty scales speed/count
6. **Boss logic**: `updateBoss(dt)` handles boss spawning (after 18 kills per stage), patterns, and defeat
7. **Scoring**: `addScore(base)` applies chain multiplier
8. **Game state**: End game on death, advance stage on boss defeat

### Draw Phase (`draw()`)
Located at line ~1520:
1. **Background**: Animated cloud layer at `BASE_W × BASE_H` (480×720 logical pixels)
2. **Game entities**: Draw player, enemies (via `drawPlane()`), boss (via `drawBossShip()`), bullets, particles
3. **UI HUD**: Score, multiplier, weapon level, health hearts, bomb count (updated via `render*` functions)
4. **Boss bar**: Health bar shown when boss active
5. **Overlays**: Game over screen, pause screen, stage intro
6. **Screen effects**: Flash on hit, vignette pulse when critical health, screen shake displacement

### Canvas & Responsive Design
- **Logical size**: 480×720 (portrait arcade aspect ratio)
- `resize()` scales to fit window while maintaining aspect ratio, clamped to 1.6× max
- Canvas rendered at logical size, CSS scales output; all coordinates use logical units

## Key Game Systems

### Weapon System
- **Fire modes**: Defined in constants starting at line ~875. Each player level (1-5) has a different pattern:
  - `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireDoubleRing()`, `fireHomingSeeded()` etc.
- **Bullet mechanics**: Bullets have `{x, y, vx, vy, w, h}` and are recycled (pushed to array, removed when off-screen)
- **Firing rate**: Tied to game loop; adjusted for difficulty

### Enemy & Boss System
- **Enemy types**: `ENEMY_TYPES` constant (line ~502) defines grunt, interceptor, tank with different speeds/bullets/hp
- **Spawn rate**: `spawnEnemy()` every N frames; waves increase with stage
- **Bosses** (line ~508): 4 boss designs; spawned after 18 kills per stage (`KILLS_PER_STAGE`)
- **Boss patterns**: `BOSS_PATTERNS` array (line ~1372) defines bullet formations per phase; boss cycles patterns as health decreases
- **Boss defeat**: Drops 5 power-ups, advances stage, increases difficulty

### Scoring & Difficulty
- **Difficulty levels** (line ~485): Easy/Normal/Hard affect enemy speed, bullet speed, spawn rate
- **Scoring**: Base points vary by enemy type; multiplied by chain counter (`multiplier`)
- **Chain mechanic**: Increments on kill, resets on getting hit; displays via `showComboPop()`
- **Progression**: Stage = kills / 18; harder waves as stage increases

### Input Handling
- **Keyboard**: Arrow keys/WASD for movement, Space to fire, Shift+X for bomb
- **Touch**: Virtual joystick on left half of screen, fire/bomb buttons on right
- **Mouse**: Tracks cursor position for aiming (on desktop, follows mouse if not on mobile)
- **Pause**: P key or Esc, or pause button

### Audio System
- **No audio files**: All sound generated procedurally via Web Audio API
- `ensureAudio()`: Lazily initializes audio context (required for some browsers)
- `beep()` & `noiseBurst()`: Generate tones and noise
- `sfx` object (line ~624): Maps sound names to function generators (gunfire, explosions, pickups, etc.)
- `panFor(x)`: Stereo panning based on horizontal position

### Power-Ups
- Spawn from defeated enemies via `maybeSpawnPowerUp(x, y)` (line ~1058)
- Types: Weapon upgrade (increases `player.level`), health, bomb
- Collision in `update()` triggers `applyPowerUp(type)`

### Collision Detection
- `rectsOverlap(a, b)`: Simple AABB collision (line ~1069)
- Checked every frame: player vs enemy bullets, player vs enemies, enemies vs power-ups

### Visual Effects
- **Explosions**: `spawnExplosion(x, y, color, count)` creates particles with velocity
- **Screen shake**: `screenShake(mag, dur)` sets displacement applied in draw
- **Flash**: Hit detection triggers white/red flash; bomb triggers large white flash
- **Vignette**: Red pulse when health = 1 (critical state)

## Game Flow

1. **Start**: Overlay shows difficulty selection, best score, controls hint
2. **Stage intro**: `showStageBanner()` displays stage number
3. **Wave**: Enemies spawn and attack; player destroys them
4. **Boss encounter**: After 18 kills, boss appears with name and health bar
5. **Boss defeat**: Drop power-ups, stage counter increments
6. **Game over**: Triggered by health reaching 0 or no lives left; shows final score, best score, stage reached
7. **Restart**: Click start button to reset and play again

## Important Constants & Configuration

Located near top of `<script>` tag (lines 463-520):

- **STORAGE_*** keys**: localStorage keys for persistence
- **DIFFICULTIES**: Scaling factors for each difficulty (enemySpeedMult, fireSpeedMult, etc.)
- **KILLS_PER_STAGE**: 18 — enemy kills before boss appears
- **THEMES**: Color palettes for different stages
- **ENEMY_TYPES**: Speed, HP, bullet speed for each enemy class
- **BOSSES**: Boss designs (colors, sizes)

## Performance Considerations

- **Frame time tracking**: `trackPerf(dt)` monitors performance; quality variable adjusts rendering based on FPS
- **Quality levels**: 0 = full detail, 1 = medium, 2 = low (affects particle count, etc.)
- **Target 60 FPS**: Frame skip logic prevents update if FPS drops too low
- **Canvas size**: Capped at 1.6× scale; logical resolution always 480×720

## Accessibility Features

- **Screen reader support**: `announce()` function posts to live region (`#srLive`)
- **Reduced motion preference**: Respects `prefers-reduced-motion` media query; toggle button in UI
- **ARIA labels**: Button titles and live region for status
- **Safe area insets**: Positions UI elements away from notches/safe areas on mobile

## Typical Development Tasks

### Adding a new enemy type
1. Add entry to `ENEMY_TYPES` constant (line ~502)
2. Update `spawnEnemy()` to pick the type
3. `update()` already handles generic enemy logic (collision, removal)

### Changing weapon firing pattern
1. Create new firing function (e.g., `fireZigZag()`)
2. Add call in player firing code within `update()` for a specific `player.level`
3. Test at different levels (1-5)

### Adjusting difficulty
1. Modify `DIFFICULTIES` object scaling factors
2. Test by clicking difficulty buttons before starting

### Adding visual effects
1. Create particles via `spawnExplosion()` or add custom particle type
2. Update `update()` to move particles
3. Update `draw()` to render them

### Debugging
- Use `console.log()` to trace game state
- Check `localStorage` in DevTools to verify persistence
- Pause the game (P key) to freeze state and inspect visually
- Monitor performance via `trackPerf()` output in console

## Notes for Future Work

- The entire game is in one file for simplicity; larger additions may benefit from modularizing
- Web Audio API audio is procedural; no music track (could be added)
- Performance is optimized for 60 FPS on mobile; quality scaling handles lower-end devices
- Touch controls are fully implemented; desktop still uses keyboard/mouse
- Difficulty affects balance significantly; test all three difficulties when tweaking gameplay
