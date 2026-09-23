# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. It runs entirely in the browser with no build process, external dependencies, or assets. The game features keyboard/mouse and touch controls, three difficulty levels, boss encounters, and a procedural weapon/enemy system.

## Running the Game

The game is a single `index.html` file. To run:
1. Open `index.html` directly in any modern web browser
2. The game auto-detects touch devices and enables virtual joystick controls
3. All game state (best score, difficulty, mute setting, motion preferences) persists via localStorage

**Controls:**
- **Desktop**: Arrows/WASD to move, Space/Click to fire, X/Shift to bomb, P/Esc to pause
- **Mobile**: Drag left side to move, tap FIRE, tap BOMB

## Code Architecture

The entire game is a single `<script>` tag in index.html using an IIFE (Immediately Invoked Function Expression) for scope isolation. The code is organized into these functional sections:

### 1. **Game State & Config** (lines 463-530)
- Constants: `DIFFICULTIES`, `THEMES`, `ENEMY_TYPES`, `BOSSES`, `BOSS_PATTERNS`
- Storage keys for persistent state (best score, mute, difficulty, reduced motion)
- Initial state setup from localStorage

### 2. **Audio System** (lines 542-676)
- Custom WebAudio API synthesizer (no external audio files)
- `beep()` and `noiseBurst()` for sound generation with frequency sweeps and filtering
- `sfx` object with named sounds for all game events (shoot, explode, bomb, boss hits, etc.)
- Audio context lazily initialized on first user interaction

### 3. **Input Handling** (lines 768-862)
- Keyboard events (arrows, WASD, space, X)
- Mouse events: movement-based steering when hovering canvas + click to fire
- Touch events: joystick movement on left side, fire/bomb buttons on right side
- Keyboard/joystick input has priority over mouse steering (no fighting)

### 4. **Game Loop & Rendering** (lines 1640-1666)
- `loop()`: requestAnimationFrame runner with delta-time, performance tracking, update/draw phases
- `draw()`: Canvas 2D rendering with theme-based sky gradients, dynamic clouds, glow effects
- Performance adaptation: quality levels (0=full, 1=medium, 2=low) based on average frame time

### 5. **Game Logic & Collisions** (lines 1148-1302)
- `update(dt)`: Player movement, firing, enemy spawning, collision detection
- Collision detection: `rectsOverlap()` for axis-aligned bounding boxes
- Damage system: bullets pierce through/deplete enemy HP, boss takes 3 damage per hit
- Chain/combo system: consecutive kills extend multiplier (up to 10x)

### 6. **Enemy System** (lines 1022-1041, 1210-1228)
- `spawnEnemy()`: Random enemy type selection based on stage and difficulty
- Enemies weave side-to-side (interceptor type only)
- Shooting patterns: "spread" (arc) or "aimed" (player-tracking)
- Spawn rate decreases over time as game ramps up difficulty

### 7. **Boss System** (lines 1043-1056, 1372-1439)
- Boss spawns after 18 enemy kills per stage
- Each boss has unique design, HP multiplier, and attack patterns
- 6 distinct bosses cycle through stages
- **Attack patterns**: 3 patterns per boss defined in `BOSS_PATTERNS`
  - Radial bursts, aimed spreads, columns, spirals, homing shots, crosses, etc.
  - Pattern fires on telegraph trigger (red glow on core, audio cue)
  - 22-frame telegraph window before each pattern executes
- Boss health bar updates in real-time

### 8. **Weapon & Power-up System** (lines 1124-1146, 1304-1322)
- **Vulcan**: Spread projectiles (1, 2, or 3 way at levels 1-3)
- **Laser**: Piercing projectiles (pierce 1-2 enemies; can fire multiple lanes)
- Power-ups: Life (+1), Bomb (+1), Weapon (upgrade level or switch weapon)
- Weapon pickups spawn randomly from defeated enemies (6% life, 8% bomb, 20% weapon)

### 9. **Drawing Functions** (lines 913-1020)
- `drawPlane()`: Player/enemy aircraft with flame, wings, cockpit
- `drawBossShip()`: Boss with procedural hull gradient, rotating engines, telegraph core glow
- `drawClouds()`: Parallax scrolling background elements

