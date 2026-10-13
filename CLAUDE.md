# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game, fully self-contained in a single HTML file (`index.html`). It's a zero-dependency game that runs client-side with procedurally generated audio (WebAudio synth, no external assets).

## Quick Start

- **Open the game**: Simply open `index.html` in a web browser
- **No build system**: This is a standalone HTML file. No npm/yarn/build steps needed
- **Run locally**: `python3 -m http.server 8000` then visit `http://localhost:8000`

## Code Architecture

The game follows a classic arcade game structure: **Game State → Update → Draw → Loop**.

### High-Level Flow

1. **Initialization** (lines 421-527)
   - Grab DOM elements and set up listeners
   - Load persisted game state from localStorage
   - Initialize audio context (lazily, on first user interaction)

2. **Input Handling** (lines 767-861)
   - **Keyboard**: Arrow keys, WASD for movement; Space/click for fire; X/Shift/P for bomb/pause
   - **Touch**: Joystick on left, Fire/Bomb buttons on right
   - **Mouse**: Move player toward cursor (auto-aim while keyboard/joystick inactive)
   - Input is gathered into global state (`keys`, `joyVec`, `mouseTarget`, `firing`)

3. **Game Loop** (lines 1640-1665)
   - `requestAnimationFrame(loop)` → calculate delta time → `update(dt)` → `draw()` → repeat
   - Adaptive frame timing: clamps dt to 50ms to handle frame drops gracefully
   - Frame counter increments every update (used for animations/patterns)

4. **Update Phase** (lines 1148-1302)
   - Move player based on input (keyboard/joystick priority over mouse)
   - Process weapon fire cooldown and spawn bullets
   - Move all bullets, particles, enemies, powerups
   - Spawn regular enemies on a diminishing timer (harder as time passes)
   - Trigger boss spawn when 18 kills reached
   - Collision detection: bullets ↔ enemies, bullets ↔ boss, enemies ↔ player, bullets ↔ player
   - Update boss attack patterns
   - Decay combo chain timer; reset combo if expired

5. **Draw Phase** (lines 1520-1637)
   - Apply optional screen shake effect
   - Render gradient sky background (theme-based)
   - Draw clouds (parallax layers)
   - Render player, enemies, bullets, explosions, powerups
   - Boss rendering with animated glowing core

### Entity Types & Key Data Structures

**Player** (`lines 876-877`)
- `{ x, y, w, h, speed, cooldown, rapid, weapon, level, invuln }`
- Weapon choices: 'vulcan' (spread shots) or 'laser' (piercing, narrower)
- Level 1–3 increases spread/pierce count; level 4 switches weapon

**Enemies** (lines 1032-1040)
- Three types: `grunt` (weak), `interceptor` (fast), `tank` (tanky)
- Each has: `hp`, `speed`, `color`, `pattern` (spread or aimed fire), `weave` (left-right drift)
- Spawn stage-locked (tank only from stage 2)
- Shoot with varying cooldowns; hit timings randomized to feel natural

**Bullets & Projectiles**
- Player bullets: `{ x, y, w, h, speed, vx, pierce, laser? }`
- Enemy bullets: `{ x, y, w, h, speed, vx?, radial?, dx?, dy? }`
- Radial bullets have velocity components (dx, dy) for pattern attacks

**Boss** (lines 1043-1055)
- State: entering (sliding in), phaseTimer, attackTimer, telegraph (flash before attack)
- 6 boss designs, each with unique attack pattern set (lines 1372-1409)
- Each boss has hpMul (difficulty-adjusted HP multiplier)
- Bosses bounce side-to-side; telegraph for ~22 frames before firing

**Particles** (line 1076-1084)
- Explosion fragments: `{ x, y, vx, vy, life, maxLife, color }`
- Spawned on enemy/boss death or player hit
- Decays alpha over lifetime

### Difficulty & Progression

**Three difficulties** (lines 485-489):
- **Easy**: Slower spawn, lower shoot chance, lower HP
- **Normal**: Balanced
- **Hard**: Aggressive spawn, high shoot chance, higher HP

**Stage progression**:
- 18 kills → Boss spawn warning + delay → Boss appears
- Boss defeat → Stage++, return to regular enemies
- Themes cycle through 6 sky/cloud color schemes per stage
- Boss templates cycle through 6 designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)

