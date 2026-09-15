# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML application with embedded CSS and JavaScript. It runs entirely in the browser using Canvas 2D rendering and WebAudio synthesis (no external assets).

- **File**: `index.html` (single entry point, ~1670 lines)
- **Runtime**: Modern browsers with Canvas and WebAudio support
- **Key Features**: Player aircraft, multiple enemy types, boss battles, weapon upgrades, combo system, difficulty modes, touch controls, local storage persistence

## Architecture

### High-Level Design

The entire game runs in a single IIFE (Immediately Invoked Function Expression) starting at line 423. The codebase follows a functional structure with game state variables and modular systems:

```
Game State
├── Player (x, y, weapon, level, invulnerability)
├── Entities (bullets, enemies, enemy bullets, particles, power-ups, boss)
├── Scoring (score, kills, multiplier chain, best score)
├── UI State (overlay, HUD elements, pause/game-over)
└── Settings (difficulty, mute, reduced motion, theme)

Systems
├── Input (keyboard, mouse, touch/joystick)
├── Physics (movement, collision detection, entity cleanup)
├── Rendering (canvas draw, clouds, planes, bosses, particles)
├── Audio (synthesized SFX via WebAudio)
├── Game Loop (requestAnimationFrame at ~60fps)
└── Difficulty/Theme (6 themes, 3 difficulty levels)
```

### Key Systems

**1. Entity Lifecycle**
- Entities are stored in arrays: `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`
- Each frame, entities update position, collisions are checked, and out-of-bounds entities are filtered
- Spawning is probabilistic (enemies) or event-driven (power-ups, boss)

**2. Player System** (line 875-1170)
- Movement via keyboard (arrows/WASD), mouse hover, or joystick (touch)
- Two weapons with 3 upgrade levels each: Vulcan (spread), Laser (pierce)
- Firing is continuous while key/button is held; cooldown throttles rate
- Invulnerability window (1.5s) after taking damage

**3. Combat System**
- **Weapons** (fireWeapon, line 1124): Vulcan spreads bullets; Laser fires lanes with pierce
- **Enemy Fire** (enemies shoot toward/spread from their position, line 1216-1226)
- **Boss Patterns** (BOSS_PATTERNS, line 1372-1409): 6 bosses × 3 unique attack patterns each
- **Collision** (rectsOverlap, line 1069): AABB checks drive hit detection
- **Damage** (line 1237-1266): Bullets deplete enemy HP; boss HP displayed in bar

**4. Difficulty & Progression** (lines 485-512)
- **DIFFICULTIES**: Easy/Normal/Hard affect spawn rate, enemy speed, shoot chance, lives, bombs, boss HP
- **ENEMIES** (ENEMY_TYPES, line 502-506): 3 types unlock at different stages
- **BOSSES** (line 508-515): 6 named bosses cycle; HP scales by difficulty × design multiplier
- **THEMES** (line 493-500): 6 visual themes cycle; stage progression is endless
- **Kills Per Stage** (KILLS_PER_STAGE = 18, line 491): After 18 kills, boss spawns

**5. Scoring & Chain Multiplier** (lines 1087-1101)
- Score = basePoints × multiplier; multiplier increases with kill chain
- Chain resets if player takes damage; 2.2s window to continue chain
- Multiplier maxes at 1 + floor(chain / 5), capping at 10x

**6. Audio** (lines 542-676)
- **WebAudio Synthesizer**: No asset files; generates sounds in-game
- **Beep**: Oscillator with optional frequency sweep, filter, pan, gain envelope
- **NoiseBurst**: Procedural noise decayed through filter
- **SFX Map** (line 624-676): 13 different sounds (shoot, laser, explosions by type, hit, bomb, powerup, boss telegraph, gameover, etc.)

**7. Rendering** (lines 1520-1638)
- **Sky**: Gradient + cloud layers (semi-transparent circles at different scroll speeds)
- **Entities**: Planes drawn via drawPlane (custom polygon), bosses via drawBossShip
- **Glow**: Glowing shadows on bullets/powerups when quality === 0
- **Particles**: Explosion debris; life decays, alpha follows

**8. Touch Controls** (lines 778-834)
- **Move Zone** (left 55% of screen): Virtual joystick; fires events touch{start,move,end}
- **Action Zone** (right 45%): Fire and Bomb buttons
- **Mouse Fallback**: Plane glides toward cursor; keyboard input overrides
- **Audio Resume**: touchstart/mousedown must call ensureAudio (unlock browser audio)

**9. Local Storage** (lines 463-466, 517-527)
- Persists: best score, mute state, difficulty, reduced motion preference

### Performance & Quality

- **Adaptive Quality** (trackPerf, line 474-483): Monitors avg frame time; downgrades quality (particle count, glow effects) if FPS dips
- **Canvas Scaling** (resize, line 754-765): Base canvas 480×720; CSS scales to fit screen (max 1.6× scale)

## Common Development Tasks

### Run the Game

Open `index.html` in a browser. Game starts paused with menu overlay.

### Test Gameplay Flow

