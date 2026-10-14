# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built in vanilla JavaScript with Canvas 2D rendering and WebAudio procedural audio synthesis. The entire game is contained in a single `index.html` file (~1669 lines).

### Key Characteristics
- **Single-file architecture** — all HTML, CSS, and JavaScript in `index.html`
- **No external dependencies** — no npm packages or build system needed
- **Procedural audio** — all sound effects are synthesized via WebAudio API
- **Canvas rendering** — uses Canvas 2D context for all game visuals
- **Touch & keyboard support** — includes mobile joystick controls
- **Accessibility-first** — uses aria-live announcements for screen readers
- **Adaptive quality** — performance monitoring adjusts rendering quality dynamically

## Running the Game

### Local Development
Simply open `index.html` in a modern web browser. The game runs without any build step:
```bash
# Option 1: Open with file:// protocol in browser
open index.html

# Option 2: Start a simple HTTP server
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

No dependencies to install, no build script to run.

### Browser Requirements
- Modern browser with Canvas 2D and WebAudio support (all major browsers)
- Touch support for mobile (iOS Safari, Android Chrome)
- ~1.5MB file size (self-contained)

## High-Level Architecture

The game follows a classic **update/render loop** pattern with game state organized in module-level variables within a single IIFE (Immediately Invoked Function Expression) for scope isolation.

### Core Loop
```
requestAnimationFrame(loop) → update(dt) → draw() → repeat
```

**Main entry point:** `loop()` function at line ~1219
- Calculates delta time since last frame
- Calls `update(dt)` to advance game logic
- Calls `draw()` to render frame
- Calls `trackPerf(dt)` for adaptive quality

### Major Game Systems

1. **Player & Input System**
   - Player object holds position, velocity, weapon state, invulnerability frames
   - Keyboard input: arrow keys + space/Z for fire/bomb
   - Mouse/touch input: aim with cursor position or virtual joystick
   - Firing patterns change based on selected weapon (laser cycles through patterns)

2. **Weapon System**
   - Player starts with VULCAN weapon (level 1)
   - 3 weapon upgrade levels (unlocked by collecting power-ups)
   - Each level changes firing pattern:
     - **Level 1 (VULCAN):** Straight shots
     - **Level 2:** Spread shot
     - **Level 3+:** Advanced patterns (laser patterns, homing shots, etc.)
   - Laser weapon has special spreading/aiming behavior

3. **Enemy System**
   - Three enemy types defined in `ENEMY_TYPES`: grunt, interceptor, tank
   - Each type has distinct speed, HP, color, and score value
   - Enemies spawn based on difficulty-scaled timers
   - Enemies shoot at player with difficulty-scaled frequency
   - Enemies are removed when HP reaches 0 or they leave screen

4. **Boss System**
   - 6 boss types in `BOSSES` array (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
   - Boss appears after 18 kills per stage
   - Boss has HP pool, telegraph (warning) animation, distinct attack patterns
   - Multiple firing patterns via `fireRadial()`, `fireAimedSpread()`, `fireSpiralBurst()`, etc.
   - Boss defeat triggers screen shake, special audio cue, stage transition

5. **Collision Detection**
   - Simple AABB (axis-aligned bounding box) checking
   - Bullets vs enemies, bullets vs player, enemies vs player
   - Collision checks in `update()` function

6. **Particle System**
   - Spawned on enemy/boss death, weapon fire (visual feedback)
   - Simple velocity-based particles with lifetime
   - Used for explosion effects and visual polish

7. **Audio System** (lines ~545-625)
   - WebAudio synthesis with no external audio files
   - `beep()` function for tone generation (frequency, duration, waveform)
   - `noiseBurst()` function for noise-based effects
   - All SFX defined in `sfx` object with named handlers
   - Dynamic audio panning based on bullet position (stereo positioning)
   - Master gain with dynamics compression for professional audio feel

8. **Difficulty System**
   - Three difficulties: easy, normal, hard
   - Parameters affected: enemy spawn rate, enemy speed, shoot frequency, lives, bombs, boss HP
   - Selected from overlay menu before game starts
   - Persisted in localStorage

9. **HUD & UI System**
   - Score, best score, multiplier, weapon name/level displayed in top bar
   - Heart icons show remaining lives (with critical-state pulsing animation)
   - Bomb count shown as bomb icons
   - Combo pop-up shows "+N pts" on enemy kills
   - Stage banner announces new stages
   - Boss health bar appears during boss fights
   - Pause overlay with buttons for mute, reduced motion, pause

10. **Performance Adaptation** (lines ~476-485)
    - Tracks last 45 frame times
    - Adjusts `quality` variable (0=full, 1=medium, 2=low) based on average FPS
    - Lower quality disables particles and reduces draw calls (see `draw()`)

### Game State Variables
Located at module scope (lines ~440-1060):
- `player` — position, velocity, weapon, HP, invulnerability, etc.
- `enemies`, `bullets`, `enemyBullets`, `powerUps`, `particles` — arrays of game objects
- `boss` — current boss object (if any)
- `score`, `lives`, `bombs`, `gameState` — game progress tracking
- `stage`, `stageKills`, `multiplier` — score/progression tracking
- `paused`, `running` — game flow control
- `frame` — absolute frame counter since game start
- `quality` — adaptive rendering quality (0=high, 2=low)

### Game Flow
1. **Title screen** — gameState='title' → player selects difficulty
2. **Playing** — gameState='playing' → normal enemy spawn and boss progression
3. **Boss fight** — boss object exists, telegraph and attack patterns active
4. **Game over** — gameState='gameover' → show stats, allow restart

## Key Functions

### Game Logic
- `update(dt)` (line ~727) — Main update loop: player movement, bullet updates, collision detection, enemy/boss AI
- `updateBoss(dt)` (line ~990) — Boss-specific logic: telegraph animation, attack patterns, HP reduction
- `draw()` (line ~1099) — Render all game objects: player, bullets, enemies, particles, HUD

### Audio
- `beep(freq, dur, type, vol, opts)` (line ~566) — Synthesize single-frequency tone
- `noiseBurst(dur, vol, opts)` (line ~597) — Synthesize white noise burst
- Functions in `sfx` object — named handlers that call beep/noiseBurst with specific parameters

### Firing Patterns (Weapon System)
- `fireRadial(b, count, speed, offset)` — Fire bullets in circle around player
- `fireAimedSpread(b, count, speed, spreadStep)` — Fan of bullets toward aim direction
- `fireColumns(b, speed)` — Vertical columns of bullets
- `fireSpiralBurst(b, speed)` — Rotating spiral of bullets
- `fireCross(b, speed)` — Cross pattern (8 directions)
- `fireHomingSeeded(b, count, speed)` — Homing bullets that track enemies
- `fireDoubleRing(b, speedA, speedB)` — Two concentric rings at different speeds

### Rendering
- `draw()` (line ~1099) — Render game canvas
- `drawClouds(speed, fillColor, rx, ry)` — Parallax scrolling cloud effect (visual background)
- `renderHearts(count, max)` — Update heart icon display
- `renderBombs(count, max)` — Update bomb icon display

### UI/Game Flow
- `togglePause()` (line ~1070) — Toggle pause state
- `endGame()` (line ~1048) — Transition to game over screen
- `loseLife()` (line ~1035) — Handle player death

## Development Notes

### Modifying Game Difficulty
Edit the `DIFFICULTIES` object (line ~485):
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, enemyShootChance: 0.45, lives: 4, bombs: 3, bossHpMul: 0.8 },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, enemyShootChance: 0.7,  lives: 3, bombs: 2, bossHpMul: 1.0 },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, enemyShootChance: 0.95, lives: 3, bombs: 2, bossHpMul: 1.3 }
};
```

