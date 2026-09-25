# Sky Fighter - Codebase Documentation

## Overview
**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file. It's a browser-based arcade game with keyboard/mouse/touch controls, multiple enemy types, boss battles, and procedural sound effects using the Web Audio API.

## Project Structure
- `index.html` - Complete game implementation (single-file format)
- `README.md` - Basic project metadata
- `.git/` - Git repository

## Key Game Features

### Core Mechanics
- **Player Ship**: Centrally controlled aircraft that shoots upward automatically
- **Weapons**: 
  - Vulcan (spread shots, 3 levels)
  - Laser (piercing beam, 3 levels)
  - Weapon switching via power-ups
- **Smart Bombs**: Limited-use screen-clearing attacks (configurable per difficulty)
- **Lives System**: Health-based survival with visual heart indicators
- **Score Multiplier**: Chain-based combo system (improves with consecutive kills)

### Enemy Types
Located in `ENEMY_TYPES` object (~line 502):
- **Grunt** (red): Basic small enemy, score 10
- **Interceptor** (orange): Fast, weaving pattern, score 15, appears from stage 1
- **Tank** (purple): Slow, heavily armored (4 HP), score 35, appears from stage 2

### Boss System
Located in `BOSSES` array (~line 508):
- 6 unique boss designs: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each boss has distinctive attack patterns defined in `BOSS_PATTERNS` (~line 1372)
- Boss patterns include: radial bursts, aimed spreads, teleporting, columns, spirals, homing missiles
- Telegraphing system: Boss flashes before attacking

### Difficulty Settings
In `DIFFICULTIES` object (~line 485):
- **Easy**: Slower spawns, weaker enemies, 4 lives, 1.0x boss HP
- **Normal**: Balanced difficulty, 3 lives, 1.0x boss HP (default)
- **Hard**: Fast spawns, strong enemies, 3 lives, 1.3x boss HP

## Code Architecture

### Game State Variables (~line 863-868)
- `player`: Player object with position, weapon, health
- `bullets`, `enemies`, `enemyBullets`, `particles`: Entity arrays
- `score`, `lives`, `stage`, `chain`, `multiplier`: Game metrics
- `running`, `paused`, `gameOver`: State flags
- `boss`: Boss entity (null when not active)

### Main Game Loop
`loop(now)` function (~line 1640):
1. Calculate delta time
2. Track performance (adaptive quality)
3. Update game state if running
4. Draw frame
5. Request next animation frame

### Update Logic (~line 1148)
1. Player movement (keyboard/joystick priority, mouse fallback)
2. Firing weapons and cooldown
3. Bullet/enemy updates and cleanup
4. Enemy spawning and difficulty ramping
5. Collision detection (enemy-bullet, player-enemy, player-bomb, power-ups)
6. Combo chain management

### Rendering
- Canvas is 480×720 (BASE_W, BASE_H)
- Adaptive scaling based on window size (up to 1.6x)
- Dynamic background with gradient sky, sun, and cloud layers
- Different visual themes cycle per stage (6 themes in `THEMES` array)

## Control Schemes

### Keyboard
- **Arrow Keys / WASD**: Move player
- **Space**: Fire weapon
- **X / Shift**: Drop bomb
- **P / Esc**: Pause
- **Difficulty buttons**: Selectable at start screen

### Mouse
- **Movement**: Hover over canvas to steer toward cursor
- **Click**: Fire weapon

### Touch (Mobile)
- **Left 55% of screen**: Drag to move (virtual joystick)
- **Right 45% of screen**: 
  - Red FIRE button (bottom-right)
  - Yellow BOMB button (top-right)

## Audio System (~line 542-676)
- **Web Audio API**: No external audio files, entirely synthesized
- Sound effects for: shooting, explosions, hits, bombs, power-ups, UI
- Mono/stereo with spatial panning
- Adaptive compression and gain management
- Mute toggle with localStorage persistence
- Enemy-specific explosion sounds: `explodeGrunt`, `explodeInterceptor`, `explodeTank`

