# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up arcade game built as a single-file HTML5 application. The entire game (1670+ lines) is contained in `index.html` with no external dependencies: all graphics are canvas-rendered, all audio is synthesized via WebAudio, and all logic is vanilla JavaScript.

## Running the Game

Since this is a single HTML file with no build step, simply open it in a web browser:

```bash
# Open in a browser (no server required)
open index.html
# or
firefox index.html
```

The game runs fully client-side. Local development can be done by editing `index.html` and refreshing the browser.

## Architecture Overview

### Single-File Design

All code lives in `index.html`:
- **Lines 10–358**: CSS styling for UI, canvas, overlays, animations, and responsive layout
- **Lines 422–676**: Audio system (WebAudio synth, sound effect definitions)
- **Lines 680–1670**: Game logic (input, game state, entity updates, rendering, bosses, power-ups)

### Canvas-Based Rendering

- **Canvas Setup** (lines 752–765): Adaptive sizing to fit window while maintaining 480×720 aspect ratio
- **Drawing Functions**:
  - `drawPlane()`: Renders player and enemy aircraft
  - `drawBossShip()`: Boss-specific visuals with animated parts
  - `drawClouds()`: Parallax background layers
  - `draw()`: Main render loop combining all elements
- **Rendering Order**: Sky gradient → Sun glow → Cloud layers → Player → Bullets → Enemies → Boss → Power-ups → Particles

### Game Loop

- **`loop(now)` (lines 1640–1652)**: requestAnimationFrame-driven main loop
  - Caps dt at 50ms (20 FPS minimum)
  - Calls `update(dt)` when running and not paused
  - Calls `draw()` every frame
  - Tracks performance and adapts quality

### Performance Adaptation

- **`trackPerf(dt)` (lines 476–483)**: Monitors last 45 frame times
  - `quality = 0`: Full effects (glow, shadows, more particles)
  - `quality = 1`: Medium (reduced particle count)
  - `quality = 2`: Low (minimal effects, no glow)
- Reduces effects automatically if avg FPS drops below ~40

## Key Game Systems

### Input System

Three input methods (keyboard/mouse take priority over mobile):

**Keyboard & Mouse (lines 768–862)**:
- Arrow keys + WASD: Move
- Space: Fire
- X / Shift: Bomb
- P / Escape: Pause
- Mouse: Steer by moving over canvas, click to fire

**Touch (lines 778–834)**:
- Left 55% of screen: Drag joystick to move
- Right 45% of screen: Two buttons (FIRE and BOMB)
- Joystick rendered at touch origin when active

### Player & Weapon System

**Player State** (line 877):
- Position (x, y), dimensions (40×40)
- Two weapons: `vulcan` (spread shot) and `laser` (piercing)
- Weapon levels 1–3, each with different spread/pierce counts
- Invulnerability timer (after hit: 1.5s)

**Weapon Firing** (lines 1124–1146):
- Vulcan: Up to 3 spread bullets per shot, 150ms cooldown
- Laser: Up to 3 lanes of piercing bullets, 100ms cooldown
- Different firing sounds for each

### Enemy System

**Three Enemy Types** (lines 502–506):
- **Grunt** (red): 1 HP, 10 points, basic stats
- **Interceptor** (orange): 1 HP, 15 points, weaves side-to-side, spawns from stage 1
- **Tank** (purple): 4 HP, 35 points, slower, spawns from stage 2

**Enemy Spawning** (lines 1022–1041):
- Spawn rate increases with difficulty and time
- Random positioning across top of screen
- Two shot patterns: "spread" (3-way fan) or "aimed" (direct at player)

### Boss System

**Six Bosses** (lines 508–515):
1. **VANGUARD**: Radial burst, aimed fan, V-fans
2. **BEHEMOTH**: Heavy columns, homing spread, dense radial
3. **SPECTER**: Teleport dash, tight spiral, double aimed
4. **DREADNOUGHT**: 8-way cross, homing missiles, radial wall
5. **WRAITH**: Teleport barrage, double rings, cross-spreads
6. **TITAN**: Heavy columns, full ring, heavy homing

Bosses spawn after 18 kills per stage and cycle (BOSSES[stage % 6]).

**Boss Attack Patterns** (lines 1324–1409):
- Telegraph phase (22 frames visual warning)
- Fire pattern from phase list
- 85-frame cooldown before next telegraph
- Each pattern is a function that populates `enemyBullets[]`

**Boss Health & Difficulty** (line 1046):
- Base HP: `70 + stage × 35`
- Scaled by difficulty multiplier and boss-specific hpMul
- Updated per-frame in `bossBarFill` style.width

### Difficulty System

**Three Difficulties** (lines 485–489):
- **Easy**: Spawn slower, enemies weaker, 4 lives, 3 bombs, 0.8× boss HP
- **Normal**: Balanced stats, 3 lives, 2 bombs, 1.0× boss HP
- **Hard**: Spawn faster, enemies faster/accurate, 3 lives, 2 bombs, 1.3× boss HP

Persisted in localStorage under key `skyfighter.difficulty`.

### Scoring & Chain Multiplier

