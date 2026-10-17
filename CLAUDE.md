# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla JavaScript with Canvas rendering. The entire game is contained in a single `index.html` file (~1670 lines) with no external dependencies beyond the browser's native APIs.

## Running the Game

- **Development**: Open `index.html` directly in a web browser (no build step required)
- **File**: Single HTML file contains all styling, HTML markup, and JavaScript logic
- **Browser Support**: Modern browsers with Canvas 2D, Web Audio API, and ES6 support

## Architecture Overview

The game is structured as a single Immediately Invoked Function Expression (IIFE) containing all game logic. Key architectural layers:

### 1. **UI & Presentation** (CSS, lines 10-358)
- Responsive canvas sizing with `BASE_W = 480, BASE_H = 720`
- HUD overlay showing score, multiplier, weapon, health, bombs
- Boss health bar
- Combo/stage banners
- Pause overlay and game-over screen
- Touch control zones (left for movement, right for fire/bomb)
- Visual effects: flash overlays, vignette, screen shake

### 2. **Input Systems** (lines 767-861)
Four input methods, with keyboard/joystick taking priority over mouse:
- **Keyboard**: Arrow keys or WASD for movement, Space/Shift+X for fire/bomb, P/Esc to pause
- **Mouse**: Move to steer (glides plane toward pointer), click to fire
- **Touch**: Left zone for movement (joystick), right buttons for fire/bomb
- **Joystick**: Visual on-screen joystick for touch devices

### 3. **Core Game Loop** (lines 1640-1665)
Uses `requestAnimationFrame` at ~60 FPS:
- **Timing**: Delta-time calculation with 50ms cap per frame
- **Update Phase** (line 1648): Input processing, physics, collision detection
- **Draw Phase** (line 1650): Canvas rendering

### 4. **Game State** (lines 864-869)
Central state variables:
- `player` - Position, velocity, weapon, health, invulnerability
- `bullets` - Player projectiles
- `enemies` - Enemy ships with health, movement patterns
- `enemyBullets` - Enemy projectiles
- `boss` - Current stage boss (null when not present)
- `particles` - Explosion and visual effects
- `powerUps` - Collectibles (life, bomb, weapon)
- Game flags: `running`, `paused`, `gameOver`, `stage`, `score`, `multiplier`

### 5. **Difficulty System** (lines 485-489, 741-750)
Three presets stored in localStorage:
```javascript
DIFFICULTIES = {
  easy: { spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul },
  normal: { ... },
  hard: { ... }
}
```
Affects: enemy spawn rate, speed, fire rate, boss HP, starting lives/bombs

### 6. **Enemy & Boss Design** (lines 502-515)
- **Enemy Types**: `grunt` (weak), `interceptor` (fast), `tank` (durable)
- **Boss Types**: 6 unique bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Bosses cycle through by stage: `BOSSES[(stage - 1) % 6]`
- Each boss has unique color scheme, HP multiplier, and attack patterns

### 7. **Boss Attack Patterns** (lines 1324-1409)
Six pattern arrays, one per boss type. Each pattern contains 3 attack functions:
- `fireRadial()` - Circular burst
- `fireAimedSpread()` - Aimed fan toward player
- `fireColumns()` - Vertical streams
- `fireSpiralBurst()` - Rotating spiral
- `fireCross()` - 8-direction burst
- `fireHomingSeeded()` - Aimed projectiles with spread
- `fireDoubleRing()` - Two concentric rings at different speeds

Boss AI: 22-frame telegraph, then execute pattern, then 85-frame cooldown before next telegraph.

### 8. **Weapon System** (lines 1124-1145)
Two weapon types with 3 upgrade levels each:
- **Vulcan** (default): Rapid-fire bullets, spread increases at levels 2 & 3
- **Laser**: Beam weapon, pierce increases at levels 2 & 3, fire rate 0.1s vs 0.15s
- Power-up cycles: Level 1→2→3→swap weapon to other at level 1

### 9. **Scoring & Chain Multiplier** (lines 1087-1101)
- Base score per enemy type: grunt=10, interceptor=15, tank=35, boss=500
- Chain: Each kill increments counter, resets after 2.2s without kills
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Score = baseScore × multiplier

