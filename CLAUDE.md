# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file HTML5 game — a Raiden-style vertical shoot-'em-up built entirely in vanilla JavaScript (no build tools, no dependencies). The entire game (HTML, CSS, JavaScript) is contained in `index.html` (~1669 lines).

## Running the Game

Simply open `index.html` in a web browser. No build, server, or dependencies required. The game is fully playable on desktop (mouse/keyboard) and mobile (touch).

- Desktop controls: mouse moves, click or Space to fire, X/Shift for bomb, P/Esc to pause
- Mobile: left side drag to move, tap FIRE button, tap BOMB button
- Difficulty can be changed from the start menu (Easy/Normal/Hard)

## Architecture & Key Systems

### Game Loop & Performance

The game runs via `requestAnimationFrame` with a timestep-based update model (capped at 50ms per frame). An adaptive quality system (`quality` variable) automatically scales down particles, shadows, and effects on slower devices to maintain 40+ FPS.

- `trackPerf(dt)` monitors frame times and adjusts quality (0=full, 1=medium, 2=low)
- Game state updates in `update(dt)`, rendering in `draw()`
- Canvas is 480×720 logical pixels, scaled responsively to fit the window

### Game State & Flow

Game state variables are global in the IIFE:
- `player` — position, weapon type/level, health, invulnerability timer
- `enemies`, `enemyBullets`, `bullets`, `particles`, `powerUps` — managed as arrays
- `boss` — null when inactive, spawned after 18 enemy kills per stage
- `score`, `lives`, `stage`, `chain`, `multiplier` — progression and scoring
- `running`, `paused`, `gameOver` — game phase flags

Key flow:
1. Start → show overlay with difficulty selection
2. `resetGame()` initializes state and starts the loop
3. `update()` advances enemies, bullets, collision detection, stage progression
4. Boss spawns automatically after `KILLS_PER_STAGE` (18) enemies are killed
5. Stage advances after boss defeat; difficulty ramps over time
6. Game ends when lives reach 0

### Collision & Damage Model

- `rectsOverlap(a, b)` performs AABB collision detection
- Player bullets deal 1 HP to enemies (3 HP to bosses)
- Laser weapons have piercing (set via `pierce` property); regular bullets don't
- Smart bombs instantly kill all visible enemies/projectiles and damage boss
- Enemy contact, bullets, or boss contact cause 1 life loss (with invulnerability period)

### Weapon System

Two weapon types, each with 3 levels:

**Vulcan** (default):
- Level 1: single center shot
- Level 2: dual spread (±0.12 angle)
- Level 3: triple spread (±0.24 angle)

**Laser**:
- Level 1: single lane
- Level 2: dual lanes (±6 pixels offset)
- Level 3: three lanes (two sides + center)
- All shots have pierce mechanic; laser has faster fire rate (0.1s vs 0.15s)

Power-ups randomly spawn on enemy death: life (+1 health), bomb (+1 smart bomb), weapon (level up or swap weapon/reset level).

### Enemy Types

Defined in `ENEMY_TYPES`:
- `grunt` (red, 34×34): 1 HP, baseline speed, score 10, appears from stage 1
- `interceptor` (orange, 26×26): 1 HP, 1.7× speed with weaving, score 15, appears from stage 1
- `tank` (purple, 46×46): 4 HP, 0.55× speed, score 35, appears from stage 2+

Spawn behavior:
- `spawnEnemy()` picks a random type based on stage availability
- ~40% chance of interceptor, ~15% chance of tank (when available), else grunt
- Pattern is either "spread" (3-way bullet fan) or "aimed" (at player)
- Spawn rate increases over time via `difficultyRamp`

### Boss System

Six boss designs in `BOSSES` array (cycled per stage):
1. VANGUARD — radial burst, aimed fan, V-fan
2. BEHEMOTH — heavy columns + homing spread + dense radial
3. SPECTER — teleport dash + tight spiral + dual aimed
4. DREADNOUGHT — 8-way cross + homing missiles + dense radial
5. WRAITH — teleport barrage + double ring + cross-spread
6. TITAN — heavy columns + full ring + relentless homing

Each has unique attack patterns in `BOSS_PATTERNS`. Boss HP scales with `stage` and `difficulty` via multipliers. Bosses have a 22-frame telegraph before each attack pattern fires.

### Audio (Web Audio Synthesis)

No external files — all SFX are synthesized using Web Audio API:
- `ensureAudio()` initializes AudioContext on first user interaction
- `beep(freq, dur, type, vol, opts)` creates pitched tones with optional sweeps/filters
- `noiseBurst(dur, vol, opts)` creates filtered noise for explosions
- `sfx` object contains named effects: `shoot`, `laser`, `explode*`, `hit`, `bomb`, `powerup`, `bossHit`, `bossTelegraph`, `bossDown`, `stageClear`, `uiClick`, `gameover`
- Mute state persists in localStorage

