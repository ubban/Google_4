# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file HTML/CSS/JavaScript game with the following features:
- Player-controlled aircraft with movement and firing
- Multiple enemy types and boss encounters
- Weapon and power-up system
- Difficulty levels (Easy, Normal, Hard)
- Score multiplier/chain system for consecutive kills
- Synthesized sound effects via Web Audio API
- Touch and keyboard controls
- Responsive canvas rendering with adaptive performance

## File Structure

The entire game is contained in `index.html` (~1669 lines):
- **Lines 10-359**: CSS (styles, animations, layouts for HUD and UI)
- **Lines 422-1669**: JavaScript game logic

No external dependencies or assets—the game is completely self-contained.

## Running the Game

Simply open `index.html` in a web browser. The game starts with a menu overlay. No build step or server required.

## Architecture Overview

The game uses a **single global scope with modular organization**:

### Game State (Lines 863-869)
Centralized state object containing all game entities:
- `player`: Position, speed, weapon, health
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Arrays of game objects
- `score`, `lives`, `stage`, `chain`, `multiplier`: Game progress
- `boss`: Current boss entity (null when inactive)

### Main Game Loop (Lines ~1400+)
The game runs a continuous `animate(time)` function called via `requestAnimationFrame`:
1. **Update phase**: Move all entities, handle collisions, update timers
2. **Logic phase**: Spawn enemies, handle boss patterns, update score chains
3. **Render phase**: Draw canvas, HUD elements, apply screen effects

### Key Systems

#### 1. Difficulty System (Line 485)
`DIFFICULTIES` object defines per-difficulty values:
- Enemy spawn rates
- Enemy speed and shoot chance
- Player lives and bombs
- Boss HP multiplier

Add new difficulties by adding properties to `DIFFICULTIES` and updating difficulty buttons in HTML.

#### 2. Enemy Spawning (Line 1022)
`spawnEnemy()` creates enemies with:
- Type selection based on stage and RNG
- Properties from `ENEMY_TYPES` (grunt, interceptor, tank)
- Position variance, shoot patterns (aimed/spread), weaving motion

To add enemy types: extend `ENEMY_TYPES` object with `{ w, h, hp, speedMul, color, dark, score, minStage, weave }`.

#### 3. Boss System (Line 1043)
`spawnBoss()` creates stage bosses:
- Boss design selected from `BOSSES` array (6 unique designs, repeating per stage)
- Attack patterns handled in main loop (~Line 1500)
- Boss phases (entering → combat → defeated)

To add bosses: extend `BOSSES` array with `{ name, hullTop, hullBottom, wing, core, hpMul }`.

#### 4. Weapon System (Line 871)
Player has two weapons: `vulcan` (default) and `laser`. Weapon level caps at 5.
- Vulcan: Single rapid shots
- Laser: Fewer, wider spread shots

Weapon shooting happens in the main update loop (~Line 1370). Modify `player.cooldown` and `player.rapid` mechanics to adjust weapon behavior.

#### 5. Collision Detection (Line 1069)
`rectsOverlap(a, b)` checks AABB collision between two objects. Used for:
- Player/enemy bullets
- Player/enemies
- Player/power-ups

#### 6. Audio System (Lines 542-676)
Web Audio API with synthesized effects (no external files):
- `beep(freq, dur, type, vol, opts)`: Oscillator-based tones with sweeps and filters
- `noiseBurst(dur, vol, opts)`: Noise-based explosions
- `sfx` object maps game events to audio functions

Muting controlled by localStorage. All audio is processed through a master gain node with compression.

#### 7. Input Handling (Lines 767-861)
Three control schemes with priority:
1. **Keyboard**: WASD/arrows for movement, Space/click to fire, X/Shift for bomb
2. **Mouse**: Player follows cursor while hovering canvas
3. **Touch**: Joystick in bottom-left, fire/bomb buttons in bottom-right

Active control scheme auto-prioritized—keyboard/joystick override mouse input.

