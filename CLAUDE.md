# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The entire game (1669 lines) runs in a canvas element with procedurally generated audio using the Web Audio API. Players control a fighter plane, switch between weapons, drop smart bombs, and defeat stage bosses across multiple difficulty levels.

## Architecture

The game uses a **single-file monolithic architecture** with code organized into logical sections:

### Core Structure
- **HTML/CSS** (lines 1-499): UI layout, HUD styling, overlay screens, responsive design with `clamp()` for mobile scaling
- **JavaScript** (lines 500-1666): Entire game logic in a single IIFE (Immediately Invoked Function Expression)

### Game Loop & Timing (lines 1640-1653)
```
loop(now) → trackPerf(dt) → update(dt) → draw() → requestAnimationFrame(loop)
```
- Delta time capping at 50ms prevents large jumps
- Update and draw are only called when `running && !paused`
- Screen shake effects applied during rendering

### Key Game Systems

#### 1. **Audio System** (lines 545-676)
- Procedural audio using oscillators (sine, square, sawtooth, triangle)
- `beep()`: Frequency synthesis with optional frequency sweep and filters
- `noiseBurst()`: White noise with lowpass filtering
- `panFor()`: Stereo panning based on screen position
- `sfx` object: ~10 sound effects (shoot, laser, explode, bomb, powerup, hit, etc.)
- Master gain control with mute button (localStorage persistence)

#### 2. **Input System** (lines 768-861)
- **Keyboard**: Arrow keys/WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Canvas hover controls plane position, click to fire (crosshair cursor)
- **Touch**: Split zones - left 55% for joystick movement, right 45% for fire button
- **Joystick mechanics**: Virtual analog stick with 50px deadzone; normalizes to -1 to 1 range
- `keys{}` object tracks all pressed keys; `mouseActive` flag prevents conflicts between control types

#### 3. **Game State** (lines 864-910)
Primary state variables:
- `player`: position, velocity, weapons, health, invulnerability frames
- `bullets`: player projectiles with pierce/laser flags
- `enemies`: array of enemy planes; removed when off-screen
- `boss`: single boss ship (null when not active)
- `score`, `lives`, `stage`, `chain`, `multiplier`: progression tracking
- `running`, `paused`, `gameOver`: game flow flags
- `frame`: incremented each update (used for animations, patterns)

#### 4. **Rendering** (lines 1520-1638, 913-1050)
- **Canvas size**: 480×720 (BASE_W/BASE_H); scaled to fit window up to 1.6×
- **Theme system**: 6 themes with sky gradients, sun color, cloud color (cycles per stage)
- **Drawing functions**:
  - `drawPlane()`: Procedural plane geometry with optional flame
  - `drawBossShip()`: Boss with animation and telegraph visual
  - Particles with fade and scaling
  - Enemy/boss health bars
- **Visual effects**: Screen shake, flash on hit/bomb, combo popup, stage banner, boss health bar, vignette (red pulse on critical health)

#### 5. **Enemy System** (lines 1093-1228)
- **Enemy types** (ENEMY_TYPES): grunt, interceptor, tank (different HP, speed, score, appearance)
- **Spawning**: Random selection, difficulty ramp increases spawn rate over time (every 5s harder)
- **Behaviors**:
  - Weaving: sine wave movement using frame-based animation
  - Shooting: spread pattern (3-bullet spread) or aimed shots at player
  - Bullet variety: straight, aimed, spread
- **Pooled spawning**: 10 kills required per stage before boss warning appears (1200ms delay)

#### 6. **Boss System** (lines 1372-1454)
- **6 boss designs** (BOSSES array): Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan
- **Boss patterns** (BOSS_PATTERNS): Each boss has 3 attack patterns (radial, aimed, columns, spirals, cross-fire, homing, etc.)
- **Attack cycle**: Telegraph (22 frames visual warning) → Fire pattern → AttackTimer cooldown (85 frames)
- **Boss movement**: Horizontal bouncing, velocity reversal at screen edges
- **Telegraphing**: White core flash + sound effect before attack
- **Entering**: Boss slides down from top with collision disabled until in position

