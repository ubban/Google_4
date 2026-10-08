# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in vanilla HTML5 Canvas and JavaScript. The game is self-contained in a single `index.html` file with no external dependencies or build process.

**How to run:** Open `index.html` in a web browser. The game runs entirely client-side with local storage for high scores.

## Game Architecture

### Core Game Loop
The game runs on a `requestAnimationFrame` loop (line 1640). Each frame:
1. Calculates delta time (`dt`) for frame-independent gameplay
2. Calls `update(dt)` to advance game state (collisions, movement, spawning)
3. Calls `draw()` to render everything via Canvas 2D API
4. Adapts rendering quality based on frame time via `trackPerf()` (lines 474-482)

### Key State Variables
- **Player:** Position, weapon, ammo, health, invulnerability frames
- **Enemies:** Array of spawned enemies with type-specific properties
- **Bullets:** Player and enemy bullets, tracked separately for collision detection
- **Boss:** Single optional boss entity with complex attack patterns
- **Game State:** `running`, `paused`, `gameOver`, current `stage`, `score`, `multiplier` (combo system)

### Main Systems

#### 1. **Difficulty System** (line 485)
Four difficulty levels configured in `DIFFICULTIES` object control:
- Spawn rates and patterns
- Enemy movement speed
- Enemy shoot frequency
- Player lives and bombs
- Boss HP multipliers

Difficulty is persisted to `localStorage` and can be changed pre-game.

#### 2. **Enemy Spawning & Types** (line 502, 1022)
Three enemy types with different stats:
- **Grunt:** Basic fast enemy, 1 HP
- **Interceptor:** Slightly faster, weaves side-to-side
- **Tank:** Slow, tanky (4 HP), high score reward, appears from stage 2+

Spawn timer decreases based on frame count (line 1201), creating progressive difficulty ramp.

#### 3. **Weapon System** (line 1124)
Two weapons with 3 levels each:
- **Vulcan:** Spread shot (up to 3 bullets at level 3)
- **Laser:** Piercing shots in lanes (up to 3 shots at level 3)

Weapon level increases via power-ups; at level 3 you can switch weapons.

#### 4. **Boss System** (line 508, 1043)
- **Six boss types** with unique designs and attack patterns (line 508-515)
- Each boss has **3 attack patterns** in `BOSS_PATTERNS` array (line 1372)
- Boss spawns after 18 enemy kills per stage
- Telegraph mechanic: boss briefly flashes core before attacking (line 1000)
- Damage calculation: player bullets deal 3 HP, bombs deal 24 HP per boss

#### 5. **Scoring & Combo System** (line 1087)
- Killing enemies adds score: `base * multiplier`
- Chain counter starts on first kill, increments on consecutive kills
- Multiplier increases every 5 kills (cap: 10x)
- Chain resets after 2.2 seconds of no kills
- Breaking the chain also breaks on damage taken

#### 6. **Input Handling** (line 768)
Three input modes (with priority order):
1. **Keyboard:** Arrow keys or WASD for movement, Space/X/Shift for fire/bomb
2. **Joystick (touch):** Left side drag, right buttons for fire/bomb
3. **Mouse:** Movement auto-targets; click to fire

Keyboard/joystick take priority over mouse to prevent conflicts.

#### 7. **Audio System** (line 542)
Pure Web Audio API synthesis—no external sound files:
- Uses oscillators (square, sawtooth, sine, triangle) and noise bursts
- Spatial panning based on event x-coordinate
- Mute toggle saved to localStorage
- `sfx` object (line 624) contains all sound effects

#### 8. **Visual Themes** (line 493)
Six sky themes that cycle per stage, each with:
- 3-stop gradient for sky
- Cloud colors and speeds
- Sun glow effect
- Each theme has a distinct name (e.g., "Volcanic Ridge", "Arctic Wastes")

### Collision Detection

Uses simple AABB (axis-aligned bounding box) collision via `rectsOverlap()` (line 1069). Checked for:
- Player bullets ↔ enemies
- Player bullets ↔ boss
- Enemy bullets ↔ player
- Enemies ↔ player
- Boss ↔ player
- Power-ups ↔ player

### Particle Effects & Screen Effects

- **Explosions:** Spawned via `spawnExplosion()` with configurable particle count (scales with quality)
- **Screen shake:** Magnitude and duration set per event via `screenShake()`
- **Flash effects:** White flash for hits, bomb flash
- **Vignette:** Red pulsing border during critical health (1 HP)
- **Reduced motion:** CSS `prefers-reduced-motion` respected; toggleable UI button disables all animations

## File Structure

Single file: `index.html`
- Lines 1-359: HTML structure and CSS
- Lines 360-421: HTML body and canvas elements
- Lines 422-1667: JavaScript game code

## Key Modifications & Extensions

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502) with stats
2. Add explosion sound to `EXPLODE_SFX` object (line 678)
3. Add sound function to `sfx` object if needed (line 624)
4. Optionally add to `minStage` filter in `spawnEnemy()` (line 1026)

### Adding a New Boss
1. Add to `BOSSES` array (line 508) with name and colors
2. Add 3 attack pattern functions to `BOSS_PATTERNS` (line 1372)
3. HP scaling is automatic via `hpMul`

### Adding a Power-up Type
1. Add spawn logic to `maybeSpawnPowerUp()` (line 1058)
2. Add rendering in `draw()` power-up section (line 1617)
3. Add effect logic to `applyPowerUp()` (line 1304)

### Tweaking Difficulty
Edit `DIFFICULTIES` object (line 485) or adjust the spawn rate logic (line 1201).

### Adding Visual Effects
Add particle spawning to relevant events, or add new CSS animations and use `classList` to trigger them (similar to combo pop, stage banner).

## Performance Optimizations

- **Quality scaling:** `quality` variable (0-2) scales particle counts and shadow blur based on frame rate
- **Canvas sizing:** Uses CSS scaling rather than changing canvas resolution for speed
- **DOM minimalism:** Only HUD elements updated via DOM; game world rendered to Canvas
- **Requestable reflow:** `offsetWidth` access in animation triggers forces layout recalc for clean animation restarts

## Testing the Game

- **Manual testing:** Open in browser, play through stages
- **Difficulty testing:** Click difficulty buttons pre-game to test each level
- **Touch testing:** Use browser DevTools device emulation or physical device
- **Audio testing:** Toggle mute button; use browser DevTools to verify Web Audio context is active
- **Storage testing:** Open DevTools → Application → Local Storage to inspect high score persistence

## Browser Compatibility

Requires:
- Canvas 2D API
- `requestAnimationFrame`
- Web Audio API (optional; game works silent if unavailable)
- LocalStorage
- Touch events (optional; desktop fallback to mouse/keyboard)

Tested on modern Chrome, Firefox, Safari, Edge.
