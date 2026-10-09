# Sky Fighter — Raiden-Style Vertical Shoot-'em-Up

## Overview

**Sky Fighter** is a browser-based arcade-style vertical shooter (shmup) inspired by classic Raiden games. It's a single-file HTML application (`index.html`) with embedded CSS and JavaScript that requires no external assets or build tools.

- **Player Goal**: Survive 6 stages by defeating increasingly difficult enemies and bosses
- **Mechanics**: Weapon switching, smart bombs, score multipliers, progressive difficulty
- **Audio**: Fully synthesized using Web Audio API (no external audio files)
- **Storage**: Persistent high scores, difficulty preference, and accessibility settings via localStorage

## Project Structure

### File: `index.html` (~1,670 lines)
Single self-contained HTML file with three main sections:
1. **HTML body**: Canvas element (#gameCanvas), HUD display, controls UI
2. **CSS**: Embedded styles for responsive layout, animations, dark theme
3. **JavaScript**: Complete game engine in one `<script>` block

### Key Constants & Configuration

**Game Constants** (lines ~150-530):
- `CANVAS_W, CANVAS_H`: 360×640px viewport (optimized for mobile/portrait)
- `STORAGE_*`: localStorage keys for persistence (best score, mute state, difficulty, reduced motion)
- `DIFFICULTIES`: Object defining `normal`, `hard`, and `brutal` difficulty parameters
  - Controls: enemy spawn rate, shoot frequency, boss HP multipliers, bomb cooldown
- `ENEMY_TYPES`: Three enemy classes with stats (HP, speed, appearance, score values)
  - `grunt`: 1 HP, red, slow
  - `interceptor`: 1 HP, orange, fast, weaving pattern
  - `tank`: 4 HP, purple, slow, high value
- `BOSSES`: Array of 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
  - Each boss has color scheme (hullTop, hullBottom, wing, core) and HP multiplier

**Game State** (lines ~540-800):
- `running, paused, gameOver`: Game loop control
- `player`: Player ship object (position, speed, weapon, level, invulnerability, cooldown)
- `enemies, bullets, enemyBullets, powerUps, particles`: Dynamic game objects
- `boss`: Active boss or null
- `stage, score, chain, chainTimer, multiplier`: Progression tracking
- `quality`: Graphics quality level (0=high, 1=medium, 2=low) based on performance
- `difficulty, muted, reducedMotion`: Player settings

## Core Systems

### Player Ship (`player` object)
- **Position**: `x, y` coordinates (center of ship)
- **Dimensions**: 24×32 px
- **Speed**: 280 px/s movement
- **Weapons**:
  - `vulcan`: Machine gun with spread pattern (depends on level)
  - `laser`: Piercing laser beam with multi-lane capability
- **Level**: 1–3, affects weapon spread and projectile properties
- **Invulnerability**: Briefly invincible after hit (`invuln` timer)
- **Cooldown**: Prevents firing too fast (`cooldown` timer)

**Weapon Details**:
- Vulcan (line 1126-1134):
  - Level 1: Single center shot
  - Level 2: Two-lane spread (±0.12 radians)
  - Level 3: Three-lane spread (±0.24, center)
  - Fire rate: 0.15s
- Laser (line 1136-1145):
  - Level 1: Single center beam
  - Level 2: Dual beams (±6px offset)
  - Level 3: Dual beams + center = 3 lanes with pierce=2
  - Fire rate: 0.1s
  - Pierces through `pierce` count of enemies

### Enemies
- **Spawning** (`spawnEnemy`, line 1022-1041):
  - Random X position, enter from top
  - Type distribution: 15% tanks, 25% interceptors (if stage allows), rest grunts
  - Weaving interceptors add sine oscillation to horizontal movement
  - Pattern: 15% spread shots, 85% aimed shots
- **Behavior**:
  - Downward movement at type-specific speeds
  - Shooting at `shootTimer` intervals
  - Aimed shots calculate angle toward player; spread shots fire in cone
  - Destroyed enemies spawn power-ups (6% life, 8% bombs, 20% weapons)

### Bosses
- **Structure** (lines 960-1020): Multi-part rendering with hull, wings, core
- **Spawning** (`spawnBoss`, line 1043-1056):
  - 6 distinct designs per stage (stages loop through 6 bosses)
  - Base HP: 70 + stage×35, scaled by difficulty and boss multiplier
  - Telegraphed attacks with visual warning (pulsing glow)
- **Patterns** (lines 1235-1290+):
  - Multiple attack phases based on `patternIndex`
  - Circle shots, spread patterns, aimed bursts
  - Boss movement (horizontal bouncing, vertical entering/exiting)

### Audio System (Web Audio Synthesis)
**No external audio files—all sound generated procedurally.**

**Synth Functions** (lines 542-623):
- `beep(freq, dur, type, vol, opts)`: Sine/square/sawtooth oscillator with optional sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: White noise with decay and filter
- `panFor(x)`: Converts screen X coordinate to stereo panning value

**Sound Effects** (`sfx` object, lines 624-682):
- `shoot()`: 760→480 Hz square sweep with lowpass filter
- `laser()`: 1500→2400 Hz sawtooth upsweep with highpass filter
- `explodeGrunt()`: Low-pitched sawtooth + noise for small enemy
- `explodeInterceptor()`: Mid-pitch square bursts
- `explodeTank()`: Low rumble + noise for heavy enemy
- `hit()`: Chord-like triangle/sawtooth for player damage
- `bomb()`: Noise burst with highpass filter sweep
- `powerUp()`, `chainUp()`, `stageAlert()`: UI feedback sounds

### Rendering Pipeline
- **Main render function** (line 1410+): Canvas clear, draw particles/enemies/player/UI
- **Quality levels** (line 1314-1326):
  - Level 0: Full particle effects, shadows, smooth rendering
  - Level 1: Fewer particles (60%), light shadows
  - Level 2: Minimal particles (35%), no shadows
- **HUD display** (lines 263-295): Score, high score, lives, bombs, weapon, multiplier
  - Hearts emoji (❤️) for lives, with critical health animation
  - Ammo-style bomb display
- **Screen effects**:
  - `flashHit()`: White flash on player damage (0.25s)
  - `flashBomb()`: Longer flash for bomb (0.5s)
  - `screenShake()`: Canvas offset for impact feedback
  - `vignette`: Red pulsing edge effect for low health

### Collision Detection
- **`rectsOverlap(a, b)`** (line 1069-1071): AABB collision using distance from centers
- **Player ↔ Enemy bullets**: Damage and invulnerability
- **Player ↔ Power-ups**: Pickup and effect application
- **Bullets ↔ Enemies**: Damage and deduction of enemy HP; spawns explosions
- **Bullets ↔ Boss**: Cumulative damage; triggers telegraph state

### Game Loop & Timing
- **Update cycle** (`update` function, lines 1148-1400):
  - Player movement (keyboard + mouse target OR gamepad)
  - Weapon firing based on held fire input
  - Enemy and bullet movement
  - Collision checks
  - Score & chain decay
  - Boss state transitions
- **Render cycle** (line 1408+): Clears canvas, draws all entities
- **Frame rate**: RAF-based; `dt` (delta time) passed for frame-independent logic
- **Performance monitoring**: `quality` auto-reduces if frame rate drops

### Input Handling
1. **Keyboard**: Arrow keys or WASD for movement, Space for fire
2. **Mouse**: Click-to-move targeting (when `mouseActive` true)
3. **Gamepad**: Stick movement + button fire, with dead-zone filtering
4. **Touch**: Drag to aim (sets `mouseTarget`), touch fire buttons on screen

### Persistence & Settings
**localStorage Keys** (lines ~155-160):
- `STORAGE_BEST`: High score
- `STORAGE_MUTE`: Audio toggle
- `STORAGE_DIFF`: Selected difficulty (normal/hard/brutal)
- `STORAGE_REDUCED`: Reduced motion preference

**Accessibility**:
- Respects `prefers-reduced-motion` media query
- Disables animations and screen shake when reduced motion enabled
- Readable text contrast and emoji-based icon feedback

## Game Flow

1. **Menu State** (`!running`): Player selects difficulty, sees high score, mute/motion controls
2. **Stage Start**: Banner announcement, boss warning, enemy spawning begins
3. **Gameplay Loop**:
   - Enemies spawn at increasing rate as round progresses
   - Player destroys enemies, builds score multiplier chain
   - Boss emerges at threshold enemy count
   - Boss defeated → Stage incremented, next boss spawns
4. **Player Death**:
   - Lose one life, invulnerability period
   - Game over at 0 lives → return to menu
5. **Score Display**: Real-time HUD with multiplier (1–10x)

## Key Algorithms & Techniques

### Chain Multiplier System
- Kill enemy → `chain++`
- Multiplier = 1 + floor(chain / 5), capped at 10x
- Chain resets if no kills for 2.2 seconds
- Visual feedback: multiplier badge pulses and updates in real-time

### Boss Telegraph Mechanic
- Boss enters "telegraphing" state before attack
- Visual: pulsing core, reduced opacity
- Gives player predictable warning windows
- Matches arcade shmup tradition of fair telegraphed patterns

### Weaving Enemy Movement
- Interceptor uses `baseX + Math.sin(weaveSeed + y/50) * 40` for horizontal offset
- Creates predictable but non-linear target path

### Particle System
- Simple kinetic particles for explosions
- Velocity, position, life counter
- Color tint for explosion type
- Culled by quality setting during performance throttle

### Power-Up Mechanics
- `weapon`: Increments `player.level` up to 3
- `bomb`: Adds to `bombCount` (max `maxBombs`)
- `life`: Heals player (increments `player.lives`)
- Random spawn probability on enemy death

## Development & Modification Guide

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with `{ w, h, hp, speedMul, color, dark, score, minStage }`
2. Add rendering logic in render section (see lines 920-950)
3. Enemy automatically spawns on appropriate stages

### Adjusting Difficulty
Edit `DIFFICULTIES` object (lines ~175-200) parameters:
- `enemySpeed`: Base pixel/sec movement
- `enemyShootChance`: 0–1 probability
- `spawnRate`: Smaller = faster spawns
- `bossHpMul`: Multiplier for boss health
- `bombCooldown`: Seconds between bomb uses

### Adding Boss Attacks
1. Create new pattern in `spawnBoss` or attack patterns section
2. Add to `patternIndex` branching logic (lines ~1240-1290)
3. Set `telegraph` property to enable warning

### Tuning Audio
Sound parameters in `sfx` object (lines 624+):
- `freq`: Starting frequency in Hz
- `dur`: Duration in seconds
- `vol`: Volume 0–1
- `sweepTo`: Target frequency for frequency ramp
- `filterFreq`: Cutoff frequency for resonance
- `pan`: Stereo position -1 to +1

### Optimization Tips
- Quality auto-downgrades; manually force with `quality = 0/1/2`
- Particle count scales by quality setting
- Canvas size (360×640) chosen for performance on mobile
- All rendering is 2D canvas (no WebGL)

## Known Limitations & Design Choices

1. **Single-file deployment**: No build tool needed; easy to host or fork
2. **No persistence of progression**: Only high score saved (intentional arcade feel)
3. **Mobile-optimized aspect ratio**: 360×640 may be unconventional on desktop
4. **Quality auto-throttle**: Reduces visual fidelity if frame rate drops below target
5. **No network features**: Fully offline, no leaderboards
6. **Synth-only audio**: Procedural means smaller download, but limits audio design space

## Future Enhancement Ideas

- Additional boss patterns (phase 2 mechanics, cooperative attacks)
- Weapon variety (spread shot, homing missiles, shield)
- Endless mode vs. stage-based progression
- Particle system overhaul (pooling, GPU acceleration)
- Mobile touch controls refinement
- Replay saving & playback