#### 7. **Weapon System** (lines 1085-1092)
- **Two weapons**: Vulcan (rapid fire bullets) and Laser (piercing beam)
- **Levels**: 3 upgrade levels per weapon, swap weapons at level 3+1
- **Firing**: Cooldown-based (player.cooldown); laser has pierce counter; vulcan rapid-fire uses `player.rapid` counter
- **Weapon upgrades**: Power-up pickups cycle weapon type and level

#### 8. **Bomb System** (lines 1050-1083)
- **Smart bomb**: Clears all enemy bullets, damages boss
- **Resources**: Start with 2 bombs (difficulty dependent), max 2 (pickups refill)
- **Triggering**: X key or bomb button; requires `bombCount > 0`
- **Effect**: Radius 200px explosion, 0.3s flash animation, damage scales with power-up chain

#### 9. **Score & Progression**
- **Score**: Enemy kills add points based on type + multiplier
- **Chain system**: Kill streak multiplier (2× at 10 kills, 3× at 20, etc.); resets on damage
- **Difficulty levels** (DIFFICULTIES): Easy, Normal, Hard, Insane — affect spawn rate, enemy HP, bomb count, starting lives
- **Persistence**: Best score and difficulty preference stored in localStorage

#### 10. **Collision Detection** (lines 1232-1294)
- `rectsOverlap()`: AABB collision for all entities
- Priority: Player bullets vs enemies → Player bullets vs boss → Enemies/bullets vs player → Pickups vs player
- Player invulnerability frames (1.5s) after taking damage; blink animation

#### 11. **Power-Ups** (lines 1289-1322)
- **Types**: Life (restore health), Bomb (add ammo), Weapon (upgrade/cycle)
- **Spawn**: 30% drop rate from grunts, 40% from interceptors, 50% from tanks
- **Movement**: Fall straight down; despawn off-screen
- **Pickup logic**: Apply effect, increment counter, play sound

## Development Notes

### Common Tasks

**Adjusting difficulty**: Modify `DIFFICULTIES` object (line ~860) — spawn rate, HP, bomb count, starting lives.

**Adding/modifying enemy patterns**: Edit `ENEMY_TYPES` for grunt/interceptor/tank, or modify shooting logic (line ~1217).

**Creating new boss patterns**: Add functions to `BOSS_PATTERNS` array (line 1372); each function receives boss object and fires `enemyBullets`.

**Customizing themes**: Edit `THEMES` array (likely around stage selection code) — includes sky gradients, sun/cloud colors.

**Tuning weapon balance**: Adjust cooldown (line ~878), damage multipliers (boss takes 3× per bullet), or pierce behavior.

**Sound effects**: Add to `sfx` object; use `beep()` or `noiseBurst()` with frequency/duration/filter params.

### Key Constants & Lookup Tables
- `KILLS_PER_STAGE`: Enemies to kill before boss spawns
- `DIFFICULTIES`: 4 presets (Easy/Normal/Hard/Insane)
- `BOSSES`: Boss visual designs (colors, shapes)
- `BOSS_PATTERNS`: 6 bosses × 3 patterns each
- `ENEMY_TYPES`: grunt, interceptor, tank definitions
- `THEMES`: Background themes per stage

### Performance & Mobile
- `quality` flag (localStorage) enables/disables canvas shadows (expensive on mobile)
- `reducedMotion` flag respects prefers-reduced-motion; tones down shake/flash
- Touch joystick auto-hides keyboard on small screens
- Canvas scales responsively; resolution always 480×720, CSS transform handles size

### Storage
- `localStorage.setItem(STORAGE_BEST, score)`: Best score persistence
- `localStorage.setItem(STORAGE_DIFF, name)`: Difficulty choice
- `localStorage.setItem(STORAGE_MUTE, '1'|'0')`: Mute state
- `localStorage.setItem(STORAGE_QUALITY, '1'|'0')`: Graphics quality

## Testing & Running

No build step required — open `index.html` directly in a browser.

**Desktop**: Arrow keys or mouse + click to play.
**Mobile**: Touch controls appear automatically (left side joystick, right side fire).
**Difficulty**: Click difficulty buttons before starting (persists).

## Entry Points

- Game loop starts at line 1665: `requestAnimationFrame(loop)`
- Start button handler at line 1655: Initializes game state and resets overlay
- All game state reset occurs in `resetGame()` (line 875)
