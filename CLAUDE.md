# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 game — a Raiden-style vertical shoot-'em-up arcade game. The entire codebase is contained in `index.html` with embedded CSS styling and vanilla JavaScript game logic, no external dependencies.

### Quick Start

- **Open the game**: Simply open `index.html` in a web browser
- **No build step required** — the game runs directly
- **Supported on desktop and mobile** — uses WebAudio API for sound, Canvas 2D for graphics, and touch-enabled controls

## Architecture & Code Structure

The `index.html` file is organized into distinct logical sections (marked with comments):

### 1. **HTML & DOM Elements** (lines 362-419)
- Main canvas (`#gameCanvas`) for game rendering
- HUD elements: score, lives (hearts), bombs, weapon info, boss health bar
- Overlay for menu/pause/game-over screens
- Touch control zones (for mobile devices)
- Accessibility elements (screen-reader live region)

### 2. **CSS Styling** (lines 10-358)
- **Theme system**: Dark space theme with CSS custom properties
- **Responsive layout**: Uses `clamp()` and viewport units for mobile/desktop scaling
- **Canvas sizing**: Base 480×720px, scales up to 1.6x on larger screens
- **Animations**: Pulse effects, flash overlays, combo popups, stage banners
- **Accessibility**: Screen reader only text, reduced-motion preferences respected

### 3. **Game Logic Sections** (lines 422+)

#### **Performance Tracking** (lines 473-483)
- Adaptive quality system (3 levels) that degrades rendering if frame time drops
- Tracks last 45 frames to decide quality dynamically

#### **Configuration Objects** (lines 485-515)
- **DIFFICULTIES**: Three difficulty modes (easy/normal/hard) with spawn rates, enemy speed, HP multipliers
- **THEMES**: 6 theme variations for sky gradients, clouds, sun colors
- **ENEMY_TYPES**: Grunt, Interceptor, Tank with properties (HP, speed, score)
- **BOSSES**: 6 boss types with unique attack patterns and visual designs
- **BOSS_PATTERNS**: Attack pattern functions for each boss (radial bursts, aimed spreads, etc.)

#### **Audio System** (lines 542-676)
- **WebAudio API synth**: No external audio files; all sounds generated procedurally
- **Sound effects**: 12 SFX (shoot, laser, explosions, hits, bombs, powerups, boss sounds)
- **Mute toggle** with localStorage persistence
- **Spatial panning**: Sounds pan left/right based on screen position

#### **Input Handling** (lines 767-861)
- **Keyboard**: Arrow keys / WASD for movement, Space/mouse for fire, X/Shift for bombs, P/Esc to pause
- **Mouse**: Move to steer (crosshair targeting), click to fire
- **Touch**: Left side drag for joystick movement, right buttons for FIRE/BOMB
- **Mobile detection**: Automatically enables touch UI if device supports touch

#### **Game State** (lines 863-911)
- Core variables: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Scoring: `chain` counter (resets if no kills for 2.2s), `multiplier` increases with chain
- `resetGame()`: Initializes state for new game

#### **Game Loop & Update** (lines 1148-1302)
- **`update(dt)`**: Processes all game physics and logic each frame
  - Player movement (keyboard/joystick priority over mouse)
  - Weapon firing with cooldown
  - Bullet/enemy movement
  - Collision detection (bullets vs enemies, bullets vs boss, enemies vs player, etc.)
  - Chain/multiplier logic
  - Particle updates
  - Difficulty scaling (spawn rate increases over time)

#### **Rendering** (lines 1520-1638)
- **`draw()`**: Renders everything
  - Screen shake effect applied via canvas translation
  - Sky gradient and themed clouds (parallax scrolling)
  - Player plane, enemies, boss, bullets, particles, powerups
  - HUD rendering handled by DOM (not canvas)
- **`drawPlane()`**: Renders player and enemy planes
- **`drawBossShip()`**: Renders bosses with animated cores and wings
- **`drawClouds()`**: Parallax cloud layers

#### **Game Flow Functions**
- **Enemies**: `spawnEnemy()` uses difficulty-based spawn rates; three types with different behaviors
- **Bosses**: `spawnBoss()` triggered after 18 kills per stage; uses stage-based designs
- **Power-ups**: `maybeSpawnPowerUp()` (6% life, 8% bomb, 20% weapon drops)
- **Damage & Deaths**: `addScore()`, `loseLife()`, `killBoss()`, `endGame()`
- **Pause/Resume**: `togglePause()` shows overlay

#### **Boss Attack Patterns** (lines 1372-1409)
Each boss has 3 unique attack patterns in rotation:
- **VANGUARD**: Radial bursts, aimed fan, V-fan spreads
- **BEHEMOTH**: Heavy columns, homing spreads, dense radial walls
- **SPECTER**: Teleport dash, spiral burst, dual aimed shots
- **DREADNOUGHT**: 8-way cross fire, homing missiles, radial walls
- **WRAITH**: Teleport barrage, expanding double rings, aimed cross-spread
- **TITAN**: Heavy columns, full rings, relentless homing volleys

