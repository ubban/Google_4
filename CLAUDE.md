# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML5 web application. It features:
- Procedurally generated enemies and bosses with unique attack patterns
- Multiple difficulty levels (Easy/Normal/Hard)
- Weapon upgrade system (Vulcan vs. Laser with 3 power levels)
- Boss encounter system with stage progression
- Score multiplier chain system
- Fully procedural audio using WebAudio API
- Mobile touch controls and keyboard input
- No external dependencies — pure vanilla HTML/CSS/JavaScript and Canvas API

## Running & Development

### To play locally
Open `index.html` directly in any modern web browser. The game runs entirely in the browser with no build step required.

### Key files
- `index.html` — The entire game: HTML structure, inline CSS, and the game loop/logic

### Performance & Testing
The game includes **adaptive quality scaling**: if frame times exceed ~25ms (40fps), it reduces particle/visual fidelity. Check the `quality` variable and `trackPerf()` function around line 474.

## Architecture Overview

### Single-File Structure
Everything is in one HTML file (~1670 lines) to keep deployment simple:
1. **`<head>`** — Inline CSS (~360 lines) with responsive design using `clamp()` for mobile
2. **`<body>`** — DOM structure: canvas, HUD, overlay screens, touch controls
3. **`<script>`** — Complete game logic in an IIFE (~1300 lines)

### Game Loop & State
- **Main loop**: `loop(now)` at line 1640 — runs via `requestAnimationFrame`
- **Update phase**: `update(dt)` at line 1148 — handles input, movement, collision, spawn logic
- **Draw phase**: `draw()` at line 1520 — renders all entities, effects, and HUD
- **Game state**: Global variables (player, bullets, enemies, boss, score, lives, etc.) at line 864

### Entity Systems

#### Player
- Location: ~line 877 (in `resetGame()`)
- Attributes: position, weapon (vulcan/laser), level (1–3), health, invulnerability
- Weapons: 
  - **Vulcan** (default): spread pattern increases with level (1–3 bullets)
  - **Laser**: pierce damage, levels 1–3 add lanes; can penetrate multiple enemies
- Input: Keyboard (arrow keys/WASD), mouse (position-based steering), joystick (touch), Space/click to fire

#### Enemies
Three types defined in `ENEMY_TYPES` at line 502:
1. **Grunt** — basic enemy, 1 HP, 10 points
2. **Interceptor** — fast, weaving pattern, 1 HP, 15 points
3. **Tank** — slow, 4 HP, 35 points; spawns stage 2+

Enemy behaviors:
- Spawning: `spawnEnemy()` at line 1022, difficulty-scaled spawn rates
- Movement: Vertical descent, optional weave pattern (interceptor)
- Firing: Two patterns — `spread` (fan) or `aimed` (at player)
- Collision detection: `rectsOverlap()` at line 1069

#### Boss System
- **When**: After 18 kills per stage (constant `KILLS_PER_STAGE` = 18 at line 491)
- **Spawn**: `spawnBoss()` at line 1043; teleports in with warning banner
- **Difficulty scaling**: HP multiplied by difficulty factor + boss design multiplier
- **Update**: `updateBoss(dt)` at line 1411 — handles movement, telegraphing, pattern attacks
- **Patterns**: Each boss has 3 attack patterns in `BOSS_PATTERNS` array (line 1372); pattern rotates each attack
- **Boss designs**: 6 bosses in `BOSSES` array (line 508) with unique names, colors, and HP multipliers

#### Bullets & Collision
- **Player bullets**: Stored in `bullets` array; projectiles move up, are culled off-screen
- **Enemy bullets**: Stored in `enemyBullets` array; two movement types — simple velocity or radial (for boss patterns)
- **Collision logic**: Lines 1232–1266 — check bullet-enemy, bullet-boss, enemy-player, enemy bullet-player collisions

### Game Systems

#### Difficulty Settings (line 485)
Three presets: Easy, Normal, Hard. Affect:
- Enemy spawn rates and minimum spawn intervals
- Enemy speed and shoot chance
- Player starting lives (4 vs 3)
- Bombs (3 vs 2)
- Boss HP multiplier (0.8 vs 1.0 vs 1.3)

#### Score & Multiplier Chain
- **Chain system**: Line 1088 — each kill increments `chain` counter and resets `chainTimer` (2.2s)
- **Multiplier**: Increases every 5 kills; capped at 10x (`1 + Math.min(9, Math.floor(chain / 5))`)
- **Break condition**: Chain resets if player takes damage or timer expires
- **Milestone popups**: Every 10 kills triggers "X KILLS!" popup

#### Power-Ups (line 1058)
Three types, dropped randomly on enemy death:
1. **Life** (6%) — restores 1 HP, max = starting lives
2. **Bomb** (8%) — restores 1 bomb, max = difficulty-set limit
3. **Weapon** (20%) — upgrades weapon level; at level 3, swaps to alternate weapon at level 1

