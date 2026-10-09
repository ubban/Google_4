# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire application is a single `index.html` file containing HTML, CSS, and JavaScript (~1670 lines). No build system, dependencies, or external assets are required.

## How to Run

Simply open `index.html` in any modern web browser. The game:
- Runs at 480×720 canvas size, scaling to fit the viewport (up to 1.6× on large screens)
- Supports mouse, keyboard, and touch (mobile) controls
- Uses browser localStorage for persistence (best score, muted state, difficulty, reduced motion)
- Synthesizes all sound with WebAudio API (no external audio files)

### Controls
- **Mouse**: Move to steer, click to fire
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X or Shift to bomb, P or Esc to pause
- **Mobile**: Drag left zone to move (drag-based joystick), tap FIRE button, tap BOMB button

## Code Architecture

The game is organized as a single IIFE (Immediately Invoked Function Expression) with the following major systems:

### Input System (lines 768–861)
Handles keyboard, mouse, and touch input:
- `keys` object tracks pressed keys
- Touch joystick with visual feedback (`#joystickBase`, `#joystickStick`)
- Mouse steering: plane glides toward pointer when hovering the canvas
- Keyboard/joystick input takes priority over mouse when active

### Game State (lines 863–911)
Manages all mutable game objects:
- `player`: position, velocity, health, weapon, level, invulnerability timer
- `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Game flags: `running`, `paused`, `gameOver`
- Score, lives, kills, stage, chain multiplier

### Physics & Collision (lines 1069–1294)
- `rectsOverlap(a, b)`: simple AABB collision detection
- Bullet–enemy collision: checks pierce level (laser weapons can pass through)
- Bullet–boss collision: boss takes 3 damage per hit
- Enemy/bullet–player collision: triggers `loseLife()`
- Power-up pickup: applies `applyPowerUp()` logic

### Rendering (lines 1520–1637)
Layered canvas drawing:
1. Sky background (gradient per stage theme)
2. Sun glow effect
3. Parallax scrolling clouds
4. Player plane and enemy planes (shared `drawPlane()` function)
5. Bullets (styled per weapon type; laser bullets glow on high quality)
6. Enemy bullets (red circles with glow)
7. Boss ship (dedicated `drawBossShip()` with animated accent details)
8. Power-ups (rotating squares per type)
9. Particle system for explosions

**Quality Adaptation** (lines 474–483): Frame time tracking reduces particle count and shadow blur on slow devices (`quality` level 0–2).

### Game Loop (lines 1640–1665)
- `update(dt)`: processes input, physics, spawning, collision detection per frame
- `draw()`: renders all objects to canvas
- **Frame-based timing**: delta time clamped to 50ms max to prevent large jumps
- Screen shake effect applied during combat
- Runs at `requestAnimationFrame` for vsync-locked rendering

### Audio System (lines 542–676)
WebAudio synthesis—all SFX are procedurally generated:
- `beep(freq, dur, type, vol, opts)`: tone generator with frequency sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: noise for explosions
- `sfx` object: sound effects for shoot, laser, hit, bomb, powerup, boss events
- Master gain + dynamics compressor for polished output
- Respects user mute preference and system reduced-motion setting

### Enemy & Boss Systems

**Enemy Types** (lines 502–506):
- **Grunt**: basic red fighter (1 HP, 10 points)
- **Interceptor**: faster, weaving (1 HP, 15 points)
- **Tank**: slow, armored (4 HP, 35 points); appears stage 2+

**Enemy Spawning** (lines 1022–1041):
- Difficulty-based spawn timers
- Random type selection with stage-based unlocks
- `weave` behavior uses sine wave for evasion pattern

**Boss System** (lines 1043–1056, 1372–1439):
- 6 boss designs that cycle: VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
- Each has unique attack patterns (3 per boss)
- Boss HP scales with difficulty and stage
- Telegraph phase before each attack
- Attack patterns use helper functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`

### Weapons & Power-ups

**Weapons**:
- **Vulcan**: rapid-fire bullets with spread levels 1–3
- **Laser**: piercing energy bolts with multi-hit capability

**Power-ups** (lines 1058–1067):
- **Life**: restore 1 HP (6% spawn rate)
- **Bomb**: refill bomb count (8% spawn rate)
- **Weapon**: level up or swap weapon (20% spawn rate)

### Score & Difficulty

**Difficulties** (lines 485–489): Easy, Normal (default), Hard
- Control spawn rate, enemy speed, enemy fire rate, starting lives, bomb count, boss HP multiplier

**Multiplier System** (lines 1087–1101):
- `chain` counter increments on each kill
- Multiplier increases every 5 kills (1× → 2× → ... → 10×)
- Chain timer resets if player doesn't kill for 2.2 seconds
- Used for scoring: `addScore(baseValue * multiplier)`

### UI & State Display (lines 34–359)
- HUD: score, best, multiplier, weapon, hearts, bombs
- Boss HP bar (shown only during boss fight)
- Overlay for pause/game over/stage clear
- Difficulty buttons (Easy/Normal/Hard)
- Accessibility: screen reader announcements, reduced-motion support

### Theming (lines 493–500)
6 visual themes that rotate by stage:
- Daylight Front
- Sunset Corridor
- Night Approach
- Storm Front
- Arctic Wastes
- Volcanic Ridge

Each theme defines sky gradient, cloud color, and sun glow.

## Key Game Design Notes

- **Difficulty Ramp**: Enemy spawn rate decreases every ~5 seconds of frame time, making the game progressively harder
- **Stage Progression**: Kill 18 enemies (`KILLS_PER_STAGE`) to trigger boss spawn warning, then boss arrives 1.2s later
- **Invulnerability**: Player is invulnerable for 1.5s after taking damage; no damage accumulation during this window
- **Particle System**: Particle count scales with detected device performance; explosion size varies by enemy type
- **Screen Shake**: Severity and duration tied to event type (bomb > boss hit > player hit); reduced if user prefers reduced motion

## Local Storage Keys

- `skyfighter.bestScore`: best score achieved
- `skyfighter.muted`: "0" or "1" for mute state
- `skyfighter.difficulty`: "easy", "normal", or "hard"
- `skyfighter.reducedMotion`: "0" or "1" for animation preference (defaults to system prefers-reduced-motion)

## Editing Guidelines

1. **Gameplay Balance**: Modify `DIFFICULTIES`, `ENEMY_TYPES`, `BOSSES`, or `KILLS_PER_STAGE` to tweak challenge
2. **Visual Polish**: Adjust colors, shadow blur (quality-dependent), or particle counts in `spawnExplosion()`
3. **Sound Design**: Edit `sfx` object frequency/duration/envelope parameters; `beep()` and `noiseBurst()` are flexible
4. **Enemy Patterns**: Add new boss patterns to `BOSS_PATTERNS` array; pattern functions receive `b` (boss) object and fire `enemyBullets`
5. **Performance**: Quality levels are auto-managed; manually reduce shadow blur or particle caps if targeting weak devices
6. **Accessibility**: Maintain `reduced-motion` CSS support and screen reader announcements; test with `prefers-reduced-motion: reduce` in DevTools

## Canvas & Rendering

- **Base size**: 480×720 (portrait)
- **Dynamic scaling**: up to 1.6× on large viewports
- **Rendering layers** (z-order): sky → clouds → players → bullets → boss → particles → UI overlays
- **Performance**: Shadow effects disabled on quality level 2 (low-end devices); particle count capped per level
