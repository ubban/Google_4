# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical scrolling shoot-em-up game implemented as a single self-contained HTML5 file with embedded CSS and JavaScript. The game features:
- Player-controlled fighter jet with multiple weapon types
- Enemy waves with varying types and behaviors
- Boss encounters at stage progression thresholds
- Combo/chain scoring system with multiplier mechanics
- Touch and keyboard/mouse controls
- Procedural sound generation (no audio assets)
- Adaptive performance scaling based on frame times

## Running the Game

- **Play locally**: Open `index.html` directly in a web browser (works on desktop, tablet, mobile)
- **Testing**: Open in Chromium or any modern browser with canvas and WebAudio API support
- No build step, dependencies, or server required

## Code Architecture

The game is a single ~1670 line HTML file organized into logical sections within a self-executing IIFE:

### Game State & Configuration
- **Game state variables** (line ~863): `player`, `bullets`, `enemies`, `boss`, `score`, `lives`, `stage`, etc.
- **Constants** (lines ~485-515):
  - `DIFFICULTIES`: Three difficulty presets controlling spawn rates, enemy behavior, boss HP multipliers
  - `THEMES`: Six visual themes with distinct sky gradients, cloud colors, sun tones
  - `ENEMY_TYPES`: Three enemy types (grunt, interceptor, tank) with stats
  - `BOSSES`: Six boss designs with unique stats and attack patterns
- **Storage keys** (line ~463): Using `localStorage` for best score, mute state, difficulty, reduced motion preference

### Input System (lines 768-861)
- Keyboard input via `keys` object tracking key states
- Touch controls with virtual joystick (left side) and fire/bomb buttons (right side)
- Mouse steering: player glides toward pointer with keyboard/joystick taking priority
- Special keys: Space/click to fire, X/Shift to drop bomb, P/Esc to pause
- `ensureAudio()` called on first user interaction to resume AudioContext (required by browser autoplay policy)

### Audio System (lines 542-676)
- **WebAudio synthesis** instead of external assets
- `beep(freq, duration, type, volume, opts)`: Oscillator with optional sweep, filter, pan
- `noiseBurst()`: Procedurally generated white noise for impact sounds
- `sfx` object maps game events to audio (shoot, laser, explosions by type, hit, bomb, powerup, boss hit, etc.)
- All sounds generated on-the-fly; mutable via `muted` flag with localStorage persistence

### Game Loop & Update (lines 1640-1652, 1148-1302)
- **loop()** (`requestAnimationFrame`) calls `update(dt)` and `draw()` when `running && !paused`
- **update(dt)**:
  1. Player movement (keyboard + joystick + mouse) with bounds checking
  2. Weapon firing cooldown and firing (vulcan vs laser with pierce levels)
  3. Chain/multiplier decay and management
  4. Update all bullet positions and filter off-screen
  5. Spawn enemies and check for stage -> boss transition
  6. Update boss AI and attack patterns
  7. Collision detection: bullets vs enemies, enemies vs player, enemy bullets vs player, powerups vs player
  8. Explosion particles update
- **Collision**: Simple AABB via `rectsOverlap(a, b)`
- **Performance scaling**: `trackPerf()` monitors frame times and adjusts `quality` level (0=full, 1=medium, 2=low) to reduce particle counts and shadow blur on slower devices

### Rendering (lines 1520-1638)
- Canvas clearing, sky gradient background per theme
- Radial gradient sun glow effect
- Parallax clouds (two layers at different speeds)
- Player plane drawing with engine flame when firing
- All bullets and particles
- All enemies (with HP bar for tanks)
- Boss ship with dynamic core glow and wings
- Power-up pickups (rotating, colored)
- Optional mouse target reticle when mousing
- Screen shake applied via canvas translate offset when `shakeTime > 0`

### Drawing Functions
- **drawPlane(x, y, w, h, color, darkColor, flame)**: Draws fighter jet silhouette with optional engine flame; used for player and enemies
- **drawBossShip(b)**: Boss-specific rendering with gradient hull, animated wings, glowing core that pulses when telegraphing attack
- **drawClouds(speed, fillColor, rx, ry)**: Parallax cloud layer using three overlapping ellipses per cloud

### Entity & Damage System
- **Player** (line 877): Position, size, speed, cooldown, rapid-fire, weapon type (vulcan/laser), weapon level (1-3), invuln timer
- **Bullets** (player): Vertical with optional horizontal velocity; laser bullets have pierce count
- **Enemies** (line 1032): Type, position, size, HP, shoot timer, pattern (aimed/spread), weave behavior (for interceptor)
- **Enemy bullets** (line 1219): Support radial and aimed spread patterns; boss bullets use velocity components
- **Boss** (line 1047): Entering state, telegraph animation, attack timer, pattern index cycling
- **Damage**: Bullets deal 1 HP per hit (bosses take 3 per bullet). Enemy bullets break on hit. Lasers pierce multiple enemies.

### Scoring & Progression
- **addScore(base)**: Multiplies score by current multiplier, increments chain (buildup to higher multiplier tiers)
- **Chain system**: Each kill increments chain; breaks if 2.2s elapses without a kill. Multiplier increases every 5 kills (capped at 10x)
- **Stage progression**: 18 kills triggers boss spawn after 1.2s warning. Clearing boss advances stage, resets stage kill counter
- **Power-ups**: Spawn randomly on enemy death (6% life, 8% bomb, 20% weapon); cycle weapon type or upgrade level