### 10. **Audio System** (lines 542-676)
WebAudio synthesizer (no external files):
- `beep(freq, dur, type, vol, opts)` - Oscillator with sweep, filter, panning
- `noiseBurst(dur, vol, opts)` - Noise with lowpass filter
- Sound effects map: `sfx.shoot`, `sfx.explode`, `sfx.bomb`, `sfx.hit`, `sfx.bossTelegraph`, `sfx.bossDown`, `sfx.stageClear`, `sfx.gameover`
- Mute state persisted in localStorage

### 11. **Rendering** (lines 1520-1638)
Canvas drawing pipeline:
1. Clear and apply screen shake offset
2. Draw background gradient (6 themes with parallax clouds)
3. Draw player plane with weapon-specific aura
4. Draw bullets, enemy bullets, enemies, boss, powerups, particles
5. All sprites use simple 2D paths; planes are custom vector drawings

### 12. **Performance Adaptation** (lines 474-483)
Quality levels (0=full, 1=medium, 2=low) adjust:
- Particle count in explosions
- Glow effects on bullets and powerups
- Boss glow/telegraph effects

## Data Persistence

All data stored in localStorage with prefix `skyfighter.*`:
- `bestScore` - Best score across sessions
- `difficulty` - Selected difficulty (easy/normal/hard)
- `muted` - Audio mute state
- `reducedMotion` - Accessibility setting (respects system preference)

## Stage Progression

- **Stage Length**: 18 kills to trigger boss warning
- **Boss Spawning**: ~1.2s after 18th kill on normal difficulty
- **Theme Cycling**: Stages cycle through 6 visual themes
- **Difficulty Curve**: Spawn rate decreases (difficulty ramps) over time in each stage

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502-506)
2. Update `spawnEnemy()` weighted random selection (line 1025-1030)
3. Add explosion sound to `EXPLODE_SFX` map (line 678)
4. Create sound function in `sfx` object if using custom explosion

### Adding a New Boss Pattern
1. Create attack function (e.g., `fireLaserGrid()`) in the boss pattern functions section
2. Add 3-function array to `BOSS_PATTERNS` at the correct index
3. Extend `BOSSES` array if adding a new boss type (currently 6 bosses)
4. Update colors and HP multiplier in `BOSSES` array

### Adjusting Difficulty
Modify `DIFFICULTIES` object (line 485-489):
- `spawnBase` / `spawnMin` - Enemy spawn rate (higher = slower)
- `enemySpeed` - Multiplier for all enemy speeds
- `enemyShootChance` - Probability enemy will shoot (0-1)
- `lives` / `bombs` - Starting values
- `bossHpMul` - Boss HP multiplier

### Changing Game Balance
- Player speed: line 877 (`speed: 260`)
- Weapon cooldown: line 1177 (`fireDelay = 0.1` or `0.15`)
- Enemy bullet speed: lines 1219, 1223 (`speed: 200-220`)
- Boss HP base: line 1046 (`70 + stage * 35`)

### Adding Visual Polish
All visual parameters in `draw()` function (lines 1520-1638):
- Theme colors in `THEMES` (lines 493-500)
- Plane drawing in `drawPlane()` (lines 913-968)
- Boss drawing in `drawBossShip()` (lines 970-1020)
- Particle size and shape (line 1633)

## Known Constraints

1. **Single File**: All code in one HTML file; splitting would require build process
2. **Canvas Only**: No WebGL; performance limited by 2D canvas implementation
3. **No Local Storage Quota**: Game data (~100 bytes) is minimal
4. **Touch Controls**: Assumes touch screen resolution (mobile-optimized)
5. **Audio Context**: Requires user interaction to initialize (click/tap)

## Testing Notes

- Game state is deterministic given seed (for potential replay/tournament features)
- Quality adaptation system responds to framerate; test on varied hardware
- LocalStorage must be available; test in private/incognito mode
- Mobile: test on iOS (Safari) and Android (Chrome) for touch input
- Reduced motion: system preference respected; verify with prefers-reduced-motion media query