### 10. **UI & Accessibility** (lines 680-750)
- HUD displays score, best, multiplier, weapon level, lives, bombs
- Boss health bar with name
- Screen effects: flash on hit, vignette pulse when critical health, screen shake
- Screen reader announcements via `announce()`
- `prefers-reduced-motion` support disables animations/shake/flash
- Pause menu with difficulty selection

## Key Game Constants

| Constant | Purpose |
|----------|---------|
| `DIFFICULTIES` | 3 difficulty presets: spawn rates, enemy speed, lives, bombs, boss HP multiplier |
| `THEMES` | 6 visual themes with sky gradient colors and cloud styling |
| `ENEMY_TYPES` | Grunt (basic), Interceptor (fast, weaving), Tank (slow, 4 HP) |
| `BOSSES` | 6 boss designs with distinct visual styles and HP multipliers |
| `BOSS_PATTERNS` | 3 attack patterns per boss type (18 total unique patterns) |
| `KILLS_PER_STAGE` | 18 kills triggers boss spawn |
| `BASE_W`, `BASE_H` | Game resolution: 480x720 (scales to fit window) |

## Extending the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502-506) with w, h, hp, speedMul, color, dark, score, minStage
2. Ensure typeKey in `spawnEnemy()` can select it
3. Add explosion SFX to `EXPLODE_SFX` if needed (line 678)
4. Update `drawPlane()` calls if different visual is needed

### Adding a New Boss
1. Add boss design to `BOSSES` array (line 508-515): name, hullTop/Bottom, wing, core colors, hpMul
2. Create 3 attack patterns in `BOSS_PATTERNS` array (line 1372-1409)
3. Use existing pattern functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc., or create new ones
4. Boss will automatically cycle through all 6 bosses as stages progress

### Adding a New Theme
1. Add entry to `THEMES` array (line 493-500): sky gradient stops, cloud color, sun color, name
2. Themes cycle by stage: `THEMES[(stage - 1) % THEMES.length]`

### Adding a New Sound Effect
1. Create function in `sfx` object (line 624-676)
2. Use `beep()` or `noiseBurst()` with audio parameters (frequency, duration, volume, filter)
3. Call via `sfx.newEffectName()` when appropriate

## Performance & Optimization

- **Quality levels**: Automatically adapted based on frame rate
  - Level 0: Full effects (shadows, glow, particle count)
  - Level 1: Medium (reduced glow, fewer particles)
  - Level 2: Low (no glow, minimal particles)
- **Canvas scaling**: Internal resolution 480x720, scales to fit window up to 1.6x
- **Particle count**: Scales with quality level in `spawnExplosion()`
- **Frame timing**: Capped dt at 0.05s prevents large jumps on frame drops

## Important Implementation Details

- **Invulnerability**: Player gets 1.5s invuln after being hit (blinking during this time)
- **Chain system**: Killing enemies consecutively extends combo (max 10x multiplier, 2.2s decay)
- **Boss telegraph**: 22-frame delay before attack pattern fires; core glows and audio warns player
- **Screen shake**: Magnitude and duration are reduced when reduced motion is enabled
- **Touch controls**: Joystick appears only while actively dragging on move zone
- **Mouse steering**: Low-priority input; keyboard/joystick always override for movement
- **Reduced motion**: Applies to vignette pulse, flash, screen shake, combo pop, stage banner, card animations

## Storage Keys

All persistent state uses localStorage with these keys:
- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Mute toggle (1 or 0)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference (1 or 0)

## Common Modifications

### Adjust Difficulty Balancing
Edit the `DIFFICULTIES` object to change spawn rates, enemy speed, lives, bombs, and boss HP multipliers.

### Change Game Resolution
Modify `BASE_W` and `BASE_H` (line 753). Note: HUD, boss bar, and UI all use clamp() for responsive scaling.

### Adjust Weapon Balance
Modify `fireWeapon()` to change fire rate (`fireDelay`), spread angles, or pierce counts.

### Tune Boss AI
Edit attack patterns in `BOSS_PATTERNS` or create new pattern functions. Patterns execute via `updateBoss()` telegraph system.

### Add Screen Effects
Use `screenShake(mag, dur)` for camera shake, `flashHit()`/`flashBomb()` for flash effects, or toggle `vignetteEl` classes for vignette.