### Input Handling

- **Keyboard**: ArrowKeys/WASD move, Space fires, X/Shift bombs, P/Esc pauses
- **Mouse**: position targets player; click or move fires
- **Touch**: left-side drag zone for movement, right-side buttons for fire/bomb (joystick rendered if needed)
- Mouse and keyboard/joystick input are blended; keyboard/joystick takes priority when active

### Visual Themes

Six stage themes in `THEMES` array cycle with stage progression. Each theme defines a sky gradient, cloud color/appearance, and sun glow. Themes change every 6 stages.

### Screen Effects

- `screenShake(mag, dur)` — camera shake (reduced via reduced-motion setting)
- `flashHit()` / `flashBomb()` — white flash overlay
- `showComboPop(text)` — floating combo milestone text (10 kills, 20 kills, etc.)
- `showStageBanner(text)` — stage transition banner
- Reduced motion respects `prefers-reduced-motion` and can be toggled in-game

### Difficulty Settings

`DIFFICULTIES` object scales:
- `spawnBase` / `spawnMin` — enemy spawn rate (tighter on hard)
- `enemySpeed` — enemy velocity multiplier
- `enemyShootChance` — % chance enemy fires back
- `lives` — starting lives
- `bombs` — max smart bombs
- `bossHpMul` — boss health multiplier

### Scoring & Chain System

- Base score for each enemy type (grunt 10, interceptor 15, tank 35, boss 500)
- Multiplier increases by 1 for every 5 consecutive enemy kills (caps at 10×)
- Chain resets if no kills for 2.2 seconds
- Kills milestone (every 10) shows combo pop-up

### Persistent State

Stored in localStorage:
- `skyfighter.bestScore` — high score
- `skyfighter.muted` — mute toggle
- `skyfighter.difficulty` — selected difficulty
- `skyfighter.reducedMotion` — motion preferences

## Key Areas for Modification

### Game Balance

- **Spawn rate**: Adjust `spawnBase` / `spawnMin` in `DIFFICULTIES`
- **Enemy behavior**: Edit `ENEMY_TYPES` (hp, speed, score) or behavior in `spawnEnemy()`
- **Weapon power**: Edit `fireWeapon()` spread patterns and bullet speeds
- **Boss health**: Modify baseHp formula in `spawnBoss()` and `hpMul` in `BOSSES`
- **Boss patterns**: Add/modify patterns in `BOSS_PATTERNS` array
- **Stage progression**: Change `KILLS_PER_STAGE` to tune difficulty curve
- **Scoring**: Edit `score` value in `addScore()` or per-enemy in types

### Visual Polish

- **Colors**: Enemy colors in `ENEMY_TYPES`, boss colors in `BOSSES`, theme sky/cloud/sun in `THEMES`
- **Canvas size**: Change `BASE_W` and `BASE_H` (logically; scaling is automatic)
- **Particle effects**: Adjust spawn count and color in `spawnExplosion()`
- **Animation speed**: Frame-rate calculations are in `frame` increments (lower = slower)

### Audio Tuning

- Tweak frequencies, durations, and volumes in `sfx` object
- Adjust filter frequencies and types for different tones
- Change pan/spatial audio in effect definitions

### Input/Controls

- Joystick deadzone: change `maxDist` in `joystickMove()` (currently 50)
- Key bindings: modify `keydown`/`keyup` event listeners
- Mouse steering interpolation: adjust `player.speed * 1.7` in `update()`

## Performance Optimization Notes

1. **Adaptive quality** already scales particle count and shadow effects — tweak thresholds in `trackPerf()` if needed
2. **Collision detection** uses AABB; no spatial partitioning (acceptable for this scope)
3. **Particle pooling** not used; particles are created/destroyed freely. For larger counts, implement an object pool
4. **Canvas clipping** is not used; all drawing happens every frame. Consider clipping for complex scenes
5. **Touch event listeners** use `{ passive: false }` to enable `preventDefault()`; necessary for correct touch handling

## Testing Notes

- No automated test suite exists (this is a single-file game, not a library)
- Manual testing: verify each difficulty level, all weapon types, all boss patterns
- Mobile: test on various screen sizes and touch devices
- Accessibility: screen reader announcements via `announce()` function; reduced motion respected

## Common Edits Checklist

Before pushing changes:
1. Test on both desktop (mouse/keyboard) and mobile (touch) if controls changed
2. Verify difficulty balance by playing through 2-3 stages on each difficulty
3. Check audio SFX don't clip or distort (Web Audio gain levels)
4. Ensure no touch listeners leak (always clean up on `touchend`/`touchcancel`)
5. Verify localStorage keys don't conflict with other projects
6. Test on a slower device if visual changes were made (quality scaling)
