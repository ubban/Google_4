# Sky Fighter — Codebase Documentation

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built entirely in vanilla HTML5, Canvas, and Web Audio API. The game is contained in a single `index.html` file (~1670 lines) with no external dependencies beyond the browser's native APIs.

## Game Concept

Players control a fighter plane flying upward through enemy squadrons, collecting weapon power-ups, and defeating stage bosses. The game features three difficulty levels, progressive stages with distinct visual themes, a combo/multiplier scoring system, and procedurally varied boss attack patterns.

## Architecture

The entire game runs in a single IIFE (Immediately Invoked Function Expression) in the `<script>` section, ensuring all variables are scoped and isolated from global pollution.

### Core Data Structures

```javascript
// Player object
player = { 
  x, y,           // position
  w, h,           // dimensions (40x40)
  speed: 260,     // movement speed
  weapon: 'vulcan' | 'laser',
  level: 1-3,     // weapon upgrade level
  cooldown,       // fire rate throttle
  invuln          // invincibility frames after hit
}

// Enemy object (extends with type-specific properties)
enemy = {
  x, y, w, h,
  speed,          // pixels/sec * 60 in update()
  type: 'grunt' | 'interceptor' | 'tank',
  hp,             // hits to kill
  pattern: 'spread' | 'aimed',
  shootTimer,     // countdown to next shot
  weave: boolean  // interceptor weaves left/right
}

// Boss object
boss = {
  x, y, w: 90, h: 80,
  hp, maxHp,
  design: BOSSES[stage % 6],
  entering: boolean,  // initial descent animation
  telegraph: number,  // pre-attack warning frame count
  patternIndex,       // which attack pattern is next
  vx: 1 | -1          // horizontal movement direction
}
```

## Game Loop

**Function: `loop(now)` (line 1640)**

- Called via `requestAnimationFrame()` at ~60fps
- Calculates delta time: `dt = Math.min((now - lastTime) / 1000, 0.05)`
- Calls `update(dt)` if game is running and not paused
- Calls `draw()` to render the current frame
- Tracks performance and adjusts visual quality adaptively

## Core Game Systems

### 1. Input Handling

**Keyboard/Mouse (lines 768–861)**
- Arrow keys / WASD: move player
- Space bar / mouse click: fire weapon
- X / Shift: detonate bomb
- P / Escape: pause
- Mouse movement: plane glides toward pointer (lower priority than keyboard)

**Touch Controls (lines 814–834)**
- Left 55% of screen: joystick for movement
- Bottom-right: FIRE and BOMB buttons
- Joystick limited to 50px radius

### 2. Player Movement & Combat

**Function: `update(dt)` — Player Section (lines 1152–1181)**

```javascript
// Normalize movement vector from multiple input types
let mvx = 0, mvy = 0;
if (keys['ArrowLeft'] || keys['KeyA']) mvx -= 1;
if (keys['ArrowRight'] || keys['KeyD']) mvx += 1;
// ... (similar for vertical)
if (joyActive) { mvx += joyVec.x; mvy += joyVec.y; }
const mag = Math.hypot(mvx, mvy);
if (mag > 1) { mvx /= mag; mvy /= mag; } // Normalize diagonal movement

// Apply movement with collision bounds
player.x += mvx * player.speed * dt;
player.y += mvy * player.speed * dt;
player.x = Math.max(player.w / 2, Math.min(canvas.width - player.w / 2, player.x));
player.y = Math.max(player.h / 2, Math.min(canvas.height - player.h / 2, player.y));
```

**Weapon System: `fireWeapon()` (line 1124)**

- **Vulcan** (default): spreads up to 3 shots vertically depending on level
  - Level 1: 1 shot at center
  - Level 2: 2 shots at ±0.12 radians
  - Level 3: 3 shots at ±0.24 radians
- **Laser**: narrow high-speed shots with pierce capability
  - Level 1: 1 shot
  - Level 2: 2 shots at ±6px offset
  - Level 3: 3 shots (center + 2 sides)
  - Bullets persist until `pierce` count drops to 0

### 3. Enemy Spawning & Behavior

**Function: `spawnEnemy()` (line 1022)**

- Random spawn at top of canvas with random x-coordinate
- Difficulty ramps spawn rate: `spawnTimer = Math.max(d.spawnMin, d.spawnBase - (frame / 300) * 4)`
- Randomly selects enemy type based on stage and probability:
  - Tank (15% if stage ≥ 2)
  - Interceptor (25%, adds weaving)
  - Grunt (default)

**Enemy Update (lines 1210–1227)**