### Adding New Enemy Types
1. Add entry to `ENEMY_TYPES` object (line ~502) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update enemy spawning logic in `update()` to include new type
3. Add color rendering in `draw()` enemies section

### Adding New Bosses
1. Add entry to `BOSSES` array (line ~487) with properties: `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Create new boss object in `update()` with `x`, `y`, `hp`, `hpMax`, `attacking`, `telegraph`, etc.
3. Add attack pattern function and call in `updateBoss()`
4. Add rendering logic in `draw()` boss section

### Weapon Upgrade Balance
Player weapon upgrades are triggered by collecting power-ups (line ~890):
```javascript
if (player.level < 3) {
  player.level++;
}
```
The firing pattern is selected in `update()` based on `player.level`. Adjust spread angles and bullet speeds here.

### Audio Synthesis Tuning
All audio is generated via `beep()` and `noiseBurst()`. Modify the `sfx` object (line ~624) to adjust:
- Frequency (in Hz) for pitch
- Duration for note length
- `sweepTo` for frequency glides
- `filterFreq` for tone shaping
- Volume (`vol` parameter)

### Touch Control Tuning
Virtual joystick is rendered with `drawJoystick()` and updated in touch event handlers. Adjust:
- Joystick size/position in CSS (search `.joystick`)
- Deadzone and max displacement in touch handlers (around line ~1180)

### Performance Considerations
- Adaptive quality system (`quality` variable) disables expensive features at 0 FPS:
  - Lower quality: particles may be skipped or reduced
  - Check `if (quality < 2)` conditions before expensive operations
- Canvas size is adjusted on window resize (line ~754)
- Particle count is limited to prevent performance degradation

## Common Tasks

### Adjust Enemy Behavior
- Spawn rate: Edit `spawnBase` and `spawnMin` in `DIFFICULTIES`
- Movement: Modify `enemy.y += enemy.speed * enemy.speedMul * dt` in `update()`
- Shooting: Change `enemyShootChance` parameter

### Add Visual Effects
- New particle type: Create particle object `{ x, y, vx, vy, life, ... }` and push to `particles` array
- Screen shake: Call `screenShake(magnitude, duration)`
- Flash effect: Call `flashHit()` or `flashBomb()`
- Combo pop-up: Call `showComboPop("+100 pts")`

### Adjust Scoring
- Base score per enemy: Edit `ENEMY_TYPES[type].score`
- Multiplier system: Change `multiplier` variable and `multiplier *= scaleFactor` logic
- Best score tracking: Automatically saved to localStorage on game over

## Persistence
Game state is persisted in browser localStorage with these keys:
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Last selected difficulty
- `skyfighter.reducedMotion` — Reduced motion preference

Clear localStorage if needed for testing fresh state.
