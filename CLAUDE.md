# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. It features stage progression, multiple boss encounters, weapon systems, difficulty levels, and Web Audio API synthesis for sound effects.

## Development & Execution

### Opening the Game
```bash
# Open directly in a browser (no server needed for local development)
open index.html
# Or serve locally to test on devices:
npx http-server
```

### No Build Process
This is a single-file game with no build step. All CSS and JavaScript are embedded in `index.html`. Changes to the file are immediately visible when reloaded in the browser.

### Testing Changes
1. Edit `index.html`
2. Reload the browser (Ctrl+R or Cmd+R)
3. Test gameplay, canvas rendering, audio, and UI responsiveness

## Architecture

The game is organized within a single IIFE (Immediately Invoked Function Expression) in the `<script>` block, starting at line 423. Everything is scoped to avoid global pollution.

### Core Game Loop
- **Frame-based**: Uses `requestAnimationFrame` for 60fps target
- **Delta time**: `dt` is passed through update functions for frame-independent movement
- **Rendering**: Canvas 2D API with dynamic sizing (responsive)
- **Performance**: Adaptive quality system (lines 473–483) throttles particle/effect count on slow devices

### Entity Systems

**Player** (`player` object)
- Position, velocity, weapons, health, invulnerability frames
- Follows mouse/touch input with smooth acceleration
- Two weapon types: Vulcan (rapid fire) and Laser (slower, more powerful)
- Weapon levels increase damage; at max level, weapon type toggles instead

**Enemies** (spawned via `spawnEnemy()`, lines 1022–1041)
- Three types: Grunt, Interceptor, Tank
- Each type has min stage, speed multiplier, HP, color, and score value
- Defined in `ENEMY_TYPES` constant (line 502)
- Shoot with difficulty-adjusted probability; patterns include radial and aimed fire

**Boss** (`boss` object)
- Six designs in `BOSSES` array (line 508): Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan
- Each boss has unique colors, HP scaling, and attack pattern set
- Enters from top, bounces horizontally, uses telegraph system (red warning before attack)
- Attack patterns in `BOSS_PATTERNS` (line 1225+): radial bursts, homing, spirals, aimed spreads, etc.

**Projectiles**
- Player bullets: tracked in `playerBullets` array
- Enemy bullets: tracked in `enemyBullets` array
- Collision detection via `rectsOverlap()` function (line 1089)
- Enemy bullets include `radial` flag for movement pattern

**Particles**
- Explosions, on-hit effects: stored in `particles` array
- Life-based decay with velocity
- Spawned by `spawnExplosion()` and `spawnExplosion()` calls

### Game State

**Difficulty** (line 485–488)
- Three levels: easy, normal, hard
- Each adjusts: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP
- Stored in localStorage; selected via radio buttons on start screen

**Stages & Themes** (line 493–500)
- Six themes with unique sky gradients, cloud colors, sun tints
- Themes cycle as stages progress
- 18 enemy kills required per stage (`KILLS_PER_STAGE`, line 491) before boss spawns
- Boss spawn triggers `updateBoss()` phase

**Scoring & Combo** (lines 883–920)
- Score increases on enemy kill; multiplier from combo chain
- Combo resets if player hit; displays via `showComboPop()`
- Best score stored in localStorage

**Lives, Bombs, Weapons**
- Lives restored by life power-ups; max determined by difficulty
- Bombs stored in `bombCount`; used for screen-clear attack
- Weapon and level affect fire rate and damage

### Audio System

**Web Audio Synthesis** (no external assets)
- `ensureAudio()` (line 545): Initialize AudioContext on first user interaction
- `beep()` function (line 566): Creates tones with sweep, filter, panning
- `noiseBurst()` function (line 597): Procedural noise for explosions
- `sfx` object (line 624): Named sounds for shoot, explosion, powerup, boss telegraph, etc.
- Master gain with dynamics compressor for balanced mix

### UI & HUD

**On-Canvas HUD** (positioned absolutely, lines 34–64 in CSS)
- Lives (❤️), bomb count (💣), weapon type and level
- Score, multiplier, best score
- Boss health bar (appears when boss active)
- Combo popup (animates up-center on kill chain)
- Stage banner (full-screen announcement on new stage)
- Vignette effect (red border when 1 life left)
- Flash overlay (white flashes on hit/bomb)

**Controls Panel** (top-right buttons, line 180+)
- Mute toggle (🔇/🔊)
- Motion reduction toggle (🎆/🚫) — disables shake and flash effects
- Pause button (⏸)

**Touch Controls** (lines 319–380)
- On mobile: Virtual joystick for movement, fire button, bomb button
- Joystick base fixed; stick rotates/scales with input
- Buttons are large touch targets for game play

