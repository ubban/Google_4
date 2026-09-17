# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML5 file (`index.html`). It requires no build system or external dependencies — just open the file in a browser to play.

## Running the Game

```bash
# Open the HTML file in your browser (local development)
# On Mac/Linux:
open index.html
# Or serve locally:
python3 -m http.server 8000
# Then visit http://localhost:8000/index.html
```

The game is fully self-contained with:
- Inline CSS for styling and animations
- Embedded JavaScript game logic (~1200 lines)
- Web Audio API for procedural sound effects (no audio files)
- Canvas rendering for all graphics

## Architecture & Code Structure

The entire game logic is wrapped in an IIFE (`(function () { 'use strict'; ... })()`) to avoid global scope pollution. The codebase is organized into logical sections:

### Core Game State
- **Player entity**: position, speed, health, weapon, cooldown, invulnerability frames
- **Entities**: arrays for bullets, enemies, enemy bullets, particles, power-ups, boss
- **Game variables**: score, lives, stage, spawn timer, combo chain, multiplier

### Game Loop
```
requestAnimationFrame(loop)
  → update(dt)          // Update game state
  → draw()              // Render to canvas
```

The loop runs every frame, with delta time (dt) clamped to 0.05s max to prevent huge jumps on lag. `shakeTime` handles screen shake effects.

### Key Subsystems

#### 1. **Rendering System** (`draw()`)
- Canvas 2D context with save/restore for state management
- Theme system: 6 different sky/cloud color schemes cycling by stage
- Entity rendering: planes (player/enemies), bullets, particles, boss ship
- Adaptive glowing effects based on quality level
- Visual feedback: screen shake, flash effects, vignette on low health

#### 2. **Entity System**
- **Enemies**: grunt, interceptor, tank (3 types with different stats)
- **Boss fights**: 6 unique boss designs with progressive difficulty
  - Each boss has a specific attack pattern set in `BOSS_PATTERNS`
  - Bosses telegraph attacks before firing (telegraph cooldown = 22 frames)
  - Multiple attack types: radial, aimed spread, columns, spirals, homing shots

#### 3. **Collision Detection**
Simple AABB overlap: `rectsOverlap(a, b)` checks if two rectangles intersect. Used for:
- Player-enemy collision → damage
- Player-bullet collision → damage
- Player-powerup collision → pickup
- Bullet-enemy collision → enemy HP depletion
- Bullet-boss collision → boss HP depletion

#### 4. **Audio System** (procedural synthesis)
- **WebAudio API**: creates oscillators, filters, and noise bursts at runtime
- No external audio files — all SFX synthesized on-the-fly
- `beep(freq, dur, type, vol, opts)`: single tone with frequency sweep and filtering
- `noiseBurst(dur, vol, opts)`: white noise with filter
- Spatial audio: `panFor(x)` pans sounds left/right based on screen position
- `ensureAudio()`: lazily initializes AudioContext (required for user interaction)
- Master gain + compressor for consistent loudness

#### 5. **Difficulty System**
Three difficulty levels with distinct spawn rates, enemy speed, shoot chance, starting lives/bombs:
- **Easy**: 75 frame spawn base, 1.0× enemy speed, 0.45 shoot chance, 4 lives, 3 bombs, 0.8× boss HP
- **Normal**: 58 frame spawn base, 1.4× enemy speed, 0.7 shoot chance, 3 lives, 2 bombs, 1.0× boss HP
- **Hard**: 40 frame spawn base, 1.9× enemy speed, 0.95 shoot chance, 3 lives, 2 bombs, 1.3× boss HP

Stored in localStorage per user preference.

#### 6. **Weapon System**
- **Vulcan**: rapid-fire bullets with spread pattern (level 1–3)
  - Level 1: single center stream
  - Level 2: dual stream
  - Level 3: triple spread
- **Laser**: fewer, faster piercing lasers (level 1–3)
  - Level 1: single center
  - Level 2: dual stream
  - Level 3: triple with center
- Weapon pickups cycle: Vulcan 1→2→3, then swap to Laser 1, etc.

#### 7. **Scoring & Combo System**
- Kill streak = `chain` (incremented on each kill)
- `multiplier = 1 + floor(chain / 5)` → max 10× at 50 kills
- Chain timer resets if player takes damage or waits 2.2s without killing
- Boss defeated: +500 points

#### 8. **Performance Adaptation**
Tracks frame time over 45 frames and adjusts rendering quality:
- **Quality 0** (full): full particle count, shadow effects on all objects
- **Quality 1** (medium): reduced particles (60%), some shadows
- **Quality 2** (low): minimal particles (35%), no shadows

Thresholds: upgrade if avg frame > 1/40s, downgrade if < 1/56s.

#### 9. **Input Handling**
- **Keyboard**: arrow keys or WASD to move, Space/X to fire/bomb, P/Esc to pause
- **Mouse**: move pointer to steer (glides smoothly), click to fire
- **Touch**: drag left area for joystick, tap right buttons for fire/bomb
- **Joystick priority**: keyboard/joystick input overrides mouse steering

