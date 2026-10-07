# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-em-up game built as a single-file HTML application. The game features:
- Three difficulty levels (easy, normal, hard)
- Progressive stage system with increasingly challenging enemies
- Six unique boss encounters with pattern-based attack systems
- Weapon power-up system with three distinct weapon types
- Smart bomb mechanics
- High score persistence via localStorage

## Architecture

### Single-File Design
All game code is in `index.html` (~1,669 lines). The file is organized into distinct sections:

1. **HTML Structure & CSS** (lines ~1-420): Canvas, overlay UI, touch controls, and all styling
2. **Game Script** (lines ~422-1666): Wrapped in an IIFE to isolate namespace

### Core Game Systems

**Game Loop** (line ~1640)
- Uses `requestAnimationFrame` for smooth rendering (~60fps)
- Frame-limited to max 50ms per frame to prevent catching up if performance drops
- Calls `update(dt)` for game logic, then `draw()` for rendering
- Adaptive performance tracking adjusts quality based on frame times

**State Variables**
- `running`: Whether a game is in progress
- `paused`: Whether game is paused (P or Esc key)
- `gameOver`: Whether player has lost all lives
- `stage`: Current stage number (1-indexed)
- `player`: Object with x, y, w, h, vx, vy, weapon, invuln, etc.
- `boss`: Current boss object (null if none active)
- `enemies`: Array of active enemy objects
- `playerBullets`: Array of player projectiles
- `enemyBullets`: Array of enemy projectiles
- `particles`: Array of visual particles (dust, explosions, etc.)
- `pickups`: Array of weapon and bomb pickups

### Enemy System
Enemy types defined in `ENEMY_TYPES` (line ~502):
- **Grunt**: Basic red fighter, 1 HP, appears from stage 1
- **Interceptor**: Fast orange enemy with weaving behavior, 1 HP, appears from stage 1
- **Tank**: Slow purple heavy unit, 4 HP, appears from stage 2+

Spawning controlled by difficulty settings (`DIFFICULTIES` object, line ~485):
- Spawn rate decreases as stage progresses (fewer frames between spawns)
- Minimum spawn delay increases difficulty
- Enemy speed and shoot chance scale with difficulty

### Boss System
Six boss designs in `BOSSES` (line ~508) with unique color schemes and HP multipliers:
1. **VANGUARD**: Radial burst, aimed fan, V-fan patterns
2. **BEHEMOTH**: Heavy columns, homing spread, dense radial wall
3. **SPECTER**: Teleport dash, spiral, double aimed shots
4. **DREADNOUGHT**: 8-way cross fire, homing missiles, radial wall
5. **WRAITH**: Teleport barrage, expanding double ring, cross-spread
6. **TITAN**: Heavy columns, full ring, homing volley

Each boss cycles through 3 attack patterns with telegraph warning animations before each attack. Boss patterns are defined in `BOSS_PATTERNS` (line ~1372).

### Weapon System
Three main weapons accessible via weapon power-ups:
1. **Spread Shot**: Fires bullets in a cone pattern (3 bullets)
2. **Laser**: Concentrated beam with higher damage falloff
3. **Twin Shot**: Dual streams from left/right (no implementation detail visible in scanned portion)

Weapon state stored in `player.weapon` and level tracked in `player.weaponLevel`.

### Visual Effects & Feedback
- **Screen shake**: `screenShake(mag, dur)` - applies temporary canvas offset
- **Flash effects**: `flashHit()` (brief white flash on player damage), `flashBomb()` (bomb detonation flash)
- **Combo pop**: Large animated text showing combo/score milestone
- **Stage banner**: Large text showing stage name and theme
- **Vignette**: Red vignette overlay when taking damage (visual hurt feedback)

### Audio System
WebAudio synthesizer-based (no external sound files). Key synthesis functions:
- `beep(freq, dur, type, vol, opts)`: Tonal sound effects (tunable frequency)
- `noiseBurst(dur, vol, opts)`: Noise-based effects (explosion, hit sounds)
- Sound effects muted via localStorage preference

