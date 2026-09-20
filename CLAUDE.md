# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built as a single HTML file (`index.html`). The game runs directly in the browser with no build step, dependencies, or external assets. It features:

- Player-controlled airplane with two weapon types (Vulcan and Laser)
- Three enemy types with distinct behaviors
- Six procedurally-designed boss encounters with unique attack patterns
- Combo system with score multipliers
- Three difficulty levels with progressive challenge scaling
- WebAudio synthesis for all sound effects (no pre-recorded audio)
- Full touch and keyboard controls with adaptive performance scaling

## Architecture Overview

### Single-File Structure

All code (HTML, CSS, JavaScript) lives in `index.html` (1669 lines):
- **Lines 1–360**: HTML markup + inline CSS styles
- **Lines 422–1669**: Game logic in a single IIFE (Immediately Invoked Function Expression)

### Major Game Systems

1. **Game Loop** (lines ~1100–1300)
   - Main `update(dt)` function called by `requestAnimationFrame`
   - Handles player input, entity updates, collision detection, and state transitions

2. **Player Ship** (lines ~875–912, ~1100)
   - Position, velocity, weapon state (vulcan/laser), level, invulnerability timer
   - Weapons have cooldown, rapid-fire buildup, and pierce/spread mechanics
   - Health represented as lives; collision with enemies or bullets triggers invulnerability

3. **Enemy System** (lines ~502–506, ~1200–1228)
   - Three enemy types: `grunt` (fast), `interceptor` (agile/weaving), `tank` (tanky)
   - Each type has unique visuals, hp, speed, score value
   - Enemies spawn with configurable rates, shoot with pattern or homing behaviors
   - Removed from game when off-screen

4. **Boss System** (lines ~508–515, ~1411–1449, ~1372–1409)
   - Six boss types with unique color schemes and attack patterns
   - Boss patterns defined in `BOSS_PATTERNS` array (one pattern array per boss)
   - Each pattern fires bullets in different shapes: radial bursts, aimed spreads, double rings, homing shots
   - Boss states: `entering` (animation), `telegraph` (warning flash), attacking
   - Boss health = base * difficulty multiplier

5. **Particle System** (lines ~1296–1297, particle spawning scattered throughout)
   - Lightweight particles for visual effects (explosions, pickups, etc.)
   - Each particle has position, velocity, lifetime; removed when expired

6. **Collision Detection** (lines ~1232–1295)
   - `rectsOverlap()` function checks AABB collisions
   - Player vs. enemies, player vs. enemy bullets, player vs. pickups, bullets vs. boss, all checked in `update()`

7. **Difficulty System** (lines ~485–489)
   - Easy, Normal, Hard: control spawn rate, enemy speed, shoot frequency, starting lives/bombs, boss health
   - Difficulty can be changed before a game starts; persists in localStorage

8. **Audio System** (lines ~542–676)
   - WebAudio synth: oscillators, filters, gain envelopes, panning
   - 12 SFX functions: `shoot`, `laser`, `hit`, `bomb`, `powerup`, `bossHit`, etc.
   - Dynamic panning based on screen position
   - Muted state persists in localStorage

### Game State Variables

Core state (lines ~863–869):
- `player`: Ship object with x, y, w, h, speed, cooldown, weapon, level, invuln timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Arrays of entities
- `boss`: Currently active boss or null
- `score`, `lives`, `stage`, `stageKills`: Game progress
- `chain`, `multiplier`: Combo system (kills build chain; timer resets on miss)
- `frame`, `running`, `paused`, `gameOver`: Game state flags

## Key Mechanics

### Weapon System
- **Vulcan** (default): Rapid-fire spread pattern (cooldown ~8 frames, multiple bullets/shot)
- **Laser**: High-damage piercing beam (cooldown ~12 frames, penetrates enemies)
- Weapon level 1–3 increases spread; level 4 switches to alternate weapon

### Scoring & Combo
- Base score from enemy kills (grunt=10, interceptor=15, tank=35, boss=500)
- Combo multiplier increases on consecutive kills within a time window
- Score: `kills × base × multiplier`