- Move downward at type-specific speed
- Interceptors weave: `x = baseX + Math.sin(frame * 0.05 + weaveSeed) * 60`
- Randomly choose attack pattern: spread (15%) or aimed (85%)
- Fire bullets on cooldown (30–120 frame interval)

### 4. Boss System

**Boss Spawning: `spawnBoss()` (line 1043)**

- Triggers after `stageKills >= KILLS_PER_STAGE` (18 kills)
- Health scales with difficulty: `hp = baseHp * d.bossHpMul * design.hpMul`
- Displays warning banner with boss name

**Boss Rendering: `drawBossShip(b)` (line 970)**

- Draws boss hull with gradient fill, wings, and glowing core
- Core pulses white during telegraph (pre-attack) phase
- Animated jets at bottom

**Boss AI: `updateBoss(dt)` (line 1411)**

1. **Entering phase**: Boss descends from top at 60 px/sec
2. **Active phase**: 
   - Moves left/right at 55 px/sec, bounces off walls
   - Waits for `attackTimer` to expire
   - Enters telegraph phase (core glows, 22 frames)
   - Executes attack pattern from `BOSS_PATTERNS`

**Boss Attack Patterns (lines 1372–1409)**

Each boss has 3 unique patterns, cycling through them:

| Boss | Pattern 1 | Pattern 2 | Pattern 3 |
|------|-----------|-----------|-----------|
| VANGUARD | 14-way radial burst | 5-bullet aimed fan | 3-lane wide V-pattern |
| BEHEMOTH | 5 straight columns | 7-bullet aimed spread | 20-way radial wall |
| SPECTER | Teleport + 10-way radial | 8-way spiral burst | Double 2-bullet aimed shots |
| DREADNOUGHT | 8-way cross | 5 homing missiles | 24-way dense radial |
| WRAITH | Teleport + 8-way cross | Double ring (10 + 10) | Double 3-bullet aimed spreads |
| TITAN | 5 columns | 26-way radial ring | 7 homing missiles |

### 5. Collision Detection

**Function: `rectsOverlap(a, b)` (line 1069)**

```javascript
return Math.abs(a.x - b.x) < (a.w + b.w) / 2 && 
       Math.abs(a.y - b.y) < (a.h + b.h) / 2;
```

Checks collision between two axis-aligned rectangles using center-distance math.

**Collision Resolution (lines 1232–1287)**

- Enemy bullets hit player → damage, invulnerability frames
- Player bullets hit enemies → damage, explosion
- Player/boss collision → damage
- Power-ups → apply effect
- Bomb detonation → instant kill all enemies on screen + boss damage (24 HP)

### 6. Scoring & Multiplier System

**Chain/Multiplier Tracking (lines 1087–1101)**

```javascript
function addScore(base) {
  chain++;
  chainTimer = 2.2;  // seconds before chain breaks
  multiplier = 1 + Math.min(9, Math.floor(chain / 5));
  score += base * multiplier;
}

function breakChain() {
  chain = 0;
  multiplier = 1;
}
```

- Max multiplier: **10x** (when 50+ consecutive kills without break)
- Chain resets if no kill within 2.2 seconds
- Multiplier ranges from **1–10x** based on active chain

**Base Scores (ENEMY_TYPES)**

- Grunt: 10 pts
- Interceptor: 15 pts
- Tank: 35 pts
- Boss: 500 pts
- Bomb detonation: 5 pts per enemy

### 7. Power-Up System

**Spawn Rate: `maybeSpawnPowerUp(x, y)` (line 1058)**

- 6%: Extra Life
- 8%: Smart Bomb
- 20%: Weapon Upgrade
- 66%: No drop

**Weapon Upgrade Logic: `applyPowerUp()` (lines 1304–1322)**

- Levels 1–2: Increase weapon level (more shots)
- Level 3: Switch to alternate weapon, reset level to 1
  - Vulcan ↔ Laser

### 8. Visual Effects & Rendering

**Screen Shake (line 726)**

```javascript
function screenShake(mag, dur) {
  shakeMag = reducedMotion ? mag * 0.15 : mag;
  shakeTime = reducedMotion ? Math.min(dur, 0.1) : dur;
}
```

Applied in `draw()` via canvas translation.

**Flash Effects (lines 698–709)**

- `flashHit()`: white flash on damage (25ms)
- `flashBomb()`: bright white flash on bomb (500ms)
- Respects reduced-motion preference

**Vignette (lines 726–730)**

Red pulsing vignette effect plays when player has 1 life remaining, adding urgency.

**Particle System**