### Weapon System
- **Vulcan** (line 1126): Straight bullets with increasing spread (level 1: center, level 2: left+right, level 3: left+center+right)
- **Laser** (line 1135): Straight upward, two lanes at level 2+, three lanes at level 3; pierce count allows hitting multiple enemies/boss
- **Weapon upgrades**: Picking up weapon power-up cycles through vulcan levels 1-3, then switches to laser level 1, etc.

### Boss Attack Patterns (lines 1324-1409)
Six boss designs each with three attack patterns cycling on repeat:
- **Radial bursts**: Bullets spread outward in all directions
- **Aimed spreads**: Fan of bullets toward player with spread angles
- **Teleport attacks**: Boss repositions then attacks (SPECTER, WRAITH)
- **Columns**: Vertical bullet streams
- **Homing**: Bullets track toward player position with randomized aim
- **Double rings**: Two concentric radial patterns at different speeds
- Bosses telegraph attacks (core glows) for ~370ms before firing

### UI Overlay & Pause
- **Start/Game Over overlay** (line 401): Shows game title, difficulty selector, best score, final stats
- **HUD** (line 367): Live score, best score, multiplier, weapon name/level, hearts (lives), bombs
- **Boss health bar** (line 376): Shows during boss fight with boss name
- **Combo pop** (line 381): Floating "+N KILLS!" text at every 10 kills
- **Stage banner** (line 382): Intro text and boss warning
- **Pause overlay**: Accessible via P or pause button; resumes game
- **Accessibility**: Screen reader announcements via `srLive` element (line 390)

### Key State & Settings
- **Difficulty**: Three presets affect spawn rates, enemy stats, boss HP. Persisted to localStorage.
- **Muted**: Audio mute toggle, persisted. Prevents audio setup if muted.
- **Reduced motion**: Toggles animations, screen shake, flash, particle count for accessibility. Respects system `prefers-reduced-motion` OS setting.
- **Best score**: High score persistence across sessions.

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with stats: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Add explosion SFX to `EXPLODE_SFX` map (line 678) if unique
3. Optional: Create unique explosion sound in `sfx` object
4. Enemy spawn logic in `spawnEnemy()` filters by `minStage` and weighted probability

### Adding a New Boss
1. Add entry to `BOSSES` array (line 508) with design colors: `hullTop`, `hullBottom`, `wing`, `core`, and `hpMul` multiplier
2. Add attack pattern array to `BOSS_PATTERNS` (line 1372) with three pattern functions
3. Boss index maps to both arrays automatically; stage 1 = BOSSES[0], stage 7 wraps to BOSSES[0] again
4. Each pattern function receives boss object and calls helper functions (`fireRadial`, `fireAimedSpread`, etc.)

### Tuning Difficulty
Edit `DIFFICULTIES` object (line 485):
- `spawnBase`/`spawnMin`: Control enemy spawn rate
- `enemySpeed`/`enemyShootChance`: Affect enemy behavior
- `lives`/`bombs`: Starting values
- `bossHpMul`: Multiplier on boss base HP

### Modifying Canvas Size or Aspect Ratio
- `BASE_W` and `BASE_H` (line 753) define logical game canvas size
- `resize()` function scales to fit window while respecting aspect ratio
- All drawing operations use logical canvas coordinates; scaling handled by CSS

### Visual Tweaks
- **Theme colors**: Edit sky gradients, cloud colors, sun tones in `THEMES` (line 493)
- **Enemy appearance**: Colors in `ENEMY_TYPES`, plus `drawPlane()` function
- **Boss appearance**: `drawBossShip()` function; colors pulled from boss design
- **Particle effects**: `spawnExplosion()` creates colored particle bursts

## Important Implementation Details

### Frame-Independent Movement
- Delta time `dt` (in seconds) multiplied by speed constants to ensure smooth motion regardless of frame rate
- Capped at 0.05s max delta to prevent large jumps on frame stutters

### Touch Control Priority
- Virtual joystick input takes priority over mouse when active
- Joystick base follows touch start position; stick constrained to 50px radius

### Enemy Weaving
- `weave` flag on enemy type enables sinusoidal left-right movement
- Calculated from `weaveSeed` random value per enemy for variation in phase

### Screen Shake
- Applied via canvas context translate; proportional to damage (loseLife = 10, bomb = 14, boss kill = 18)
- Reduced by 85% when reduced-motion is enabled

### Invulnerability
- Player gets 1.5s invuln window after taking damage; blinks every 4 frames
- Prevents frame-perfect collision stacking

### Performance Adaptation
- Last 45 frame times tracked; if average > 1/40s, quality decreases
- If average < 1/56s consistently, quality increases
- Affects: particle counts (scales 1.0x, 0.6x, 0.35x), shadow blur on bullets/boss, sun glow

## Testing

Since this is a game, manual testing in the browser is primary:
1. **Desktop**: Test keyboard (arrows, WASD, Space, X) and mouse steering
2. **Mobile/Tablet**: Test touch controls (joystick + buttons), landscape/portrait orientation
3. **Reduced motion**: Toggle in-game and verify animations disabled
4. **Audio**: Verify mute/unmute works and accessibility announcements play (screen reader)
5. **Difficulty modes**: Verify enemy/boss behavior changes with Easy/Normal/Hard
6. **Stage progression**: Confirm boss appears after 18 kills, stage counter increments
7. **Scoring**: Verify combo multiplier increments and chain timeout resets score multiplier to 1x