### Performance Optimization
Adaptive quality system (`quality` variable, line ~474):
- Tracks last 45 frame times
- Scales quality down (quality 2 = low) if avg frame time exceeds 1/40s
- Scales quality up if avg frame time better than 1/56s
- Quality affects particle effects and rendering detail

## Common Development Tasks

### Running the Game
1. Open `index.html` directly in a web browser (no build step required)
2. Or host via simple HTTP server: `python3 -m http.server 8000` then visit `http://localhost:8000`

### Making Difficulty Adjustments
Edit the `DIFFICULTIES` object (line ~485):
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

### Adding/Modifying Boss Patterns
1. Define attack pattern function in appropriate boss section (e.g., within fireRadial, fireColumns, etc.)
2. Add to `BOSS_PATTERNS` array (line ~1372) in the 3-pattern sequence for the boss index
3. Patterns are functions that take boss object `b` and spawn enemy bullets via `enemyBullets.push()`

### Adding New Enemy Types
1. Add entry to `ENEMY_TYPES` object (line ~502) with properties: w, h, hp, speedMul, color, dark, score, minStage
2. Modify `spawnEnemy()` (line ~1022) to include the new type in spawn pool based on current stage
3. Update `update()` function (line ~1148) collision logic if special behavior needed

### Adjusting Weapon Behavior
Weapon firing is handled by `fireWeapon()` (line ~1124), which branches on `player.weapon` type. Each weapon type spawns playerBullets with specific patterns:
- Spread: 3 bullets at angles
- Laser: Single centered bullet with higher speed
- Twin: Two offset bullets

### Testing Gameplay
- Use browser DevTools (F12) → Console to inspect game state: `player`, `enemies`, `stage`, `boss`
- Modify game variables directly in console to test specific scenarios
- Difficulty can be changed via difficulty buttons before game start, or via `setDifficulty('hard')` in console

### Persistence
Game state persists to localStorage via these keys (line ~463):
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute toggle (0 or 1)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion reduction preference (0 or 1)

## Key Functions by Purpose

**Collision & Physics**
- `rectsOverlap(a, b)`: AABB collision detection
- `canvasPointFromEvent(e)`: Convert mouse/touch event to canvas coordinates

**Rendering**
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Draw player or enemy ship
- `drawBossShip(b)`: Draw boss with detailed hull/wing rendering
- `spawnExplosion(x, y, color, count)`: Particle explosion effect
- `draw()`: Main render function (scanned portion not fully visible)

**Game Logic**
- `update(dt)`: Main update tick - handles all game state changes
- `applyPowerUp(type)`: Process weapon/bomb pickups
- `addScore(base)`: Add score with multiplier chain
- `breakChain()`: Reset combo/multiplier on damage
- `useBomb()`: Smart bomb detonation logic
- `loseLife()`: Handle player taking damage
- `endGame()`: Game over sequence

## Themes & Visuals
Six sky themes in `THEMES` (line ~493) that cycle per stage. Each theme has:
- Sky gradient colors (3 values)
- Cloud color (RGB)
- Sun color (RGB)
- Theme name (displayed on stage banner)

## Controls
- **Desktop**: Arrow keys or WASD to move, Space/Click to fire, X or Shift to bomb, P or Esc to pause
- **Mobile**: Drag finger on left side to move (joystick), tap FIRE button, tap BOMB button
- Reduced motion preference respected via `prefers-reduced-motion` media query

## Important Notes
- Game is **canvas-based and fully self-contained** — no external assets, images, or audio files
- Uses WebAudio synthesizer for all sound effects
- Performance-adaptive (scales quality based on frame times)
- Accessibility: screen reader announcements for major events via `announce()` function
- Mobile-first responsive design with safe-area insets for notched devices