Each explosion creates 3–14 particles (quality-adjusted) that:
- Spawn at object position
- Travel outward with random velocity (±200 px/sec)
- Fade over 0.5 seconds
- Render as colored circles with size proportional to remaining life

**Theme System (lines 493–500)**

Six visual themes rotate per stage:

1. **Daylight Front**: Blue sky, white clouds
2. **Sunset Corridor**: Orange/pink sky
3. **Night Approach**: Dark blue sky
4. **Storm Front**: Green sky
5. **Arctic Wastes**: Cyan sky
6. **Volcanic Ridge**: Red/orange sky

Each theme defines sky gradient colors, cloud color, and sun color.

**Drawing Functions**

- `drawPlane(x, y, w, h, color, darkColor, flame)` (line 913): Draws player/enemy planes with optional thrust flame
- `drawBossShip(b)` (line 970): Boss-specific rendering with gradient hull and glowing core
- `drawClouds(speed, fillColor, rx, ry)` (line 1507): Parallax cloud layers

### 9. Audio System (Web Audio API)

**Initialization: `ensureAudio()` (line 545)**

Creates lazy-initialized `AudioContext`, `DynamicsCompressor`, and `masterGain` node.

**Audio Functions**

- `beep(freq, dur, type, vol, opts)`: Generates oscillator-based tone with optional sweep, filter, and pan
- `noiseBurst(dur, vol, opts)`: Procedurally generated white noise burst

**SFX Library (lines 624–676)**

| Sound | Function | Technique |
|-------|----------|-----------|
| Shoot | `sfx.shoot()` | Square wave sweep 760→480Hz |
| Laser | `sfx.laser()` | Sawtooth sweep 1500→2400Hz |
| Hit | `sfx.hit()` | Double oscillator + noise burst |
| Bomb | `sfx.bomb()` | Heavy noise burst + bass sweep |
| Power-up | `sfx.powerup()` | Ascending 3-note beep sequence |
| Boss Telegraph | `sfx.bossTelegraph()` | Sweeping sine wave pair |
| Boss Defeat | `sfx.bossDown()` | Descending bass rumble (5 notes) |
| Stage Clear | `sfx.stageClear()` | Rising 4-note chime (G-B-D-G) |

All sounds respect the mute button and reduced-motion preference.

### 10. Pause & Game States

**Game State Variables (lines 864–869)**

- `running`: Game is active (not on title/game over screen)
- `paused`: Game is paused (overlay visible)
- `gameOver`: Game ended, waiting for restart
- `lives`: Current remaining lives
- `stage`: Current stage (increments after boss defeat)
- `stageKills`: Kills in current stage (resets after boss)

**Pause Logic: `togglePause()` (line 1491)**

- Shows overlay with resume button
- Blocks all input processing
- Pause button toggles between ⏸ (paused) and ▶ (running)

**Game End: `endGame()` (line 1469)**

- Stops game loop
- Checks for new personal best
- Displays final stats (score, stage, best)
- Shows "New Best Score!" message if applicable

## Difficulty System

**DIFFICULTIES object (lines 485–489)**

| Stat | Easy | Normal | Hard |
|------|------|--------|------|
| Spawn Base | 75 | 58 | 40 |
| Spawn Min | 40 | 26 | 18 |
| Enemy Speed | 1.0× | 1.4× | 1.9× |
| Enemy Shoot Chance | 45% | 70% | 95% |
| Lives | 4 | 3 | 3 |
| Bombs | 3 | 2 | 2 |
| Boss HP Multiplier | 0.8× | 1.0× | 1.3× |

Difficulty affects:
1. Enemy spawn rate (lower base = more frequent)
2. Enemy bullet velocity and accuracy
3. Boss health pool
4. Starting lives and bombs

## Game Progression

### Stage Structure

1. **Spawn Phase**: Enemies spawn continuously for 18 kills
2. **Boss Warning**: After 18 kills, 1.2 second delay before boss appears
3. **Boss Phase**: Single boss enemy with unique attack pattern
4. **Stage Clear**: 500 bonus points, advance to next stage

### Theme & Boss Rotation

- Themes rotate every stage: `THEMES[(stage - 1) % 6]`
- Bosses rotate every stage: `BOSSES[(stage - 1) % 6]`
- Game theoretically never ends (stages infinite)

## Storage & Persistence

**LocalStorage Keys**

| Key | Type | Purpose |
|-----|------|---------|
| `skyfighter.bestScore` | Number | All-time high score |
| `skyfighter.muted` | '0' \| '1' | Audio muted flag |
| `skyfighter.difficulty` | 'easy' \| 'normal' \| 'hard' | Last selected difficulty |
| `skyfighter.reducedMotion` | '0' \| '1' | Motion effect intensity |