### Bomb Mechanic
- Limited use (2–4 per game depending on difficulty)
- Clears all on-screen bullets and stuns enemies
- Large AoE explosion effect

### Power-ups
- Spawn randomly from defeated enemies
- Types: extra life, extra bomb, weapon upgrade
- Persist in-game until collected or screen clear

## How to Modify the Game

### Adjust Difficulty Parameters
Edit the `DIFFICULTIES` object (lines 485–489):
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502–506) with w, h, hp, speedMul, color, score, minStage
2. Add sprite drawing logic in `drawPlane()` variant (around line 913) or create a new draw function
3. Reference in `spawnEnemy()` (around line 1100) spawn logic

### Modify Boss Attack Patterns
Edit `BOSS_PATTERNS` array (lines 1372–1409). Each boss gets an array of pattern functions:
```javascript
b => fireRadial(b, 14, 150),     // Radial burst, 14 bullets, speed 150
b => fireAimedSpread(b, 5, 260, 0.14),  // 5-shot aimed spread
```

Available firing functions: `fireRadial`, `fireAimedSpread`, `fireColumns`, `fireSpiralBurst`, `fireCross`, `fireHomingSeeded`, `fireDoubleRing` (lines 1324–1370).

### Adjust Visual Themes
Edit `THEMES` array (lines 493–500) to change sky gradients, cloud/sun colors per stage. Themes assigned sequentially as stages progress.

### Add or Modify Sound Effects
The `sfx` object (lines 624–676) contains all sound effects as function calls to `beep()` and `noiseBurst()`:
- `beep(freq, duration, type, volume, opts)`: Oscillator-based tone with optional sweep, filter, pan
- `noiseBurst(duration, volume, opts)`: White noise with filter
- opts: sweepTo, filterType, filterFreq, pan, delay

Example: `shoot: () => beep(760, 0.07, 'square', 0.045, { sweepTo: 480, filterType: 'lowpass', filterFreq: 2600 })`

### Performance & Quality Scaling
Lines 473–483 track frame times and adjust `quality` (0=full, 1=medium, 2=low). This is observable but not directly used in rendering; implement quality-based optimizations by checking `quality` variable in draw/update loops.

### Canvas & Controls
- **Canvas resolution**: 480×720 (BASE_W, BASE_H, line 753)
- **Input priority**: Keyboard/touch joystick > mouse steering (mouse acts as passive targeting)
- **Touch joystick**: Visual on-screen on touch devices; radius 100px, stick 44px
- **Mobile buttons**: Fire and Bomb buttons appear in action zone on right side

## Game Flow

1. **Overlay (start screen)**: Display difficulty buttons, best score, controls hint
2. **Game loop**: Update player, spawn/move enemies, check collisions, draw all
3. **Stage progression**: After 18 kills, spawn boss; after boss defeat, increment stage
4. **Game over**: Triggered when lives reach 0; show final stats
5. **Pause**: Toggle with P or Esc; overlay freezes updates but remains visible

## Development Notes

### No Build Step
Simply open `index.html` in a browser. All CSS is inline; no external stylesheets or scripts.

### Testing Changes
- Open in Chrome DevTools (F12) to inspect canvas and DOM
- Use localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`
- Clear localStorage to reset game state: `localStorage.clear()`

### Screen Shake & Flash
- `screenShake(magnitude, duration)`: Modifies canvas transform every frame
- `flashHit()` and `flashBomb()`: Trigger brief white flash overlay
- Respected by reduced-motion preference (lines 349–358)

### Accessibility
- Full keyboard controls (arrows, space, X, P/Esc)
- Touch-first design for mobile
- Reduced motion mode respects `prefers-reduced-motion` media query
- Screen reader live region for announcements (`srLive`, line 451)

### Known Constraints
- Single-threaded (all physics, collision, audio in main thread)
- Canvas draws every frame (no dirty-rect optimization)
- No networking or persistence beyond localStorage
- Audio requires user interaction (click/touch) to unmute per browser autoplay policy
