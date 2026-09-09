# Sky Fighter – Codebase Documentation

## Overview
**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game built as a single HTML file with embedded CSS and JavaScript. The game features adaptive difficulty, multiple weapon types, boss battles with pattern-based attacks, and full responsive design supporting desktop and mobile controls.

- **File:** `index.html` (single file, ~1700 lines)
- **Technologies:** Vanilla HTML5 Canvas, Web Audio API, CSS Grid/Flexbox
- **Platform:** Desktop (mouse/keyboard) + Mobile (touch controls with virtual joystick)

## Architecture Overview

### Game Loop
The core game loop runs at 60 FPS using `requestAnimationFrame`:
- **Initialization:** `resetGame()` sets up player, enemies, and level state
- **Input:** Keyboard (arrows, WASD, Space, X, P) and mouse/touch handled via global listeners
- **Update:** `update(dt)` processes physics, collisions, spawning (called only when `running && !paused`)
- **Render:** `draw()` renders all game elements to canvas with optional screen shake
- **Performance:** Adaptive quality system throttles particle count and shadows based on frame time

### Game State Variables
```javascript
// Player
player { x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }

// Collections
bullets[], enemies[], enemyBullets[], particles[], powerUps[], boss

// Scoring & Progress
score, lives, maxLives, bombCount, maxBombs
frame, kills, stageKills, stage, spawnTimer
chain, chainTimer, multiplier

// UI
running, paused, gameOver, bossWarned
```

### Key Constants
- **BASE_W, BASE_H:** Canvas resolution (480×720, scales up to 1.6x or fills viewport)
- **DIFFICULTIES:** Three presets (easy/normal/hard) affecting spawn rates, enemy behavior, boss HP
- **KILLS_PER_STAGE:** 18 kills required before boss spawns
- **THEMES:** 6 sky/cloud color schemes cycling per stage
- **ENEMY_TYPES:** grunt, interceptor, tank with varying HP, speed, score values
- **BOSSES:** 6 boss designs with unique attack patterns, visual appearance, and HP multipliers

---

## Core Systems

### 1. Input Handling

**Desktop:**
- Arrow keys or WASD: Move player
- Space: Fire weapon
- X or Shift: Drop bomb
- P or Escape: Pause
- Mouse: Move to cursor (when hovering canvas), click to fire

**Mobile:**
- Left zone (55% width): Virtual joystick for movement
- Right zone (45% width): FIRE and BOMB buttons
- Touch activates audio context (required by browser policy)

**Input State:**
- `keys{}` object tracks held keys
- `mouseActive` flag and `mouseTarget {x, y}` for mouse steering
- `joyActive`, `joyVec {x, y}`, `joyOrigin` for touch joystick

### 2. Player Movement & Weapon System

**Movement:**
```javascript
// Priority: keyboard/joystick > mouse glide
// Player speed: 260 px/s, clamped to canvas bounds
```

**Weapons:**
1. **Vulcan** (default): Spread shot that expands with levels
   - Level 1: Single center bullet
   - Level 2: Two angled bullets
   - Level 3: Three bullets (left/center/right)
   - Fire rate: 150ms

2. **Laser**: Piercing beam(s) that pass through enemies
   - Level 1: Single center laser
   - Level 2: Two side lasers
   - Level 3: Center + two side lasers (allows 2 piercings per laser)
   - Fire rate: 100ms

Weapon changes via power-up: Each level increases current weapon; level 4 switches to other weapon at level 1.

### 3. Enemy System

**Spawning:**
- Spawn timer decreases each frame; when ≤ 0, spawn one enemy
- Spawn rate scales with difficulty and accelerates over time (`difficultyRamp = frame / 300 * 4`)
- Enemy types unlock by stage (tank at stage 2+)

**Enemy Types:**

| Type | Size | HP | Speed | Score | Behavior |
|------|------|-----|-------|-------|----------|
| Grunt | 34×34 | 1 | 1.0x | 10 | Straight down, spread shot (45% rate) |
| Interceptor | 26×26 | 1 | 1.7x | 15 | Fast, weaving side-to-side, aimed shot (70% rate) |
| Tank | 46×46 | 4 | 0.55x | 35 | Slow tanky, aimed shot (95% rate), shows HP bar |