**Game Over Screen** (overlay)
- Final score, best score, final stage reached
- Difficulty radio buttons to select next game
- "Start Game" button to restart

### Storage Keys
```javascript
const STORAGE_BEST = 'skyfighter.bestScore';
const STORAGE_MUTE = 'skyfighter.muted';
const STORAGE_DIFF = 'skyfighter.difficulty';
const STORAGE_REDUCED = 'skyfighter.reducedMotion';
```

## Key Functions & Their Roles

### Game Flow
- `resetGame()` (line 875): Initialize/reset all entities, lives, score, and UI
- `update(dt)` (line 1148): Main update loop — moves entities, handles collisions, spawns enemies
- `draw()` (line 1462): Render canvas background, all entities, particles, and HUD elements
- `gameLoop(t)` (line 1507): requestAnimationFrame callback — manages timing and calls update/draw

### Collision & Damage
- `rectsOverlap(a, b)` (line 1089): AABB collision detection
- `loseLife()` (line 1290): Reduce lives, set invuln, update UI, trigger end-game if needed
- `killEnemy()` (line 1268): Remove enemy, score, spawn explosion, check for boss trigger
- `killBoss()` (line 1332): Boss death sequence, stage advance, next boss queue

### Spawning
- `spawnEnemy()` (line 1022): Pick random enemy type (respects minStage), spawn at random x, top
- `spawnBoss()` (line 1043): Create boss from current stage's design, set HP, show bar
- `spawnExplosion()` (line 1467): Particle burst and sound effect at position and color

### Player Input
- Mouse/touch move: `mousemove` listener (line 1516) updates `mouseTarget` position
- Shoot: Automatic when game active; controlled by `fireDelay` timer
- Bomb: Click/tap bomb button, trigger `useBomb()` (line 1258)
- Pause: Toggle game pause state

### Power-Ups
- `applyPowerUp(type)` (line 1194): Handle life, bomb, weapon power-up logic
- Types: `'life'`, `'bomb'`, `'weapon'` — spawned randomly after enemy kill
- Weapon power-ups upgrade level or cycle between weapon types at max level

## Common Development Tasks

### Adjusting Difficulty
Edit the `DIFFICULTIES` object (line 485–488). Parameters:
- `spawnBase` / `spawnMin`: Enemy spawn rate (lower = more frequent)
- `enemySpeed`: Multiplier applied to enemy velocity
- `enemyShootChance`: Probability each frame an enemy fires
- `lives` / `bombs`: Starting count
- `bossHpMul`: Scale factor applied to boss base HP

### Adding a New Boss Design
1. Add object to `BOSSES` array with name and colors (hull, wing, core)
2. Add attack pattern array to `BOSS_PATTERNS` at same index
3. Patterns are functions that call firing helpers: `fireRadial()`, `fireAimedSpread()`, `fireCross()`, etc.

### Adding a New Enemy Type
1. Add to `ENEMY_TYPES` object with stats (width, height, HP, speed, colors, score, minStage)
2. Add corresponding SFX to `EXPLODE_SFX` map (line 678)
3. Add SFX function to `sfx` object (e.g., `explodeNewType()`)
4. `spawnEnemy()` will automatically consider it based on minStage

### Tweaking Visuals & Animations
- Colors are inline in CSS (lines 1–420) and in drawing code (e.g., `drawEnemy()` at line 1409)
- Animations use CSS keyframes (@keyframes) or procedural JS in draw functions
- Canvas rendering in `draw()` function handles entity shapes and fills

### Sound Customization
Edit `sfx` object. Beep parameters:
- `freq`: Frequency in Hz
- `dur`: Duration in seconds
- `type`: Oscillator type ('sine', 'square', 'sawtooth', 'triangle')
- `vol`: Gain/volume (0–1)
- `opts`: { sweepTo, filterType, filterFreq, pan, delay }

## Performance Notes

- **Particle throttling**: Quality variable (0–2) reduces particle count on slow devices
- **Touch responsiveness**: Joystick updates at full frame rate; buttons use pointer events
- **Canvas scaling**: Uses `max-width: 100%; max-height: 100%` with `contain` for aspect ratio
- **Storage**: localStorage calls are minimal (only on setting change, not per frame)
- **Audio**: Web Audio context created on first user interaction (not upfront)

## Browser Support

- Modern browsers with Canvas 2D, requestAnimationFrame, Web Audio API
- Tested on desktop and mobile (iOS Safari, Android Chrome)
- Fallback: If Web Audio unavailable, game continues silently
- Touch controls enabled on devices with touch support

## File Structure

```
Google_4/
├── index.html          # Single-file game (HTML, CSS, JS)
├── README.md           # Project description
└── .git/               # Version control
```

All game code is in `index.html` between `<script>` tags (line 422–1670).