## Visual Effects
- **Screen Shake**: Magnitude-based intensity (reduced in reduced-motion mode)
- **Flash**: Brief white flash on hits/bombs
- **Vignette**: Red pulsing edge indicator when health critical (1 life)
- **Combo Pop**: Text animation displaying kill milestones
- **Stage Banner**: Large animated text on stage start/boss warning
- **Particle Explosions**: Colorful burst effects on enemy/boss destruction
- **Reduced Motion Mode**: Disables animations for accessibility (respects system preference)

## Performance Optimizations
- **Adaptive Quality** (~line 474-483):
  - Level 0: Full effects (glow, shadows, many particles)
  - Level 1: Reduced particles (~60% of normal)
  - Level 2: Minimal effects (~35% of normal)
  - Auto-scales based on frame time (target 40-56 FPS)
- Enemy/bullet culling: Remove off-screen entities
- Particle count limited by quality level

## Data Persistence
LocalStorage keys:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Motion reduction preference

## Power-Up System (~line 1058)
Randomly dropped from defeated enemies:
- **Life** (6% chance): +1 max health (capped at maxLives)
- **Bomb** (8% chance): +1 bomb (capped at maxBombs)
- **Weapon** (20% chance): Level up weapon or switch weapons

## Progression
- **Stages**: Triggered after 18 enemy kills (KILLS_PER_STAGE)
- **Boss spawning**: After stage kills threshold, boss enters with warning banner
- **Theme cycling**: Background changes every stage (6 unique themes)
- **Difficulty ramping**: Enemy spawn rate decreases as frame count increases
- **Stage limit**: Cycles through bosses endlessly (stage % 6)

## Accessibility Features
- **Screen Reader Support**: `srLive` element with aria-live region
- **Reduced Motion**: Respects prefers-reduced-motion media query
- **Keyboard Navigation**: Full keyboard control without mouse
- **High Contrast**: Dark backgrounds with bright accent colors
- **Safe Area Insets**: Mobile notch/safe area support via CSS env()

## Modification Guide

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (~line 502)
2. Set `minStage` to control when it appears
3. Create spawn chance condition in `spawnEnemy()` (~line 1022)
4. Add explosion sound to `EXPLODE_SFX` (~line 678) if needed

### Adding a New Boss
1. Add entry to `BOSSES` array (~line 508)
2. Create attack pattern function in `BOSS_PATTERNS` (~line 1372)
3. Boss index determines which pattern set is used
4. Adjust `hpMul` for difficulty scaling

### Tweaking Game Balance
- `DIFFICULTIES`: Adjust spawn rates, enemy speed, lives, bomb count
- `KILLS_PER_STAGE`: Controls boss encounter frequency
- Weapon spread angles in `fireWeapon()` (~line 1124)
- Enemy bullet speeds and patterns in boss pattern functions
- Power-up drop rates in `maybeSpawnPowerUp()` (~line 1058)

### Adding Visual Themes
1. Add entry to `THEMES` array (~line 493)
2. Provide `sky` (3-color gradient), `cloud`, `sun` RGB colors
3. Give it a descriptive `name`

## Performance Considerations
- Single-file design minimizes HTTP requests
- Canvas rendering optimized for 480×720 resolution
- Particle count auto-scales with frame rate
- Enemy and bullet pooling via array manipulation (no object allocation overhead)
- Screen shake intentionally reduced in reduced-motion mode
- Audio context lazily initialized on first interaction

## Known Limitations
- Single HTML file (no build system)
- No persistence between sessions except best score
- Boss patterns are predetermined, not AI-driven
- Touch controls fixed to screen proportions (55/45 split)
- Quality setting auto-detects and cannot be manually overridden by user

## Common Debugging
- Open DevTools Console (`F12` → Console)
- Game logs are minimal (uses announce() for accessibility)
- Frame counter visible via `frame` variable
- Check `running`, `paused`, `gameOver` state flags
- Performance profile via performance.now() delta calculations

---
Last updated: 2026-09-25