**Chain Mechanic** (lines 1087–1101):
- Each kill increments `chain` and resets 2.2s timer
- Multiplier = 1 + floor(chain / 5), capped at 10×
- Chain breaks if 2.2s passes without a kill
- Shows combo pop at every 10-kill milestone

### Power-up System

**Three Types** (lines 1058–1066, 1304–1322):
- **Life**: Restore 1 HP (up to max)
- **Bomb**: Restore 1 bomb (up to max)
- **Weapon**: Upgrade weapon level or swap weapon (cycling between vulcan/laser)

6% spawn rate on enemy death, selected by random roll.

### Audio System

**WebAudio Synth** (lines 542–676):
- No external audio files
- `beep()`: Sine/square/sawtooth oscillator with optional frequency sweep, filter, pan
- `noiseBurst()`: Pink-noise-like burst with filter
- All sounds are procedural one-liners in the `sfx` object

**SFX Categories**:
- **Weapon**: shoot (vulcan), laser (laser weapon)
- **Explosions**: explodeGrunt, explodeInterceptor, explodeTank (by enemy type)
- **Player**: hit (damage taken), bomb (smart bomb detonation)
- **Boss**: bossHit (hit by player), bossTelegraph, bossDown
- **Progression**: powerup, stageClear, gameover, uiClick

Muted state persisted in localStorage.

### Visual Effects

**Screen Shake** (lines 726–730):
- Magnitude and duration set by event (10 for hit, 14 for bomb, 18 for boss death)
- Reduced to 15% intensity if reduced-motion is enabled
- Applied as canvas translate in draw loop

**Flash Effects** (lines 698–709):
- `flashHit()`: Brief white overlay on player damage
- `flashBomb()`: Longer white overlay on bomb detonation
- Skipped entirely in reduced-motion mode

**Vignette** (lines 75–87):
- Red radial vignette that pulses when player at 1 life
- Helps signal critical state without sound

**Particle System** (lines 1073–1085, 1628–1636):
- Spawns on enemy death, bomb detonation, hits
- Simple velocity-based particles that fade and shrink
- Particle count scaled by quality setting

### Collision Detection

**Rectangle Overlap** (lines 1069–1071):
- Used for all entity-entity and entity-player collision
- Center-based: `|dx| < (w1 + w2)/2 && |dy| < (h1 + h2)/2`
- Checked in order: player bullets vs enemies → bullets vs boss → enemies/bullets vs player → power-ups vs player

### UI & Overlay System

**HUD** (lines 367–374):
- Shows score, best score, multiplier, weapon, lives (hearts), bombs (bombs)
- Updated in real-time by `scoreEl`, `multEl`, `weaponLvlEl`, `renderHearts()`, `renderBombs()`

**Overlays** (lines 401–419):
- Start screen: Difficulty selection, controls hint, best score
- Pause screen: Resume button
- Game over: Final score/stage/best, play again

**Dynamic Theme System** (lines 493–500):
- Six themes cycling by stage: Daylight Front → Sunset → Night → Storm → Arctic → Volcanic
- Each theme has sky gradient colors, cloud color, sun color
- Applied to canvas background gradient in draw()

## Persisted State

All saved to localStorage:
- `skyfighter.bestScore`: Best score achieved
- `skyfighter.muted`: Mute toggle (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (0 or 1, respects system prefers-reduced-motion)

## Development Notes

### Adding a New Enemy Type

1. Define in `ENEMY_TYPES` object with stats (w, h, hp, speedMul, color, dark, score, minStage, optional weave)
2. Add explosion sound to `EXPLODE_SFX` map
3. Add spawn condition in `spawnEnemy()` based on `stage` and random roll
4. Implement firing pattern in enemy update loop (already generic: spread vs aimed)

### Adding a New Boss Pattern

1. Define pattern function in `BOSS_PATTERNS` array that calls firing helpers
2. Firing helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
3. Each pattern function receives boss object `b` and populates `enemyBullets[]`
4. Bosses automatically use `BOSS_PATTERNS[designIndex]` when they telegraph

### Adding a New Theme

1. Push new theme object to `THEMES` array with sky gradient colors, cloud/sun color strings, and display name
2. Themes cycle automatically: `THEMES[(stage - 1) % THEMES.length]`

### Gameplay Balance Tuning

- **Difficulty**: DIFFICULTIES object controls spawn rates, enemy stats, lives, bombs, boss HP
- **Enemy spawn rate**: Base (spawnBase, spawnMin) in DIFFICULTIES, also ramps with frame count
- **Boss difficulty**: Base HP = `70 + stage × 35`, multiplied by difficulty and boss hpMul
- **Weapon damage**: Bullets deal 1 HP to enemies; 3 HP to boss (hardcoded, see lines 1237, 1259)
- **Multiplier cap**: Currently 10×; adjust in `addScore()` line 1090

### Common Edits

- Change BASE_W/BASE_H (line 753) to adjust canvas aspect ratio (currently 480×720)
- Modify spawn timer calculation (lines 1201–1202) to tweak enemy density
- Adjust KILLS_PER_STAGE (line 491) to change when bosses appear
- Change player speed (line 877) or weapon cooldowns (lines 1177) for different feel
- Modify particle count scaling (line 1074) if performance is poor on target devices