#### 10. **Accessibility**
- Screen reader announcements via `announce()` (updates `#srLive` aria-live region)
- Reduced motion preference: respects `prefers-reduced-motion` CSS media query
- Toggleable reduced motion: disables screen shake, flash, and animations
- Stored in localStorage

#### 11. **Power-ups**
Three types drop from enemy kills:
- **Life** (6% drop): restore 1 HP
- **Bomb** (8% drop): restore 1 bomb
- **Weapon** (20% drop): upgrade weapon or swap weapon type

### Stage Progression
- **Kills per stage**: 18 regular enemies before boss spawns
- **Boss spawn warning**: brief delay (1.2s) with stage banner animation
- **Boss defeated**: spawn explosion, sound effect, +500 score, advance to next stage
- **Theme rotation**: 6 themes cycle with stage (same theme every 6 stages)

### Mobile/Responsive
- Canvas scales to fit window (max 1.6× base resolution)
- Safe area insets for notch devices
- Touch-friendly buttons (78px diameter)
- Adaptive font sizes using CSS `clamp()`

## Important Code Patterns

### Entity-Oriented Updates
All entities (player, enemies, bullets, powerups) follow this pattern:
```javascript
// Update position
entity.x += vx * dt;
entity.y += vy * dt;

// Clamp to canvas
entity.x = Math.max(entity.w / 2, Math.min(canvas.width - entity.w / 2, entity.x));

// Filter dead entities
entities = entities.filter(e => e.alive);
```

### Particle System
Spawned at collisions (`spawnExplosion`):
- Randomly scattered velocity in all directions
- Linear life decay
- Rendered as circles with size/opacity based on remaining life

### Boss Attack Patterns
Each boss has 3 attack patterns that rotate. Example pattern function:
```javascript
b => fireRadial(b, 14, 150)  // Fire 14 bullets in radial burst at speed 150
```

Patterns are looked up by: `BOSSES[stageIndex].designIndex` → `BOSS_PATTERNS[index]`

### Local Storage Keys
- `skyfighter.bestScore`: high score
- `skyfighter.muted`: audio mute state (1 or 0)
- `skyfighter.difficulty`: last chosen difficulty
- `skyfighter.reducedMotion`: reduced motion toggle

## Modifying the Game

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` filter and spawn logic if needed
3. Add collision effect sound to `EXPLODE_SFX` if unique explosion

### Adding a New Boss
1. Add entry to `BOSSES` array with `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Add 3 attack pattern functions to `BOSS_PATTERNS`
3. Increase `BOSSES.length` handling (loops modulo by length)

### Adding a New Difficulty
1. Add entry to `DIFFICULTIES` object with spawn/speed/shoot/lives/bomb/hpMul values
2. No UI changes needed (difficulty buttons are data-driven)

### Adding a New Weapon
1. Add weapon type check in `fireWeapon()` and weapon swap logic
2. Create bullet specs (spread, speed, pierce, etc.)
3. Add fire sound effect to `sfx` object

### Adding Sound Effects
Web Audio synth functions:
- `beep(freq, dur, type, vol, opts)`: tone with optional sweep/filter/pan/delay
- `noiseBurst(dur, vol, opts)`: noise with optional filter/pan
- Add to `sfx` object and call in game events

Example: `sfx.custom = () => beep(800, 0.1, 'square', 0.05);`

### Adding Visual Effects
1. Screen shake: `screenShake(magnitude, duration)` (respects reducedMotion)
2. Combo popup: `showComboPop(text)` with animation
3. Stage banner: `showStageBanner(text)` with animation
4. Flash effect: `flashHit()` or `flashBomb()`
5. Particles: `spawnExplosion(x, y, color, count)`

All disable animations if `reducedMotion` is true.

## Testing Checklist

- [ ] Game starts and loads overlay
- [ ] Difficulty buttons toggle (no game running)
- [ ] Player spawns at bottom center
- [ ] Enemies spawn and move down
- [ ] Bullets fire on click/space and move up
- [ ] Collision detection: enemy kills deal damage, enemy bullets damage player
- [ ] Power-ups drop and apply effects (life/bomb/weapon)
- [ ] Combo multiplier increases with kill streak
- [ ] Boss spawns after 18 kills per stage
- [ ] Boss telegraph animation before attacks
- [ ] Bomb clears all enemies and boss damage
- [ ] Screen shake and flash effects trigger on hit/bomb
- [ ] Player invulnerability frames work (can't rapid-die)
- [ ] Stage theme rotates every 6 stages
- [ ] Best score persists in localStorage
- [ ] Mute button toggles sound
- [ ] Reduced motion disables animations
- [ ] Pause/resume works (P key or button)
- [ ] Game over screen shows final score and stage
- [ ] Touch controls work on mobile (joystick + buttons)
- [ ] Screen scaling works on different viewport sizes
- [ ] Screen reader announcements work (Esc → pause uses `announce()`)

## Performance Notes

- Canvas size is fixed at 480×720 (logical), scaled by CSS to fit viewport
- Particle count adapts based on frame time (quality levels 0–2)
- No off-canvas drawing or complex compositing
- Simple AABB collision detection (fast)
- Web Audio buffers are small and reused
- No memory leaks: entities filtered after death, listeners added once at init