### Performance Adaptation (lines 473-483)

Adaptive quality system tracks frame times:
- `quality = 0` (full): Glow effects, more particles, more visual polish
- `quality = 1` (medium): Reduced particle count
- `quality = 2` (low): Minimal particles, no glow

Automatically steps quality down if average frame time > 25ms, up if < 17.8ms.

### Audio System (lines 542-676)

**WebAudio synth** generates all SFX:
- `beep(freq, dur, type, vol, opts)`: Oscillator with sweep, filter, pan
- `noiseBurst(dur, vol, opts)`: White noise with filter
- 15 named SFX effects (shoot, laser, explode variants, bomb, powerup, boss hits, stage clear, etc.)
- **No external files**; all generated procedurally
- Stereo panning based on screen position (x-coordinate)
- Master gain + dynamics compressor for cohesive mix

### UI & Accessibility

**HUD** (lines 367-373): Score, best score, multiplier, weapon name/level, health hearts, bomb count

**Visual Feedback**:
- Vignette flash when health critical (1 life left)
- White flash on hit, bomb-flash on bomb detonated
- Screen shake (reduced-motion mode dampens this)
- Combo popup (every 10 kills) and stage banner animations
- Boss health bar appears when boss active

**Reduced Motion Support** (lines 520-540):
- Respects `prefers-reduced-motion: reduce` media query
- Stored in localStorage (user can toggle)
- Disables animations, reduces screen shake, removes filter effects

**Touch Controls** (lines 778-835):
- Left side: joystick for movement
- Right side: FIRE and BOMB buttons
- Auto-detects touch device and shows/hides controls

### Game State & Persistence

**Stored in localStorage**:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Sound on/off
- `skyfighter.difficulty`: Selected difficulty level
- `skyfighter.reducedMotion`: Reduced motion preference

## Modifying the Game

### Adding an Enemy Type

1. Add entry to `ENEMY_TYPES` (around line 502) with stats: `{ w, h, hp, speedMul, color, dark, score, minStage }`
2. (Optional) Add a new explosion SFX in `EXPLODE_SFX` mapping
3. Enemies spawn proportionally via the roll in `spawnEnemy()` (line 1028)

### Adding a Boss Attack Pattern

1. Write a function that fires enemy bullets (see `fireRadial`, `fireAimedSpread`, etc., lines 1324-1370)
2. Add a pattern array to `BOSS_PATTERNS` (line 1372)
3. Assign the array to a new BOSS entry in `BOSSES` list (line 508)

### Tweaking Difficulty

- Adjust `DIFFICULTIES` table (line 485): spawnBase, spawnMin, enemySpeed, enemyShootChance, lives, bombs, bossHpMul
- Adjust stage progression: change `KILLS_PER_STAGE` (line 491)

### Tuning Weapon Balance

- Vulcan: Adjust spread angles (line 1127), fire rate is `fireDelay` (line 1177)
- Laser: Adjust number of lanes (line 1136), pierce count, fire rate

### Visual Tweaking

- **Sky gradients & clouds**: Edit `THEMES` (line 493)
- **Screen shake**: `screenShake(magnitude, duration)` (line 727)
- **Particle counts**: Adjust `spawnExplosion()` particle count formula (line 1074)
- **Boss telegraph flash**: Change `telegraphing` logic in `drawBossShip()` (line 998)

## Browser Support

- Modern browsers with ES6 support (const, arrow functions, template literals)
- WebAudio API required for sound (gracefully silences if unavailable)
- Canvas 2D API for rendering
- localStorage for persistence
- Touch Events API for mobile (fallback to keyboard/mouse on desktop)

## Common Tasks

**Debug enemy spawning**: Check `spawnTimer` in update loop, enemy distribution weights in `spawnEnemy()`

**Adjust game difficulty curve**: Modify `DIFFICULTIES` and the spawn-timer ramp formula (line 1201)

**Test a specific boss**: Manually set `stage` and call `spawnBoss()` from DevTools console

**Profile performance**: DevTools Performance tab → record → play game → check frame times and quality adaptation

**Disable a feature**: Audio mute button is built-in. Reduced motion, pause, and difficulty selection are all user-facing toggles.
