# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in HTML5/Canvas with WebAudio synthesis. The complete game is self-contained in a single `index.html` file (~1,670 lines). It features:

- **Single-file architecture**: All HTML, CSS, and JavaScript in one file
- **No external dependencies**: Canvas rendering, synthesized sound effects (no audio files)
- **Responsive design**: Adapts to mobile and desktop; supports keyboard, mouse, and touch input
- **Progressive difficulty**: 6 stages with themed bosses and increasing enemy density
- **Procedural generation**: Enemy waves, power-ups, and boss encounters are algorithmically spawned
- **Performance optimization**: Adaptive quality system reduces particle effects on low-end devices

## High-Level Architecture

### Game Loop & State (`loop()`, `update()`, `draw()`)

The game runs on `requestAnimationFrame` with a fixed timestep simulation:
- `loop(now)` — Main animation frame handler; calculates delta time (capped at 50ms)
- `update(dt)` — Physics, AI, collision detection, and spawn logic
- `draw()` — Canvas rendering with screen-shake effects applied via `ctx.translate()`

**Key global state**:
- `running`, `paused`, `gameOver` — State flags
- `player` — Player object with position, velocity, weapon, invulnerability
- `enemies`, `boss` — Active enemy entities
- `bullets`, `enemyBullets` — Projectiles
- `powerUps` — Weapon/bomb drops and life pickups
- `particles` — Temporary visual effects
- `frame` — Global frame counter (incremented each update)

### Entity Systems

**Player** (`player` object)
- Properties: `x`, `y`, `w` (width), `h` (height), `vx`, `vy`, `weapon`, `weaponLvl`, `invuln`
- Input-driven movement; weapon firing rate scaled by weapon level
- Laser weapon (continuous beam, higher damage) vs. default weapon (spread shots)
- Invulnerability frames after taking damage

**Enemies** (spawned via `spawnEnemy()`)
- Types: `grunt` (fast), `interceptor` (agile), `tank` (armored)
- Difficulty settings scale spawn rate and enemy behavior
- `weave` property for interceptors adds sine-wave movement
- Each kill increments combo multiplier (resets on damage taken)
- Procedurally chosen at each spawn based on current stage

**Boss** (spawned via `spawnBoss()` when stage kills reach `KILLS_PER_STAGE`)
- 6 boss variants with unique colors and stat multipliers
- Boss health tied to difficulty multiplier (`bossHpMul`)
- Updated separately via `updateBoss(dt)` with attack patterns
- Boss bar rendered at top of screen showing health

**Projectiles**
- `bullets` — Player shots; can be laser (continuous) or normal (spread)
- `enemyBullets` — Simple red circles spawned by enemy ships
- Bullets removed when off-screen

**Power-ups** (spawned when enemies die)
- `life` — Restore 1 health (red)
- `bomb` — Add smart bomb (yellow)
- `weapon` — Upgrade current weapon or switch to laser (green)

### Difficulty System

Three difficulty modes in `DIFFICULTIES` object:
- **Easy**: Fewer enemies, slower speed, higher starting lives
- **Normal** (default): Balanced spawn/speed rates
- **Hard**: Dense spawning, faster enemies, higher health pools

Difficulty affects: spawn intervals, enemy speed multiplier, shoot chance, starting lives/bombs, boss HP

### Audio System

**WebAudio Synthesis** (no external audio files):
- `beep(freq, dur, type, vol, opts)` — Tone with optional sweep, filter, panning
- `noiseBurst(dur, vol, opts)` — Noise-based effects
- `sfx` object with named sound methods: `shoot()`, `hit()`, `bomb()`, `gameover()`, etc.
- Master gain and compressor chain for dynamic range control
- Muted state persisted to `localStorage`

**Key sound effects**:
- Weapon fire: square wave sweep
- Enemy hit: noise burst with filter
- Boss damage: lower frequency noise
- Game over: descending tone

### Input & Controls

**Keyboard**:
- Arrow keys or WASD: Move
- Space: Fire weapon
- X or Shift: Deploy smart bomb
- P or Esc: Pause

**Mouse**:
- Move to steer player
- Click to fire

**Touch** (mobile):
- Drag left side to move (joystick with visual feedback)
- Tap FIRE button (bottom right)
- Tap BOMB button (bottom right, if available)

Touch controls rendered in `#touchControls` div with joystick base and stick elements

### Rendering & Visual Effects

**Canvas Rendering Pipeline**:
1. Clear canvas and apply screen-shake translation
2. Draw sky gradient (theme-based, cycles per stage)
3. Draw sun glow (radial gradient)
4. Draw parallax clouds at two speeds
5. Draw player, enemies, boss
6. Draw bullets and effects
7. Draw particles (fade over lifetime)

**Visual Feedback**:
- Screen shake: Controlled by `screenShake(mag, dur)`, applied via `ctx.translate()`
- White flash on hit: `flashHit()` animates `#flash` element
- Red vignette pulse on critical health (below 2 lives)
- Combo pop-up text overlay (golden, scaling animation)
- Stage banner on stage start
- Boss health bar at top