**Behavior:**
- All move downward at base speed (modified by difficulty)
- Shoot pattern: either radial spread or aimed at player (determined at spawn)
- Shooting delay: 60–150 frames after each volley
- Weaving (interceptor): sine wave oscillation around spawn X

### 4. Boss System

**Boss Spawning:**
- Triggers when `stageKills >= 18` (KILLS_PER_STAGE)
- 1.2s delay after threshold reached
- Boss banner announces approach: `"WARNING — [BOSS_NAME] APPROACHING"`

**Boss States:**
- **Entering:** Boss slides down from top to Y=120
- **Active:** Horizontal patrol with pattern attacks and telegraphing

**Boss Designs (6 total, cycle per stage):**

| Name | Style | HP Mult | Pattern Count |
|------|-------|---------|---------------|
| VANGUARD | Purple, sleek | 1.0x | Radial burst → aimed fan → V-fan |
| BEHEMOTH | Orange, bulky | 1.35x | Columns → aimed spread → dense radial |
| SPECTER | Teal, agile | 0.8x | Teleport+radial → spiral → double aimed |
| DREADNOUGHT | Gray, balanced | 1.55x | 8-way cross → homing → dense radial |
| WRAITH | Violet, mystic | 0.95x | Teleport+cross → double ring → aimed cross |
| TITAN | Gold, massive | 1.7x | Columns → full ring → homing volley |

**Boss Attack Flow:**
1. Boss holds position while `telegraph > 0` (22 frames)
2. Sound cue: `bossTelegraph()`
3. Execute attack pattern from `BOSS_PATTERNS[bossIndex][patternIndex % 3]`
4. Cycle to next pattern, wait `attackTimer = 85` frames

**Boss Bullet Types:**
- **Radial:** Bullets with `dx, dy` velocity vectors (from `fireRadial`, `fireAimedSpread`, etc.)
- **Aimed:** Bullets fired at player with optional spread angles
- **Homing Seeded:** Aimed with random drift applied at spawn

### 5. Collision System

**Rectangles Overlap:**
```javascript
rectsOverlap(a, b) // true if centers are within combined half-widths/heights
```

**Collision Types:**
1. **Enemy ← Player bullet:** Damage enemy HP, reduce bullet pierce (lasers only), spawn small explosion
2. **Boss ← Player bullet:** Damage boss 3 HP, reduce pierce, play boss-hit sound
3. **Player ← Enemy/Bullet/Boss:** Lose life, spawn explosion, start invulnerability (1.5s)
4. **Player ← Power-up:** Apply effect, remove power-up

### 6. Scoring & Combo Chain

**Chain/Multiplier System:**
- Each kill increments `chain` and resets `chainTimer` (2.2s)
- `multiplier = 1 + min(9, floor(chain / 5))`  → up to 10x (at 50 kills)
- When `chainTimer` expires without new kill: `breakChain()` resets to 1x
- Score shown in HUD, updated on every kill

**Milestone Popups:**
- Every 10 kills: `"[N] KILLS!"` popup with glow effect

### 7. Power-Up System

**Types & Spawn Rates (on enemy death):**
- 6%: Life (full heart) → +1 life (capped at maxLives)
- 8%: Bomb → +1 bomb (capped at maxBombs)
- 20%: Weapon → +1 level or switch weapon
- 66%: Nothing

**Behavior:**
- Spawn at enemy death location
- Fall downward at 90 px/s
- Despawn if off-screen
- Single sprite with rotation animation

### 8. Audio System

**Web Audio Synthesis (no external files):**
- Single `AudioContext` with master gain (0.9) and compressor
- Lazy init: first input triggers `ensureAudio()`
- All sounds generated via `beep()` (sine/square/sawtooth + optional LFO/filter/pan) or `noiseBurst()`

**Sound Effects:**
- `shoot`, `laser`: Quick tones with frequency sweep
- `explodeGrunt`, `explodeInterceptor`, `explodeTank`: Unique signatures per type
- `hit`, `bomb`, `powerup`: Player interaction feedback
- `bossHit`, `bossTelegraph`, `bossDown`: Boss-specific cues
- `stageClear`, `gameover`, `uiClick`: UI/flow sounds

**Audio Config:**
- Stored in localStorage: `skyfighter.muted`
- Toggle via mute button (🔊/🔇 icon)
- Panning based on X position (`panFor(x)`)

