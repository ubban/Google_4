# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5 (single-file implementation). The game features:
- Player-controlled fighter with multiple weapon types (Vulcan, Rapid, Laser)
- Three enemy types with varied behavior patterns
- Six boss encounters with unique attack patterns
- Weapon power-ups and smart bomb mechanics
- Score multiplier chain system
- Difficulty levels (Easy, Normal, Hard)
- WebAudio API synthesis for sound effects
- Touch and keyboard/mouse controls
- Particle effects and screen shake

The entire game implementation resides in `index.html` (~1,669 lines), organized as a single IIFE with modular internal functions.

## Architecture & Key Systems

### Game State & Entities
All game entities are plain objects managed in arrays, initialized in `resetGame()`:
- **player** — Fighter object with position, weapon, level, invulnerability timer
- **enemies** — Array of enemy objects (grunt, interceptor, tank types)
- **boss** — Current boss object (if active), or null
- **bullets** — Array of player projectiles (piercing laser or vulcan shots)
- **enemyBullets** — Array of enemy projectiles
- **particles** — Array of explosion/visual effect particles
- **powerUps** — Array of collectible power-ups

Each entity is a flat object with properties like `{x, y, w, h, speed, hp, ...}` — no classes.

### Game Loop
The main loop (`loop()`) runs via `requestAnimationFrame`, called every frame:
1. Calculate delta time (dt, clamped to max 50ms)
2. Track performance for adaptive quality adjustment
3. If running and not paused: call `update(dt)` then `draw()`

### Update Phase (`update(dt)`)
Processes per-frame logic:
- **Input handling** — Read keyboard (`keys` object), joystick (`joyVec`), mouse (`mouseTarget`)
- **Player movement** — WASD/arrows/mouse tracking, clamped to canvas
- **Firing** — Space/click triggers weapon based on cooldown
- **Bullet movement** — Move all bullets and filter off-canvas ones
- **Enemy movement** — Advance enemies downward; weaving interceptors use sine wave
- **Spawning** — Spawn regular enemies by timer; trigger boss after KILLS_PER_STAGE (18) kills
- **Collision detection** — AABB overlap checks between bullets/enemies, enemy bullets/player, boss/player, player/power-ups
- **Destruction** — Remove dead enemies/boss, trigger explosions and sound
- **Chain tracking** — Decay multiplier chain timer; reset on missed kills
- **Boss AI** — Call `updateBoss(dt)` if active

### Render Phase (`draw()`)
Canvas rendering in order:
1. Clear and apply sky gradient
2. Render clouds (parallax effect)
3. Draw all particles (explosions)
4. Draw player ship (`drawPlane()`)
5. Draw enemies (colored planes via `drawPlane()` or boss via `drawBossShip()`)
6. Draw all bullets and enemy bullets
7. Draw power-ups
8. Apply screen shake offset (if active)
9. Apply red vignette (if player health critical)

### Collision & Entity Systems
- **AABB overlap** — Simple bounding-box check in `rectsOverlap(a, b)`
- **Particles** — Temporary visual effects spawned by `spawnExplosion()`, auto-removed after fade
- **Explosions** — Burst of particles in a given color; no collision damage
- **Power-ups** — Random drop on enemy death; `applyPowerUp(type)` upgrades weapon or restores health/bombs

### Audio System
WebAudio API synthesis (no external audio files):
- **`ensureAudio()`** — Lazy-initialize AudioContext on first user interaction
- **`beep(freq, dur, type, vol, opts)`** — Generate tones with optional sweep, filter, panning, delay
- **`noiseBurst(dur, vol, opts)`** — Generate filtered noise bursts
- **`sfx` object** — Pre-defined sound effects (shoot, explode, bomb, boss hit, etc.)
- Sound output routed through compressor to masterGain