#### Bomb (Smart Bomb)
- **Trigger**: X key, Shift, or BOMB button (touch)
- **Effect**: Clears all enemy bullets, destroys all on-screen enemies, damages boss (24 HP)
- **Visual/Audio**: Screen shake, flash, noise burst sound
- **Sound**: Layered: noise burst + 3 sawtooth/sine tones at decreasing pitch

#### Screen Shake & Visual Effects
- **Shake**: `screenShake(magnitude, duration)` at line 727; applies random translation each frame
- **Flash**: Brief white overlay, two animation types (`.hit` for damage, `.bomb` for detonation)
- **Vignette**: Red pulsing border when health = 1 (critical state)
- **Reduced motion**: System preference respected; users can toggle via UI (line 535)

#### Stage Progression
- **Flow**: Spawn enemies → accumulate kills → reach threshold → boss fight → advance stage
- **Boss defeated**: Score 500 points, stage increments, theme changes
- **Themes**: 6 environmental themes (line 493) rotate each stage; affect sky gradients and cloud appearance
- **Announcement**: Stage entry shows banner and announces theme name via ARIA live region

### Audio System (Line 542)
**No external assets** — entirely procedural synthesis:
- **Synth core**: `beep(freq, duration, type, volume, opts)` generates oscillator with optional sweep and filtering
- **Noise**: `noiseBurst(duration, volume, opts)` generates white noise with envelope and filter
- **Effects library**: `sfx` object (line 624) defines all sounds (shoot, explode, bomb, boss telegraph, etc.)
- **Spatial audio**: `panFor(x)` computes stereo panning based on on-screen position
- **Master control**: Mute toggle persists to localStorage; audio context lazy-initialized on first interaction

### Input Handling (Lines 767–861)
- **Keyboard**: Arrow/WASD for movement, Space to fire, X/Shift/P for bomb/pause
- **Mouse**: Position-based steering (priority given to keyboard input)
- **Touch**: Dual-zone layout — left 55% for movement (joystick), right 45% for action buttons (FIRE/BOMB)
- **Joystick**: Virtual on-screen joystick; max displacement = 50px from origin

### Rendering (Line 1520)
- **Canvas**: 480×720 (BASE_W/BASE_H); scales to fit window up to 1.6× (line 757)
- **Rendering order**: Background gradient + sky + clouds → player → bullets → enemies → boss → particles → effects
- **Drawing functions**:
  - `drawPlane(x, y, w, h, color, darkColor, flame)` — reusable plane/ship shape (used for player, enemies)
  - `drawBossShip(b)` — boss-specific multi-part design with rotating engine glow
  - `drawClouds(speed, fillColor, rx, ry)` — parallax scrolling clouds
- **Optimization**: Glowing effects (`shadowBlur`) disabled on lower quality levels (line 1579)

### Game Lifecycle
1. **Start/Reset**: `resetGame()` initializes all state, shows overlay
2. **Playing**: Update→Draw loop; pause toggles overlay
3. **End**: `endGame()` checks for best score, saves to localStorage, shows results overlay
4. **Resume**: "Play Again" restarts from `resetGame()`

### Storage & Persistence (Line 463)
Four keys in `localStorage`:
- `skyfighter.bestScore` — high score
- `skyfighter.muted` — audio mute state
- `skyfighter.difficulty` — selected difficulty
- `skyfighter.reducedMotion` — motion effect preference

## Code Navigation Tips

### Adding a new enemy type
1. Add entry to `ENEMY_TYPES` (line 502)
2. Update spawn logic in `spawnEnemy()` to include new type
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678) if needed

### Modifying boss patterns
- Edit `BOSS_PATTERNS` array (line 1372)
- Each boss has exactly 3 patterns; patterns cycle and rotate
- Firing functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing` (lines 1324–1370)

### Adjusting difficulty scaling
- `DIFFICULTIES` object (line 485): spawn rates, enemy speed, shoot chance, lives, bombs, boss HP
- Adaptive difficulty: `spawnTimer` decreases over time (line 1201: `difficultyRamp`)

### Tuning game feel
- Enemy/bullet speeds: edit spawn functions and `update()` movement code
- Player speed: line 877 (`player.speed = 260`)
- Fire rates: line 1177 (`fireDelay`)
- Sound volumes: `beep()` and `noiseBurst()` `vol` parameter, or `masterGain.gain.value` (line 556)

## Development Notes

- **No build step**: Edit `index.html` directly; refresh browser to test
- **Performance debugging**: Frame times tracked in `frameTimes` array (line 475); quality variable scales visuals
- **Accessibility**: ARIA live region (line 390) announces game state changes
- **Mobile-first CSS**: Uses `clamp()` for fluid scaling; `touch-action: none` and `user-select: none` prevent browser defaults
- **Canvas coordinate system**: (0,0) is top-left; y increases downward
- **Common tweaks**: 
  - Change difficulty spawn base: line 486
  - Adjust boss HP: line 1046 `baseHp`
  - Tweak stage themes: line 493 `THEMES` array