### 9. Visual Effects

**Screen Shake:**
- Triggered on bomb, boss defeat, player hit
- Stored as `shakeMag, shakeTime` (duration in seconds)
- Applied as random canvas translation in `draw()`
- Reduced by 85% if user prefers reduced motion

**Flash Overlay:**
- Full-screen white flash on hit or bomb detonation
- CSS animations (`flashHit`, `flashBomb`)
- Skipped if reduced motion enabled

**Vignette (Red Pulse):**
- Radial gradient overlay that pulses when `lives === 1`
- Indicates critical health state
- Respects reduced motion preference

**Particle Explosions:**
- `spawnExplosion(x, y, color, count)` creates particles with random velocity
- Particles decay over 0.5s with radius shrinking
- Adaptive: particle count scales by quality level (full / 60% / 35%)

**Combo Popup:**
- Large gold text at screen center, scales and fades over 0.6s
- Triggered every 10 kills

**Stage Banner:**
- `"STAGE [N]"` or `"WARNING — [BOSS]"` text
- Scales up and fades out over 2s at 42% from top
- Announced via accessibility live region

**Boss Health Bar:**
- Semi-transparent bar at top-center, shows current/max HP percentage
- Fades in/out with boss presence
- Color: linear gradient from red (#ff5252) to orange (#ff8a00)

### 10. Difficulty & Performance

**Difficulty Presets:**

| Setting | Easy | Normal | Hard |
|---------|------|--------|------|
| Spawn Base | 75 | 58 | 40 |
| Spawn Min | 40 | 26 | 18 |
| Enemy Speed | 1.0x | 1.4x | 1.9x |
| Enemy Shoot Chance | 45% | 70% | 95% |
| Lives | 4 | 3 | 3 |
| Bombs | 3 | 2 | 2 |
| Boss HP Mult | 0.8x | 1.0x | 1.3x |

**Adaptive Quality:**
- Tracks average frame delta time (last 45 frames)
- `quality = 0` (full): shadows, max particles
- `quality = 1` (medium): reduced particles/shadows
- `quality = 2` (low): minimal effects
- Adjusts every 45 frames based on average FPS

---

## UI & Accessibility

### HUD (Heads-Up Display)
- **Center top:** Score, Best, Multiplier (x), Weapon (name + level), Hearts (❤️), Bombs (💣)
- **Right top:** Mute (🔊), Motion toggle (🎆), Pause (⏸) buttons
- **Top-center (when boss active):** Boss name + HP bar

### Overlay (Menu/Pause/GameOver)
- Large title with gradient text
- Subtitle (when showing game over: score, stage, best)
- Difficulty buttons (Easy/Normal/Hard) — disabled during gameplay
- "Start Game" or "Resume" button
- Controls hint (keyboard + mobile)

### Accessibility
- Screen reader live region (`#srLive`) announces: state changes, score milestones, boss defeats
- Reduced motion preference respected via `prefers-reduced-motion` media query
- Motion toggle button accessible during gameplay (🎆/🚫 icon)
- All interactive buttons have `title` attributes

### Mobile Responsiveness
- Canvas: scales to fit viewport (max 1.6x native resolution)
- Touch controls only visible on touch devices
- HUD text scales with viewport (clamp functions)
- Safe area insets respected for notched devices

---

## Key Functions & Entry Points

### Game Flow
```javascript
resetGame()           // Initialize/restart game state
update(dt)            // Main game logic per frame (only if running && !paused)
draw()                // Render frame to canvas
loop(now)             // requestAnimationFrame callback
```

### Enemy & Boss
```javascript
spawnEnemy()          // Create random enemy based on stage/difficulty
spawnBoss()           // Create boss (pattern assigned by stage)
updateBoss(dt)        // Boss movement and attack logic
killBoss()            // Defeat boss, advance stage
```

### Collision & Damage
```javascript
rectsOverlap(a, b)    // AABB collision check
addScore(base)        // Award points with multiplier
loseLife()            // Lose a life, check game over
useBomb()             // Detonate bomb: clear bullets, damage enemies/boss
```

### Weapons & Power-ups
```javascript
fireWeapon()          // Spawn bullets based on current weapon/level
fireRadial(b, ...)    // Boss pattern: bullets in all directions
fireAimedSpread(...)  // Boss pattern: fan aimed at player
maybeSpawnPowerUp()   // 34% chance to drop item at position
applyPowerUp(type)    // Apply effect (life/bomb/weapon)
```

### Rendering
```javascript
drawPlane(x, y, w, h, color, dark, flame)     // Draw player/enemy sprite
drawBossShip(boss)                             // Draw boss sprite with animation
drawClouds(speed, fill, rx, ry)                // Parallax cloud layers
spawnExplosion(x, y, color, count)             // Create particle burst
```

### Input & State
```javascript
ensureAudio()         // Create AudioContext on first user gesture
resize()              // Recalculate canvas scale on window resize
setDifficulty(name)   // Store difficulty preference, update UI
togglePause()         // Pause/resume gameplay
```

---

## LocalStorage Schema

All preferences persist across sessions using `localStorage`:

```javascript
STORAGE_BEST     = 'skyfighter.bestScore'      // Number
STORAGE_MUTE     = 'skyfighter.muted'          // '0' or '1'
STORAGE_DIFF     = 'skyfighter.difficulty'     // 'easy' | 'normal' | 'hard'
STORAGE_REDUCED  = 'skyfighter.reducedMotion'  // '0' or '1' (if explicitly set; otherwise uses system preference)
```

---

## Common Modifications

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `{ w, h, hp, speedMul, color, dark, score, minStage, weave? }`
2. Add explosion SFX to `EXPLODE_SFX` mapping
3. Update `spawnEnemy()` logic to include it in the random choice
4. Create sound effect in `sfx` object (e.g., `explodeCustom()`)

### Add a New Boss
1. Add entry to `BOSSES` array with `{ name, hullTop, hullBottom, wing, core, hpMul }`
2. Add attack pattern array to `BOSS_PATTERNS` at corresponding index
3. Implement fire functions (e.g., `fireRadial`, `fireAimedSpread`, `fireCross`, etc.)
4. Boss is assigned by `(stage - 1) % BOSSES.length`, so new bosses cycle through stages

### Adjust Difficulty
1. Modify `DIFFICULTIES` constants (spawn rates, enemy speed, lives, bombs, boss HP)
2. Or tweak spawn acceleration: `difficultyRamp = Math.floor(frame / 300) * 4`

### Customize Colors & Themes
- Edit `THEMES` array for sky gradients, cloud colors, and sun color
- Change enemy/boss `color`, `dark` properties for visual style
- Modify HUD and button colors in CSS `:root` or inline `fillStyle`

### Change Canvas Resolution
- Modify `BASE_W` (480) and `BASE_H` (720)
- Adjust `KILLS_PER_STAGE` if difficulty balance shifts
- Tweak spawn `x` range to match new width

---

## Testing & Debugging

### Browser Console
```javascript
// Cheat: Skip to stage 5
stage = 5; stageKills = 0; bossWarned = false;

// Cheat: Add lives
lives = maxLives;

// Cheat: Fill bomb count
bombCount = maxBombs;

// Inspect current game state
console.log({ player, score, stage, boss });
```

### Performance Profiling
- Check `quality` variable (0 = full, 1 = medium, 2 = low)
- Monitor frame times in browser DevTools Performance tab
- Reduce `frameTimes.length` window from 45 if running on slower device

### Audio Debugging
- Check `audioCtx.state` (should be "running" after first input)
- Mute toggle persists; check `localStorage.getItem('skyfighter.muted')`
- Sounds require `masterGain` to be connected to destination

---

## Future Enhancements

- [ ] Multiple lives display beyond 3 (stack hearts differently)
- [ ] Procedural enemy formation waves
- [ ] Special weapon types (shotgun, flamethrower, etc.)
- [ ] Progressive cosmetics/skins for player ship
- [ ] Level select or stage skip
- [ ] Global leaderboard integration
- [ ] Replay recording and playback

---

## Architecture Notes

- **Single File Design:** All code in one HTML for portability and distribution
- **Canvas Only:** No DOM manipulation for game entities (performance)
- **Inline Web Audio:** No external audio assets; synthesis saves bandwidth
- **Responsive:** Base 480×720 resolution scales to fit any device
- **Accessibility First:** Screen readers, reduced-motion support, keyboard + mouse + touch
- **No Dependencies:** Vanilla JS, works in any modern browser with Canvas + Web Audio support