1. Click "Start Game" to begin
2. **Controls**: 
   - Keyboard: Arrow keys or WASD to move; Space to fire; X/Shift to bomb; P/Esc to pause
   - Mouse: Move pointer to steer; click to fire
   - Touch: Drag left for movement; tap FIRE/BOMB
3. **Progression**: Defeat 18 enemies per stage → boss appears; defeat boss → next stage
4. **Goals**: Maximize score by chaining kills and upgrading weapons

### Modify Difficulty Balance

**File**: Lines 485-489 (DIFFICULTIES object)

```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

- `spawnBase` / `spawnMin`: Enemy spawn frequency (frames between spawns; lower = faster)
- `enemySpeed`: Movement speed multiplier
- `enemyShootChance`: Probability each enemy fires on spawn
- `lives`: Starting lives
- `bombs`: Starting bombs
- `bossHpMul`: Boss HP multiplier

### Add a New Boss Pattern

**File**: Lines 1372-1409 (BOSS_PATTERNS array)

Each boss gets 3 attack patterns. Pattern functions receive boss object `b` and fire enemy bullets via helper functions:
- `fireRadial(b, count, speed, [offset])`: Ring of bullets
- `fireAimedSpread(b, count, speed, spreadStep)`: Fan toward player
- `fireColumns(b, speed)`: Vertical columns
- `fireSpiralBurst(b, speed)`: Spinning burst
- `fireHomingSeeded(b, count, speed)`: Spread with player-aimed base

Example: Add a new pattern to VANGUARD (index 0):
```javascript
BOSS_PATTERNS[0].push(
  b => { fireRadial(b, 12, 100); fireRadial(b, 12, 180, Math.PI / 12); }
);
```

### Tweak Enemy Spawning

**File**: Lines 1022-1041 (spawnEnemy) and 1197-1203 (spawn timer logic)

- Adjust type probabilities: line 1029 (tank at `r < 0.15`)
- Adjust minimum stage for unlock: line 1026 and ENEMY_TYPES.minStage
- Spawn timer calculation: line 1202 (adds difficulty ramp every 300 frames)

### Modify Sound

**File**: Lines 542-676 (audio system)

- `beep(freq, dur, type, vol, opts)`: Oscillator-based tone
  - `opts.sweepTo`: Target frequency (exponential ramp)
  - `opts.filterFreq`, `opts.filterType`: Lowpass/highpass filter
  - `opts.pan`: Stereo pan (-1 to 1)
  - `opts.delay`: Delay before note starts
- `noiseBurst(dur, vol, opts)`: Procedural noise (good for explosions)
- `sfx.<name>()`: Map of effects (line 624-676); call `sfx.uiClick()` etc. to play

To add a new sound, define it in the `sfx` object and call it when needed (e.g., `sfx.newSound()` on event).

### Change Visual Theme

**File**: Lines 493-500 (THEMES array)

Each theme defines sky gradient stops, cloud color, sun color, and name. Themes cycle per stage: `THEMES[(stage - 1) % THEMES.length]`.

Add a new theme:
```javascript
{ sky: ['#start', '#mid', '#end'], cloud: 'R,G,B', sun: 'R,G,B', name: 'Theme Name' }
```

### Adjust UI Layout

**File**: Lines 10-359 (CSS in <style>)

- Canvas size: Line 753 (BASE_W = 480, BASE_H = 720)
- HUD position/style: Lines 34-70
- Overlay (menu/game-over): Lines 200-250
- Animation keyframes: @keyframes rules throughout
- Reduced-motion support: Lines 349-358 (disables animations if user prefers)

### Debug Game State

Browser console: Access `score`, `lives`, `stage`, `player.weapon`, `player.level`, `frame`, etc. directly (not scoped in IIFE; would need console patching for live inspection).

To add logging: Insert `console.log()` in the `update()` function (line 1148+).

## Data Structures

- **Player**: `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- **Bullet**: `{ x, y, w, h, speed, [vx], [pierce], [laser] }`
- **Enemy**: `{ x, y, w, h, speed, shootTimer, canShoot, pattern, type, hp, color, dark, score, weave, weaveSeed, baseX }`
- **Boss**: `{ x, y, w, h, hp, maxHp, entering, phaseTimer, attackTimer, patternIndex, vx, design, telegraph }`
- **PowerUp**: `{ x, y, w, h, speed, type }` (type: 'life' | 'bomb' | 'weapon')
- **Particle**: `{ x, y, vx, vy, life, maxLife, color }`

## Browser Compatibility

- **Canvas 2D**: Requires `getContext('2d')` support
- **WebAudio**: Requires AudioContext (with webkit fallback)
- **Touch**: Detects `ontouchstart` or `navigator.maxTouchPoints > 0`
- **LocalStorage**: For persistence (graceful fallback if unavailable)
- **Tested Browsers**: Modern Chrome, Firefox, Safari, Edge

## Files

- `index.html` (1670 lines): Complete game
- `README.md`: Minimal title + timestamp
- `CLAUDE.md`: This file