### Theme System

Six themed stages that cycle (`THEMES` array):
1. Daylight Front (blue sky)
2. Sunset Corridor (purple/orange)
3. Night Approach (dark blue)
4. Storm Front (green)
5. Arctic Wastes (cyan)
6. Volcanic Ridge (red/orange)

Each theme has: sky gradient colors, cloud color, sun color, theme name

### Performance Optimization

**Adaptive Quality** (`quality` 0–2):
- Tracks frame time over 45 frames
- Reduces glow effects (shadow blur) on slower devices
- Increases quality when consistently fast
- Player can disable effects via motion preference (stored in `localStorage`)

## Game Flow

1. **Initialization**: Game starts paused with overlay showing difficulty selector and start button
2. **Stage Entry**: Player earns kills; stage counter increments with banner
3. **Enemy Spawning**: `spawnEnemy()` called on timer; density and enemy types depend on difficulty
4. **Boss Battle**: After `KILLS_PER_STAGE` kills, boss spawns via `spawnBoss()`
5. **Damage**: Player collision with enemy or enemy bullet triggers `loseLife()`
6. **Combo**: Successive kills without taking damage multiply score; resets on hit via `breakChain()`
7. **Game Over**: All lives lost; score compared to best; persisted to `localStorage`

## Development Notes

### Key Functions by Category

**Game State**:
- `resetGame()` — Initialize/reset all entities, lives, weapons, score
- `endGame()` — Handle game over; save best score
- `togglePause()` — Pause/resume with overlay UI

**Spawning & Updates**:
- `spawnEnemy()` — Pick random enemy type, set position/velocity
- `spawnBoss()` — Create boss with random variant
- `update(dt)` — Main game loop; updates all entities, handles collision
- `updateBoss(dt)` — Boss movement and attack patterns

**Rendering**:
- `draw()` — Main render function; clears canvas, draws all entities
- `drawPlane(x, y, w, h, color, dark, isPlayer)` — Draw ship sprite (procedural)
- `drawBossShip(boss)` — Draw boss sprite with unique colors
- `drawClouds(speed, fillColor, rx, ry)` — Parallax cloud rendering

**Audio & Feedback**:
- `beep()`, `noiseBurst()` — WebAudio sound synthesis
- `screenShake()` — Apply camera shake effect
- `flashHit()`, `flashBomb()` — Visual feedback animations
- `showComboPop()` — Render combo multiplier text

**Collision**:
- Collision detection is inline in `update()`; uses bounding box checks
- Enemy collision with player checks `Math.hypot()` distance
- Bullet hits checked via rectangle and circle overlaps

### File Structure

Single file: `index.html` (1,669 lines)
- Lines 1–420: HTML structure + CSS styling
- Lines 421–1,666: JavaScript game code in IIFE (Immediately Invoked Function Expression)

No external files; all assets (images, audio) are generated procedurally or embedded as data URIs.

### Modifying Game Content

**Change difficulty parameters**:
```javascript
const DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... }
};
```

**Add new enemy types**:
Add entry to `ENEMY_TYPES` object; update `spawnEnemy()` logic to reference new type

**Add new boss**:
Add entry to `BOSSES` array; boss choice is random from available list

**Adjust visual theme**:
Edit `THEMES` array entries or add new color gradients

**Balance gameplay**:
- `KILLS_PER_STAGE` — Enemies needed to trigger boss
- `DIFFICULTIES[mode].spawnBase` / `spawnMin` — Control enemy spawn density
- `player` initial properties — Player speed, damage, max lives

### Testing & Debugging

**Run the game**: Open `index.html` in a browser (no build step required)

**Inspect state**: Open browser DevTools console; access globals:
```javascript
player, enemies, boss, bullets, score, stage, lives
```

**Check performance**: Browser DevTools → Performance tab; the adaptive quality system logs nothing but responds to slow frames

**Test audio**: Click the mute button (🔊) to toggle sound; visual feedback persists

**Test mobile**: Use browser DevTools device emulation or test on actual device; touch controls appear automatically on mobile

**Modify mid-game** (for debugging): Pause and adjust globals in console; resume to test changes

### Common Development Tasks

**Add a new power-up type**:
1. Add type name to power-up spawn logic in `update()` under "spawn powerups"
2. Handle collection in player collision check
3. Render in `draw()` with unique color

**Change game balance**: Edit `DIFFICULTIES` object values and `KILLS_PER_STAGE`

**Add visual feedback**: Use `screenShake()`, `flashHit()`, `showComboPop()`, or particle emission

**Add sound effect**: Create function in `sfx` object calling `beep()` or `noiseBurst()`

**Extend boss behavior**: Modify `updateBoss()` to add new attack patterns or movement

## Attribution & Metadata

- **Built**: Single-file HTML5 canvas game
- **No external dependencies**: Canvas API, WebAudio API, localStorage
- **Browser support**: Modern browsers (ES6+ required)
- **Mobile**: Responsive design with touch joystick and button controls