#### 8. Visual Effects
- **Screen shake**: `screenShake(magnitude, duration)` applied to camera in render phase
- **Flash**: White flash overlay on hit/bomb detonation
- **Vignette**: Red pulsing edge glow when critical health (1 heart remaining)
- **Combo pop**: Text animation for kill chains
- **Stage banner**: Stage intro animation

Reduced motion mode (respects prefers-reduced-motion) disables animations.

#### 9. Adaptive Performance (Lines 473-482)
Tracks average frame time and adjusts quality levels:
- Quality 0: Full particle count
- Quality 1: 60% particles
- Quality 2: 35% particles

Helps maintain 60 FPS on slower devices.

## Common Development Tasks

### Tweaking Difficulty Balance
Edit `DIFFICULTIES` constants (Line 485). Key parameters:
- `spawnBase`: Frames between enemy spawns (lower = more enemies)
- `spawnMin`: Minimum spawn interval during spawn surge
- `enemySpeed`: Base enemy movement speed multiplier
- `enemyShootChance`: Probability an enemy can shoot (0–1)
- `bossHpMul`: Boss health multiplier
- `lives` / `bombs`: Starting player resources

### Adding New Enemy Type
1. Add entry to `ENEMY_TYPES` object (Line 502) with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`
2. Update `spawnEnemy()` logic (~Line 1027) to choose your new type with appropriate RNG thresholds
3. If needed, add explosion SFX to `EXPLODE_SFX` (Line 678)

### Adding New Boss
1. Add entry to `BOSSES` array (Line 508) with `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. No code changes needed—stage automatically cycles through bosses
3. Customize attack patterns in the boss attack logic (search `b.pattern` in main update loop, ~Line 1500)

### Adjusting Weapon Behavior
Weapon firing happens in the main update loop. To modify:
- Fire rate: Change `player.cooldown` decrement and fire threshold
- Bullet speed: Adjust projectile `vy` value when spawned
- Bullet spread: Change x-offset calculations when adding bullets

### Adding Screen Effects
New visual effects can be added by:
1. Creating a timer variable (e.g., `flashTimer`)
2. Triggering with `flashTimer = duration` during gameplay
3. Applying the effect in the render phase based on timer countdown

### Tweaking Spawn Rates
In `resetGame()` (Line 875), `spawnTimer` is initialized. In the main loop, increment and check:
```
spawnTimer++;
if (spawnTimer > spawnInterval) {
  spawnEnemy();
  spawnTimer = 0;
}
```
The spawn interval is calculated from difficulty (`d.spawnBase` and `d.spawnMin`).

## Code Organization Tips

- **Game constants** (DIFFICULTIES, ENEMY_TYPES, BOSSES, THEMES): ~Lines 485–499
- **Audio system**: Lines 542–676
- **Canvas setup & input**: Lines 752–861
- **Game state initialization**: Lines 863–911
- **Drawing functions**: Lines 913–1020
- **Spawn/object creation**: Lines 1022–1067
- **Main update loop**: Lines ~1350–1550
- **Render function**: Lines ~1550–1669

## Performance Considerations

- Particle count adapts dynamically (see quality system)
- Collision checks use simple AABB (fast)
- Canvas rendering is 2D context (no WebGL)
- Audio synthesis happens in real-time; muting disables synthesis to save CPU

On low-end devices, the quality variable drops to reduce particle counts automatically. If targeting even lower-end devices, consider reducing initial particle counts or skipping shadow effects in `drawBossShip()`.

## Testing

There's no automated test suite. Manual testing:
1. Test all three difficulty levels—verify enemy behavior and spawn rates
2. Test all control schemes (keyboard, mouse, touch)
3. Verify audio synthesis works and muting toggles correctly
4. Check visual effects are smooth on target devices
5. Verify boss attack patterns are challenging but fair
6. Test score multiplier chain system across different kill patterns

## Local Storage

The game persists:
- `skyfighter.bestScore`: Highest score
- `skyfighter.muted`: Audio mute state (1/0)
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion effects toggle (1/0)

These keys are used throughout for persistence across sessions.
