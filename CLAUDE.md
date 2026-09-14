# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML5 Canvas application with no external dependencies. All game logic, rendering, audio synthesis, and UI are self-contained in `index.html`.

## Running the Game

1. **Local Development**: Open `index.html` directly in a web browser (any modern browser with Canvas and Web Audio support).
2. **Web Server**: For development with live reload, use any local HTTP server:
   ```bash
   python3 -m http.server 8000
   # Then visit http://localhost:8000/
   ```
3. **Production**: Deploy the single `index.html` file as-is to any static hosting.

## Game Architecture

The game is organized around several core systems, all contained within a self-executing IIFE in the script section:

### Core Game Loop
- **Main Loop** (`animate()` function ~line 1530): Runs at 60 FPS, calls `update(dt)` for logic and renders via canvas context
- **Update Phase** (`update()` function ~line 1148): Processes player input, enemy spawning, collision detection, and game state
- **Render Phase**: Canvas 2D drawing calls render the sky background, entities, and HUD

### Entity System
- **Player Ship**: `player` object with position, health, weapons, and state
- **Enemies**: Stored in `enemies` array with type (grunt, interceptor, tank), position, HP, and AI behavior
- **Boss**: Single `boss` object with specialized rendering and attack patterns
- **Projectiles**: Separate `playerBullets` and `enemyBullets` arrays
- **Explosions**: Particle effects stored in `explosions` array
- **Power-ups**: Weapon upgrades and health packs in `items` array

### Key Subsystems

**Weapons System** (~line 871-1370)
- Six weapon types: VULCAN, TWIN, SPREAD, SPIRAL, GUIDED, DOUBLE
- Each weapon has different firing patterns implemented as dedicated functions
- Weapons level up when collecting power-ups, affecting damage/spread
- Difficulty affects damage scaling per enemy type

**Difficulty Levels** (~line 485-489)
- Easy/Normal/Hard with different spawn rates, enemy speed, and boss HP
- Stored in localStorage, can be changed from start screen
- Per-difficulty settings: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

**Audio System** (~line 542-620)
- Web Audio API synthesized sounds (no audio files)
- Sound effects: weapon fire, hits, explosions, boss defeat, level complete
- All audio queued through `beep()` and `noiseBurst()` functions with parameters for frequency, duration, envelopes, and panning
- Can be muted; toggles stored in localStorage

**Visual Effects & Accessibility**
- Screen shake on damage (controlled by `screenShake()`)
- Flash on hit/bomb (`flashHit()`, `flashBomb()`)
- Vignette effect during critical health (CSS-driven)
- Reduced motion preference respected (animations disabled when enabled)
- Screen reader support via `announce()` for critical events

**Control Systems** (~line 768-870)
- Keyboard: Arrow keys / WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- Mouse: Move to steer, click to fire
- Touch: Joystick + buttons (left side movement, right side fire/bomb)
- Mobile detection via touch event support

**Scoring & Progression**
- Base score: 10/15/35 points per enemy type
- Multiplier increases on consecutive hits without taking damage
- Chain breaks reset multiplier to 1
- Score persistence: best score stored in localStorage
- Stage progression: 18 kills advances to next stage with new visual theme and boss

### Stage & Boss System
- **Themes** (~line 493-500): Six visual themes (Daylight Front, Sunset Corridor, Night Approach, Storm Front, Arctic Wastes, Volcanic Ridge) cycling through sky gradients and lighting
- **Boss Spawning** (~line 1043-1057): Triggered after KILLS_PER_STAGE enemy kills, one unique boss per stage
- **Six Boss Types** (~line 508-515): VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN with unique colors and HP multipliers
- **Boss Attacks** (~line 1411+): Each boss uses different firing patterns (radial, aimed spread, spiral bursts, etc.)

### Performance Adaptation
- **Adaptive Quality** (~line 473-483): Monitors frame times and automatically reduces quality (enemy particle count, effect rendering) if FPS drops below 40, or increases if above 56
- Three quality levels: 0=full, 1=medium, 2=low

## Code Organization Patterns

**Data-Driven Constants**
- `DIFFICULTIES`, `ENEMY_TYPES`, `THEMES`, `BOSSES` are configuration objects
- New content (bosses, enemies, themes) can be added by extending these objects

**Rendering Functions** (line 913+)
- `drawPlane()`: Renders player/enemy ships as flat geometric shapes
- `drawBossShip()`: Renders boss with multi-part hull, wings, and core
- Projectiles drawn as simple circles/rects

**Event Handlers**
- Keyboard: `keydown`/`keyup` for input state tracking
- Touch: `touchstart`, `touchmove`, `touchend` for joystick + button zones
- Mouse: `mousemove`, `click` + `pointerlock` for aiming
- Resize: `window.resize()` to adapt canvas to viewport

## Modifying the Game

**Adding a New Enemy Type**
1. Add entry to `ENEMY_TYPES` object with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `EXPLODE_SFX` mapping for the new type's explosion sound
3. Enemy spawning logic in `spawnEnemy()` already reads `minStage` to control when enemies appear

**Adding a New Boss**
1. Add to `BOSSES` array with: `name`, `hullTop`, `hullBottom`, `wing`, `core` (hex colors), `hpMul`
2. Implement a new firing pattern function (e.g., `fireNewPattern()`) following existing patterns
3. Add call to that function in `updateBoss()` switch statement

**Adding a New Weapon**
1. Add to weapon configuration (searched for at line 871: `weaponLabel()`)
2. Implement firing pattern function (e.g., `fireCustomPattern()`)
3. Add ammo/cooldown logic to `fireWeapon()` (~line 1124)

**Adding Visual Effects**
- All CSS animations already support reduced motion via `.reduced-motion` class toggle
- Canvas particle effects controlled via quality system; add new particle types in `spawnExplosion()` / `explosions` array

## Storage Keys

All persistent data uses localStorage with prefixes:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute toggle (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion toggle (0/1)

## Testing & Debugging

- **Console**: No external dependencies means easy debugging via browser DevTools
- **Performance**: Check `quality` variable to see if performance scaling kicked in
- **Audio**: Test with mute toggle; audio context may be suspended until user interaction
- **Mobile**: Test with device emulation or on actual device; touch controls activate automatically
- **Accessibility**: Verify reduced motion toggle disables all animations; test with screen reader

## Browser Compatibility

Requires:
- Canvas 2D API
- Web Audio API (graceful degradation if unavailable)
- ES6+ (arrow functions, const/let, template literals)
- localStorage API
- Touch Events API (optional; mouse fallback works without)

Tested on modern Chrome, Firefox, Safari, and Edge.