### Input Handling
- **Keyboard** — Stored in `keys` object updated by keydown/keyup events
- **Mouse** — Click fires; movement steers if mouse is over canvas
- **Touch** — Two-zone controls: left side for movement (joystick), right side for fire/bomb buttons
- **Joystick** — Virtual joystick appears on touch; `joyVec` normalized to (-1, 1)

## Key Constants & Configuration

**Difficulty Settings** (`DIFFICULTIES`) — Spawn rates, enemy speed, shoot chance, starting lives/bombs, boss HP multiplier per difficulty.

**Enemy Types** (`ENEMY_TYPES`) — Three types with distinct stats:
- grunt: 1 HP, slow, low score
- interceptor: 1 HP, fast, weaves, medium score
- tank: 4 HP, slow, heavy score

**Boss Types** (`BOSSES`) — Six boss designs with unique colors and HP multipliers. Each boss fires unique patterns (`fireRadial`, `fireAimedSpread`, `fireSpiralBurst`, etc.).

**Weapon Types** — Two weapons accessed by `player.weapon`:
- vulcan: Fires single shot; rapid-fire with sustained button hold
- laser: 0.1s cooldown; pierces up to 2 enemies per shot

**Themes** (`THEMES`) — Six sky color schemes and cloud colors.

**Power-ups** — Three types randomly dropped:
- health: Restore 1 life (capped at maxLives)
- weapon: Upgrade weapon level (1–3)
- bomb: Restore 1 bomb (capped at maxBombs)

## Common Development Tasks

### Running the Game
1. Open `index.html` in a browser (any modern browser with Canvas & WebAudio support)
2. Select difficulty (Easy/Normal/Hard)
3. Click "Start Game" or press Start button

### Testing a Single Feature
- **New enemy type**: Add to `ENEMY_TYPES`, spawn in `spawnEnemy()`, add explosion sound to `EXPLODE_SFX`
- **New weapon pattern**: Add firing function (e.g., `fireWave()`) and call it from `fireWeapon()` or boss attack logic
- **New boss**: Add to `BOSSES` array, define attack pattern in `updateBoss()`
- **Visual effect**: Add particle spawns to `spawnExplosion()` or create new effect via canvas drawing

### Performance Optimization
- Quality adjustment happens automatically in `trackPerf()` if average frame time degrades
- Screen shake and animations respect `reducedMotion` setting for accessibility
- Enemy count is capped by spawn timer and difficulty settings; no dynamic culling needed yet

### Collision Debugging
- `rectsOverlap(a, b)` checks all pairs in collision phase — straightforward AABB
- To debug, add console logs in update phase collision loops
- Ensure all moving entities have `w` and `h` properties set

### Audio Debugging
- Mute button toggles `muted` flag (persists in localStorage)
- Audio context only initializes on first user interaction (browser policy)
- Use browser DevTools to inspect AudioContext state if synthesis fails

## File Structure

```
index.html (1,669 lines)
├── HTML head: meta, favicon, styles (CSS-in-file)
├── HTML body: canvas, HUD elements, overlay screens, touch controls
└── JavaScript (IIFE)
    ├── DOM references & state variables
    ├── Configuration constants (DIFFICULTIES, ENEMY_TYPES, BOSSES, THEMES)
    ├── Audio system (WebAudio synthesis)
    ├── Input handling (keyboard, mouse, touch, joystick)
    ├── Rendering (sky, clouds, entities, particles)
    ├── Game logic (update, collision, entity spawning)
    ├── Entity drawing (planes, bosses, explosions)
    └── Main loop (requestAnimationFrame)
```

## Persistence & Storage

Game state persists in localStorage:
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Reduced motion preference

## Browser Compatibility

- Requires: Canvas 2D, WebAudio API, requestAnimationFrame
- Tested: Modern Chrome, Firefox, Safari, Edge
- Mobile: Touch controls active on touch devices; joystick shows on touch start
- Accessibility: Full keyboard/gamepad alternative for touch, ARIA live region for screen readers, reduced-motion support
