# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5, Canvas, and JavaScript. It runs as a single self-contained HTML file with no build process, external dependencies, or build tools.

## Project Structure

- `index.html` - Complete game implementation (~1670 lines)
  - Embedded CSS (styling, animations)
  - Embedded JavaScript (game logic, rendering)
  - No external assets—all graphics drawn with Canvas API
  - Procedural audio synthesis with Web Audio API

The project requires no installation, build process, or dev server setup.

## Development Commands

Since this is a single HTML file with no build process:

- **Run the game locally**: Open `index.html` directly in a browser or use any local HTTP server:
  ```bash
  python3 -m http.server 8000
  # Then navigate to http://localhost:8000/index.html
  ```
- **No tests or linting** - Manual testing via browser is the primary validation method
- **No dependencies** - All code is vanilla JavaScript with no external libraries

## High-Level Architecture

### Game Loop & State Management

The game runs on `requestAnimationFrame` in a single loop function (line 1640). Core state includes:
- `running`, `paused`, `gameOver` - Game states
- `player` - Player plane object with position, health, weapons
- `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles` - All game entities stored in arrays
- `boss` - Current boss object (null when not active)
- `score`, `lives`, `stage`, `chain`, `multiplier` - Gameplay stats

### Rendering Pipeline

1. **Background** - Sky gradient + sun glow + parallax clouds (via `drawClouds()`)
2. **Player & Enemies** - Drawn with `drawPlane()` helper
3. **Bullets & Projectiles** - Simple rect/circle primitives
4. **Boss Ships** - Complex custom drawing in `drawBossShip()`
5. **UI/HUD** - Handled via DOM elements (score, hearts, bombs, boss bar)
6. **Effects** - Flash, vignette, screen shake applied via canvas transforms and DOM overlays

### Entity Systems

#### Player (`player` object)
- Properties: `x`, `y`, `w`, `h`, `speed`, `weapon`, `level`, `cooldown`, `invuln`
- Movement: Keyboard (arrows/WASD), mouse hover, or touch joystick
- Weapons: Vulcan (spread shot at level 1-3) or Laser (piercing at level 1-3)
- Invulnerability: Temporary frames after taking damage

#### Enemies
Spawned in waves via `spawnEnemy()`. Three types defined in `ENEMY_TYPES`:
- **Grunt** - Basic slow enemies, 1 HP
- **Interceptor** - Fast enemies, 1 HP, weave movement pattern
- **Tank** - Heavy armored enemies, 4 HP, slow, worth more points

Each enemy has:
- Position/movement (`x`, `y`, `speed`, `weave`, `baseX`)
- Shooting behavior (`shootTimer`, `canShoot`, `pattern`: spread or aimed)
- Type-specific rendering (color, dark color for shading)

Difficulty affects: spawn rate, enemy speed, shoot chance, tank/interceptor mix.

#### Bosses
Six boss designs in `BOSSES` array. Boss lifecycle:
1. Spawn with "APPROACHING" warning (line 1055)
2. Enter animation (`boss.entering = true`, slides down)
3. Attack phase loops: telegraphing → attack pattern → cooldown
4. Despawn on death, advance stage

Each boss has unique visual design and attack pattern set from `BOSS_PATTERNS` (line 1372-1409).

#### Bullets & Projectiles
- **Player bullets** - Stored in `bullets` array, move upward, can pierce or be single-hit
- **Enemy bullets** - Stored in `enemyBullets` array, radial or aimed, move downward
- Collision detection via `rectsOverlap()` (AABB, line 1069)

#### Particles
Explosion effects—short-lived sprites that fade and fall. Spawned by `spawnExplosion()`, count scales with performance quality.

### Weapons System

**Vulcan** (default):
- Spread shot increasing with level: [0] → [-0.12, 0.12] → [-0.24, 0, 0.24]
- Fast fire rate (0.15s cooldown)
- No pierce, 1 bullet per shot at level 1

**Laser**:
- Multi-lane fire increasing with level: [0] → [-6, 6] → [-6, 0, 6]
- Slightly slower fire rate (0.1s cooldown)
- Pierces enemies (1 or 2 per shot depending on level)
- Visual effect: Glow around player when equipped

### Boss Attack Patterns

Patterns are functions that fire bullets in various formations. Six boss types × 3 patterns each:
- **Radial/circular bursts** - `fireRadial()`, `fireDoubleRing()`
- **Aimed spreads** - `fireAimedSpread()` aims at player with spread
- **Homing/seeded** - `fireHomingSeeded()` with randomized aim
- **Geometric patterns** - `fireCross()`, `fireColumns()`, `fireSpiralBurst()`

Bosses telegraph before each attack (telegraph timer, flashing core) giving player time to react.

### Difficulty System

`DIFFICULTIES` object (line 485) defines three presets:
```javascript
{ spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul }
```
Affects:
- **Easy**: Slow spawns, weak enemies, 4 lives, 3 bombs, 0.8× boss HP
- **Normal**: Medium (default balance)
- **Hard**: Fast spawns, aggressive enemies, 2 bombs, 1.3× boss HP

### Stage & Progression

