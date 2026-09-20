# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 file. It runs entirely in the browser with no build tools or external dependencies.

## Getting Started

The game is self-contained in `index.html`. To develop and test:

1. **Run locally**: Open `index.html` in a web browser or use a local server:
   ```bash
   python3 -m http.server 8000
   # Then visit http://localhost:8000
   ```

2. **No build step required**: All code is embedded in the single HTML file.

3. **No tests or linting**: The project uses no test framework or linters. Validation is manual.

## Architecture & Code Organization

The game logic is implemented as a single self-executing function (IIFE) in the `<script>` tag (lines 422-1666). The architecture consists of these key systems:

### Game State Management
- **Player**: Position, weapon, level, lives, invulnerability state
- **Entities**: Arrays of `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, and `boss`
- **Game loop**: Frame counter, score, multiplier chain, stage progression
- **Persistence**: High scores, mute state, difficulty, and motion preferences stored in localStorage

### Input Handling (lines 768-861)
- **Keyboard**: Arrow keys, WASD for movement; Space/click for firing; X/Shift for bombs; P/Esc to pause
- **Mouse**: Movement steered by pointer position when hovering canvas
- **Touch**: Virtual joystick on left, FIRE/BOMB buttons on right for mobile
- Priority system: keyboard/joystick overrides mouse movement

### Rendering System (lines 913-1638)
- **Canvas-based**: All graphics drawn procedurally, no sprite sheets
- **Plane rendering**: `drawPlane()` draws player, enemies, and bosses with gradient fills
- **Boss rendering**: `drawBossShip()` creates unique visual designs per boss type
- **HUD**: Score, best, multiplier, weapon level, hearts, bombs updated via DOM
- **Effects**: Screen shake, flash, vignette pulse, combo popups, stage banners
- **Performance optimization**: `quality` variable (0=full, 1=medium, 2=low) reduces particles and effects based on frame time

### Game Update Loop (lines 1148-1302)
- **Movement**: Player movement with boundary clamping; enemy spawning and patterns
- **Combat**: Bullet-enemy collisions, boss health tracking, weapon leveling
- **Difficulty**: Spawn rates and enemy stats scale based on `difficulty` setting and stage progression
- **Scoring**: Combo chains reset after 2.2 seconds without kills; multiplier increases with kills
- **Stage progression**: 18 kills trigger boss spawn; boss defeat increments stage

### Weapon System
- **Vulcan** (rapid fire): Single or spread shots based on level (1-3)
- **Laser** (piercing): Slower fire rate, penetrates enemies based on level
- **Switching**: Weapon changes via weapon power-ups; high level cycles to alternate weapon

### Boss System (lines 1043-1439)
- **6 Boss types**: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- **6 Difficulty themes**: Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge
- **Attack patterns**: Each boss cycles through unique procedural bullet patterns (radial bursts, aimed spreads, homing missiles, etc.)
- **Telegraph system**: Visual warning before boss attacks; player has brief window to evade

### Audio System (lines 542-676)
- **Web Audio API synthesis**: No external audio files; all sounds procedurally generated
- **SFX layers**: Multi-layer sound effects (e.g., bomb combines noise burst + bass sweep)
- **Spatialization**: Pan effects based on screen position
- **Master gain & compression**: Dynamic range control

### Performance Tracking (lines 474-482)
- **Adaptive quality**: Monitors frame time and automatically scales visual effects
- **Frame time averaging**: 45-frame rolling average; decreases quality if avg > 25ms, increases if < 17.9ms

## Key Constants & Configuration

- **BASE_W / BASE_H** (753): Game canvas size (480×720)
- **DIFFICULTIES** (485-489): Spawn rates, enemy speed, shoot chance, lives, bombs per difficulty
- **ENEMY_TYPES** (502-506): Stats for grunt, interceptor, tank (health, speed, score)
- **BOSSES** (508-515): Design colors and health multipliers for each boss
- **BOSS_PATTERNS** (1372-1409): Attack pattern functions for each boss type
- **THEMES** (493-500): Color schemes for each stage's sky, clouds, and sun

## Common Development Tasks

### Adding a New Weapon Type
1. Add weapon case in `fireWeapon()` (line 1124)
2. Extend weapon logic in `applyPowerUp()` for weapon switching (line 1311)
3. Update `weaponLabel()` display string (line 871)

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` constant (line 502)
2. Add `minStage` property to unlock at appropriate stage
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678)
4. Adjust spawn probabilities in `spawnEnemy()` (line 1022)

### Adding a New Boss
1. Add boss design to `BOSSES` array (line 508) with colors and hpMul
2. Create attack pattern array in `BOSS_PATTERNS` (line 1372)
3. Optionally add stage theme to `THEMES` (line 493)

### Adjusting Difficulty Balance
- Spawn frequency: `DIFFICULTIES[].spawnBase` and `spawnMin` (line 485)
- Enemy stats: `enemySpeed`, `enemyShootChance` in difficulty config
- Boss health: `bossHpMul` in difficulty config
- Player resources: `lives`, `bombs` per difficulty

## Performance Considerations

- **Particle count**: Scaled by `quality` setting to maintain 60 FPS on lower-end devices
- **Canvas size**: Fixed at 480×720; scales via CSS transform for responsive display
- **Collision detection**: Simple AABB rect overlap checks (lines 1069-1071)
- **Rendering**: Glow effects (shadowBlur) disabled on lower quality settings
- **Touch optimization**: `touch-action: none` and `user-select: none` for mobile performance

## Browser Compatibility

- Requires ES6+ (arrow functions, template literals, const/let)
- Web Audio API for sound
- Canvas 2D Context for rendering
- LocalStorage for persistence
- requestAnimationFrame for game loop
- Touch Events API for mobile

## Notes for Future Development

- **No CSS framework**: All styling is inline; keep it self-contained
- **Procedural art**: Graphics generated via canvas paths; changing look requires tweaking draw functions
- **Single file**: Keep all game logic in one HTML file for portability
- **Testing approach**: Manual play-testing across difficulty levels and devices
- **Mobile-first**: Touch controls are primary; keyboard/mouse are secondary fallbacks