All values persist across browser sessions.

## Responsive Design

**Canvas Sizing (lines 752–765)**

```javascript
const BASE_W = 480, BASE_H = 720;
function resize() {
  const scale = Math.min(innerWidth / BASE_W, innerHeight / BASE_H, 1.6);
  canvas.width = BASE_W;   // Logical size (never changes)
  canvas.height = BASE_H;
  canvas.style.width = Math.floor(BASE_W * scale) + 'px';
  canvas.style.height = Math.floor(BASE_H * scale) + 'px';
}
```

- Logical canvas: always 480×720
- Display size scales to fit window (max 1.6× native)
- Handles window resize and device rotation

**Safe Area Insets (line 36, 183)**

CSS `env(safe-area-inset-*)` handles notched phones (iPhone X+).

## Performance Optimization

### Adaptive Quality (lines 473–483)

Monitors frame time over 45 frames:

```javascript
if (avg > 1/40) quality++;      // Slower than 40fps → reduce quality
else if (avg < 1/56) quality--; // Faster than 56fps → increase quality
```

Quality levels:
- `quality = 0`: Full glow effects, max particles
- `quality = 1`: Reduced glow, 60% particles
- `quality = 2`: No glow, 35% particles

### Canvas Operations

- Single `ctx.clearRect()` per frame (expensive)
- Minimal state changes (save/restore only for necessary transforms)
- Particle count scales with quality
- Shadow blur disabled at quality > 0

## How to Modify

### Add a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (line 502):
   ```javascript
   myEnemy: { w: 40, h: 40, hp: 2, speedMul: 1.2, color: '#ff0000', dark: '#880000', score: 20, minStage: 3 }
   ```
2. Add to `EXPLODE_SFX` map if you want custom explosion sound (line 678)
3. Enemy will automatically spawn after its `minStage`

### Add a New Boss Pattern

1. Create pattern function that calls `fireRadial()`, `fireAimedSpread()`, etc.:
   ```javascript
   b => { fireRadial(b, 16, 160); }
   ```
2. Add 3-pattern array to `BOSS_PATTERNS` (line 1372)
3. Add boss to `BOSSES` array (line 508) to assign pattern index

### Tune Difficulty

Edit `DIFFICULTIES` object (line 485) to adjust spawn rates, enemy speed, and boss health.

### Add a New Visual Theme

Add to `THEMES` array (line 493):

```javascript
{ 
  sky: ['#start', '#middle', '#end'], 
  cloud: 'r,g,b', 
  sun: 'r,g,b', 
  name: 'Theme Name' 
}
```

### Change Game Balance

- **Player speed**: Edit `player.speed` in `resetGame()` (line 877)
- **Fire rate**: Adjust `fireDelay` values in `update()` (line 1177)
- **Bomb damage**: Edit bomb code in `useBomb()` (line 1103)
- **Enemy bullet speed**: Edit `fireXxx()` function speed parameters (lines 1324+)

## Browser Compatibility

- Requires ES6+ (arrow functions, `const`/`let`, template literals)
- Web Audio API (iOS 14.5+, all modern browsers)
- Canvas 2D context (universal)
- Touch Events API (mobile)
- LocalStorage (universal)

## Known Limitations

1. **No animation frame lag compensation**: Delta time clamped to 50ms max, may cause hitches on slow devices
2. **No spatial partitioning**: Collision checks are O(n²), becomes slow with 100+ bullets/enemies
3. **No persistent difficulty**: Difficulty must be selected each run
4. **Boss patterns fixed**: No procedural generation or learning AI
5. **No leaderboard**: Scores only stored locally

## Future Enhancement Ideas

- Persistent difficulty (resume last selected)
- Difficulty progression (unlock higher difficulties)
- Boss attack patterns that evolve mid-battle
- Power-up slots (hold multiple items)
- Weapon combo system (Vulcan + Laser simultaneous fire)
- Enemy formations with coordinated patterns
- Procedurally generated waves
- Online leaderboard integration
- Replay system / ghost challenges
- Custom game modes (endless, time attack, survival)

## File Statistics

- **Total Lines**: 1670
- **JavaScript**: 1250 lines (~75%)
- **HTML**: 60 lines (~3.6%)
- **CSS**: 360 lines (~21.4%)
- **No external dependencies**

---

**Last Updated**: 2026-10-08  
**Difficulty**: Medium (single-file arcade game, straightforward mechanics)  
**Estimated Time to Add Feature**: 30–60 minutes per feature (new enemy type, boss pattern, or theme)