- **Kills per stage**: 18 enemies to trigger boss warning (line 491)
- **Themes**: 6 rotating themes (sky gradients, cloud colors) based on stage
- **Enemy types**: Tank and interceptor unlock at stage 2+
- **Boss cycle**: Bosses cycle through `BOSSES` array by stage

### Scoring & Multipliers

- Base points: Grunts 10, Interceptors 15, Tanks 35, Bosses 500
- **Chain system**: Kill within 2.2s window to increase multiplier (1 per 5 kills)
- Max multiplier: ×10 (at 50 kills in a chain)
- Chain resets on damage taken

### Input Handling

- **Keyboard**: Arrows/WASD for movement, Space/Left-Click for fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move cursor to steer player toward pointer (when keyboard inactive)
- **Touch**: Joystick in bottom-left, FIRE/BOMB buttons in bottom-right
- Input priority: Keyboard/Joystick > Mouse

### Performance Optimization

Adaptive quality system (line 474-483):
- Tracks frame times over 45-frame window
- Quality levels: 0 (full) → 1 (medium) → 2 (low)
- Reduces particle count and shadow effects at lower quality
- Automatically scales based on actual FPS

### Audio System

Web Audio API procedural synthesis (`ensureAudio()`, line 545):
- **No external audio files** — all sounds generated via oscillators and noise
- `beep()` - Frequency sweep with optional filter and pan
- `noiseBurst()` - White noise with lowpass filter
- `sfx` object - 14 named sound effects (shoot, laser, explode, hit, bomb, powerup, etc.)
- Mutable via UI button, settings persisted to localStorage

### Persistence

LocalStorage keys (line 463-466):
- `skyfighter.bestScore` - High score
- `skyfighter.muted` - Audio muted state
- `skyfighter.difficulty` - Last selected difficulty
- `skyfighter.reducedMotion` - Reduced motion preference (respects OS prefers-reduced-motion)

### UI/HUD Elements

DOM-based overlays:
- **Top HUD** (line 367-374) - Score, best, multiplier, weapon level, hearts, bombs
- **Boss bar** (line 376-379) - Boss name and health bar (shown only when boss active)
- **Overlay** (line 401-419) - Menu, pause screen, game-over stats
- **Effects** - Vignette (#vignette), flash (#flash), combo pop (#comboPop), stage banner (#stageBanner)

### Screen Shake & Effects

- `screenShake(magnitude, duration)` - Triggers offset jitter (disabled in reduced motion)
- `flashHit()` / `flashBomb()` - Screen flash animations
- `showComboPop(text)` - Floating "X KILLS!" text
- `showStageBanner(text)` - Stage transition banner

## Key Data Structures

### Plane Object
```javascript
{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }
```

### Enemy Object
```javascript
{ x, y, w, h, speed, shootTimer, canShoot, pattern, type, hp, color, dark, score, weave, weaveSeed, baseX }
```

### Boss Object
```javascript
{ x, y, w, h, hp, maxHp, entering, phaseTimer, attackTimer, patternIndex, vx, design, telegraph }
```

### Bullet Object
```javascript
{ x, y, w, h, speed, vx, pierce, laser } // player bullets
{ x, y, w, h, speed, vx, radial, dx, dy } // enemy bullets
```

### PowerUp Object
```javascript
{ x, y, w, h, speed, type } // "life", "bomb", or "weapon"
```

### Particle Object
```javascript
{ x, y, vx, vy, life, maxLife, color }
```

## Common Modifications

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with stats
2. Add render logic to `drawPlane()` if needed (or use default)
3. Add explosion SFX mapping to `EXPLODE_SFX` (line 678)
4. Update stage unlock condition in `spawnEnemy()` (line 1026)

### Adding a New Boss
1. Add design to `BOSSES` (line 508)
2. Create attack pattern functions (line 1324-1370)
3. Add pattern array to `BOSS_PATTERNS` (line 1372)
4. Boss cycle is automatic based on stage

### Tweaking Difficulty
- Modify `DIFFICULTIES` object (line 485) for spawn/enemy stats
- Modify base enemy speeds, player speed, cooldowns in `resetGame()` (line 875)
- Adjust boss HP formula in `spawnBoss()` (line 1046)

### Changing Visual Theme
- Modify `THEMES` array (line 493) to add sky gradients, cloud colors
- Themes cycle automatically by stage

### Adjusting Scoring
- Modify base scores in `ENEMY_TYPES` and `addScore()` (line 1087)
- Tweak multiplier formula on line 1090
- Adjust chain timer duration (line 1089)

## Testing & Debugging

Since there are no automated tests:
- **Manual play-testing** - Test each difficulty, all bosses, all weapon combos
- **Browser DevTools** - Use console to inspect game state:
  - Modify `score`, `lives`, `stage` directly to jump to scenarios
  - Check frame rate and performance metrics
  - Inspect canvas rendering and animations
- **LocalStorage** - Clear browser storage to reset high score/settings
- **Reduced motion** - Test with `prefers-reduced-motion` enabled to verify animations respect it

## Browser Compatibility

- Requires: HTML5 Canvas, Web Audio API, localStorage
- Touch support: Detects `ontouchstart` and `navigator.maxTouchPoints`
- Respects: `prefers-reduced-motion` media query for accessibility
- Target: Modern browsers (Chrome, Firefox, Safari, Edge)