## Key Game Mechanics

### Progression
- Game progresses through **stages** (6 themes cycle); each stage requires 18 enemy kills to trigger boss
- **Difficulty ramp**: Enemy spawn rate accelerates over time (every 300 frames → +4 base spawn time)
- Bosses get harder and new enemy types unlock at later stages

### Scoring & Chain System
- Kills extend a **chain** (resets after 2.2s of no kills)
- **Multiplier** increases with chain: +1 every 5 kills (max 10x)
- Boss kill: 500 base points (× multiplier)
- Enemy points vary: Grunt 10, Interceptor 15, Tank 35

### Weapons
- **Vulcan Gun** (default): Spread increases with level (single → 2-wide → 3-wide)
- **Laser**: Piercing shots, faster fire rate, fewer spreads but all aimed straight
- Weapon cycle: Level 1→2→3 (same weapon), then switch to laser at level 1
- Power-up frequency: 20% of enemy drops are weapons

### Lives & Bombs
- **Difficulty-dependent**: Easy (4 lives, 3 bombs), Normal (3 lives, 2 bombs), Hard (3 lives, 2 bombs)
- **Life drops**: 6% of enemy drops
- **Bomb drops**: 8% of enemy drops
- **Bomb effect**: Clears all enemy bullets, destroys all on-screen enemies, damages boss heavily (24 HP)

### Invulnerability & Damage
- Hit: 1 life lost, 1.5s invulnerability, chain broken, screen shake & flash
- Critical state: 1 life remaining triggers vignette pulse and critical audio
- Boss collision = hit (unless boss is entering)

## Development Notes

### Adding New Content

**New Enemy Type**:
```javascript
ENEMY_TYPES.newType = { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#color', dark: '#darkColor', score: 20, minStage: 3 };
// Then add to spawnEnemy() selection logic
```

**New Boss**:
- Add to `BOSSES` array with `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
- Add attack patterns to `BOSS_PATTERNS` array (same index)

**New Theme**:
- Add to `THEMES` array with `sky` (3-stop gradient), `cloud`, `sun` colors, `name`

### Performance Considerations

- **Quality system** automatically downgrades glows/shadows if FPS drops
- **Particle counts** scale with quality level (full → 60% → 35% particles)
- **Canvas size**: Fixed 480×720 internal, CSS-scaled for display (avoids expensive redraws)
- **Collision detection**: Simple AABB (axis-aligned bounding box) using `rectsOverlap()`

### Accessibility Features

- **Reduced motion**: Respects `prefers-reduced-motion` media query; can be toggled in-game
- **Screen reader announcements**: All important state changes announced via `srLive` (kills, lives remaining, boss defeated, etc.)
- **Visual feedback**: Hearts for lives, bomb count, combo popups, score multiplier
- **Touch support**: Full mobile-friendly controls with visual buttons

### Browser Storage

Three items persisted in `localStorage`:
- `skyfighter.bestScore`: Best score
- `skyfighter.muted`: Mute state (1/0)
- `skyfighter.difficulty`: Last selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (1/0)

## Debugging & Testing

### Quick Tests

1. **Start the game**: Open in browser, click "Start Game"
2. **Test a weapon**: Grab a power-up from an enemy drop
3. **Test bombs**: Press X or Shift (keyboard) or tap BOMB button (mobile)
4. **Test pause**: Press P or click pause button
5. **Test sounds**: Disable mute, play; re-enable to verify
6. **Test on mobile**: Use browser dev tools device emulation or real device

### Common Modifications

- **Tweak difficulty**: Edit `DIFFICULTIES` object spawn rates or multipliers
- **Change theme colors**: Edit `THEMES` array
- **Adjust player speed**: Edit `player.speed` in `resetGame()`
- **Modify boss HP**: Edit base `70 + stage * 35` formula in `spawnBoss()`
- **Change starting lives**: Edit `d.lives` in `resetGame()`

## File & Line Reference Guide

| Feature | Lines |
|---------|-------|
| Canvas setup & resize | 752-765 |
| Input (keyboard + mouse) | 768-861 |
| Game state initialization | 863-911 |
| Weapon firing & cooldown | 1124-1146 |
| Enemy spawning | 1022-1041 |
| Boss spawning | 1043-1056 |
| Main update loop | 1148-1302 |
| Collision detection | 1232-1287 |
| Boss AI & patterns | 1372-1439 |
| Drawing functions | 1507-1638 |
| Game loop (requestAnimationFrame) | 1640-1665 |
