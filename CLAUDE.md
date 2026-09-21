# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-'em-up game built entirely in HTML/CSS/JavaScript. The game runs in a web browser (index.html) with no external dependencies—all audio is synthesized using Web Audio API, not loaded from files.

## Architecture

The entire game implementation is in `index.html` (~1670 lines) organized as follows:

### Structure Sections
1. **HTML (lines 1-361)**: Canvas element + UI overlays + invisible touch controls + screen reader element
2. **CSS (lines 10-359)**: Game styling, animations (pulse, shake, flash, combo pop), responsive sizing, reduced-motion support
3. **JavaScript (lines 422-1667)**: Single IIFE containing all game logic

### Game Loop
- **Main loop**: `loop(now)` (line 1640) uses `requestAnimationFrame`
- **Per-frame flow**: `trackPerf()` → (if running) `update(dt)` → `draw()`
- **Time delta**: Clamped to 0.05s to prevent physics jumps on frame skips

### Key Game Systems

**Player State** (line 877):
- Position, velocity constraints, weapon system (vulcan/laser), cooldown timers
- Invulnerability frames after hits

**Enemy System**:
- **Types** (line 502-506): `grunt` (fast), `interceptor` (very fast), `tank` (slow, multi-hit)
- **Spawning** (line 1022-1041): Random position at top, weaving patterns, shoot AI
- **Difficulty scaling**: Spawn rate and enemy speed increase per-stage
- **Progression**: Stage advances after 18 kills (KILLS_PER_STAGE = 18, line 491); boss spawns after that

**Boss System** (line 1043-1410):
- 6 boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with unique attack patterns
- Each boss has HP multiplier, color scheme, and 3-pattern attack cycle
- Patterns include: radial bursts, aimed spreads, homing seeded shots, telegraphed attacks
- Boss health bar updates in real-time as damage is taken

**Weapons** (line 1124-1146):
- **Vulcan** (default): Spread shot that improves with level (1/2/3 bullets), fast fire rate
- **Laser**: Piercing shots, higher damage vs bosses, slower fire rate, glowing aura
- Weapon cycles at level 3: next power-up switches weapon and resets level to 1

**Scoring & Multiplier** (line 1087-1101):
- Kill streak extends "chain" (up to 10× multiplier), resets on taking damage
- Chain timer: 2.2 seconds per kill—must maintain steady fire rate
- Bomb kills also count toward chain (critical for high scores)

**Audio** (line 542-676):
- No asset files—all sound effects are synthesized in-game using Web Audio API
- Master gain with compressor, per-effect envelope generation
- SFX categories: shoot/laser, explosions, hits, powerups, boss audio, UI clicks
- Disabled when muted (localStorage flag)

**Input System**:
- **Keyboard** (line 768-776): Arrow keys / WASD for movement, Space to fire, X/Shift for bomb, P/Esc for pause
- **Mouse** (line 836-862): Plane follows mouse pointer when over canvas; click to fire
- **Touch** (line 778-835): Left side—virtual joystick; Right side—FIRE/BOMB buttons; auto-enables on touch devices

**Difficulty** (line 485-489):
- Easy: Slower spawns, 4 lives, 3 bombs, boss HP 0.8×
- Normal: Medium challenge, 3 lives, 2 bombs, boss HP 1.0×
- Hard: Aggressive spawns, 3 lives, 2 bombs, boss HP 1.3×
- Selected at title screen, saved in localStorage

**Performance Scaling** (line 473-483):
- Adaptive "quality" mode (0=full, 1=medium, 2=low) based on frame time
- Reduces particle/bullet count, glow effects when needed
- Tracks last 45 frames; adjusts if avg > 25ms or < 17.8ms

**Visual Themes** (line 493-500):
- 6 gradient sky themes: Daylight, Sunset, Night, Storm, Arctic, Volcanic
- Theme cycles per stage; affects sky gradient, cloud color, sun color

## Key Constants & Globals

| Constant | Value | Purpose |
|----------|-------|---------|
| `BASE_W`, `BASE_H` | 480, 720 | Canvas internal resolution |
| `KILLS_PER_STAGE` | 18 | Enemies killed before boss warning |
| `DIFFICULTIES` | object | Spawn rates, speeds, lives, bomb counts per difficulty |
| `ENEMY_TYPES` | object | Enemy properties (size, HP, speed, score) |
| `BOSSES` | array[6] | Boss color schemes and HP multipliers |
| `BOSS_PATTERNS` | array[6] | Boss attack functions per design |
| `THEMES` | array[6] | Sky gradients and cloud colors |
| `STORAGE_*` | strings | localStorage keys for best score, mute state, difficulty, reduced motion |

## Collision Detection

- **Simple AABB**: `rectsOverlap(a, b)` (line 1069-1071) checks center-based rectangle overlap
- **Checks**:
  - Player bullets vs enemies (damage/kill)
  - Player bullets vs boss (damage/kill)
  - Enemies vs player (damage)
  - Enemy bullets vs player (damage)
  - Boss vs player (damage)
  - Power-ups vs player (pickup)

## Particle System

- Spawned on enemy death/boss hit (`spawnExplosion`, line 1073-1085)
- Particles: random velocity scatter, gravity-free, lifetime-based fade
- Count scales with quality setting (full=14, medium=8, low=5)

## Rendering Pipeline

1. **Clear canvas** + apply screen shake translation
2. **Sky**: Gradient fill based on theme
3. **Sun glow**: Radial gradient overlay
4. **Clouds**: Two layered scrolling cloud patterns at different speeds
5. **Player**: Plane sprite + laser aura (if using laser weapon) + invuln blink
6. **Bullets**: Glow effect (if quality=0), yellow (vulcan) or cyan (laser)
7. **Enemy bullets**: Red circles with optional glow
8. **Enemies**: Plane sprites with tank HP bar
9. **Boss**: Complex multi-part ship design with animated core
10. **Power-ups**: Rotating colored squares
11. **Particles**: Fading circles with trail effect

## Extending the Game

### Add a New Boss
1. Add entry to `BOSSES` array (line 508-515) with hull/wing/core colors and `hpMul`
2. Add attack pattern function to corresponding index in `BOSS_PATTERNS` (line 1372-1409)
3. Pattern function receives boss object `b`, fires using helpers: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.

### Add a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502-506) with w/h/hp/color/score/minStage
2. Update spawn logic (line 1022-1041) if needed
3. Add explosion SFX mapping if needed (line 678, `EXPLODE_SFX`)

### Add a New Sound Effect
1. Add function to `sfx` object (line 624-676) using `beep()` or `noiseBurst()` helpers
2. `beep(freq, dur, type, vol, opts)` creates oscillator-based tone with envelope, optional sweep/filter/pan
3. `noiseBurst(dur, vol, opts)` creates noise with filter—useful for explosions
4. Call function in appropriate game event

### Modify Difficulty
Edit `DIFFICULTIES` object (line 485-489): `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`

### Add a Weapon Type
1. Add weapon name to weapon cycle logic (line 1311-1317, `applyPowerUp`)
2. Add firing logic to `fireWeapon()` (line 1124-1146)
3. Add weapon name + level display updates (weaponNameEl, weaponLvlEl)

## Important Quirks & Implementation Details

- **Single-file constraint**: All code lives in index.html. Minification/splitting not needed for this scope.
- **Canvas coordinate origin**: Top-left (0,0). Bullets move up (negative Y), enemies move down (positive Y).
- **Frame-based animations**: Most animations key off `frame` counter (incremented every update). Useful for predictable cycling.
- **Touch detection**: `('ontouchstart' in window)` enables virtual joystick. Joystick position in moveZone (55% left/bottom of screen).
- **Audio initialization**: Web Audio Context requires user interaction first. `ensureAudio()` creates it on first input.
- **Invulnerability timer**: 1.5 seconds post-hit; blinks every 4 frames (`Math.floor(frame / 4) % 2`).
- **Chain/combo system**: Breaks on player damage, not enemy damage. Timing window: 2.2 seconds between kills. Multiplier caps at 10×.
- **Boss telegraph**: 22 frames (line 1433) of warning flash before attack pattern fires, gives player reaction time.
- **Radial bullets**: Enemy bullets fired by boss use `dx/dy` velocity, not angle. Updates in `updateBoss()` (line 1437).
- **Power-up probability**: 6% life, 8% bomb, 20% weapon, 66% nothing (line 1058-1067).

## Testing & Debugging

- **Best score**: Persisted in localStorage. Clear console storage to reset: `localStorage.clear()`
- **Frame info**: `frame` counter available in browser console during gameplay
- **Quality level**: `quality` variable (0/1/2) adjusts particle count/glow effects; auto-adjusts on frame rate changes
- **Mute/Reduced Motion**: Toggle via UI buttons (top-right 🔊 and 🎆); persisted in localStorage
- **Pause**: Press P or Esc, or click ⏸ button (persists game state, no reset)

## Browser Requirements

- Canvas 2D context (universal support)
- Web Audio API (for synthesized SFX; graceful fallback if unavailable)
- localStorage (for persistent settings)
- requestAnimationFrame (modern browsers)
- Touch events + pointer tracking (mobile support)
- Recommended: Chrome, Firefox, Safari (recent versions)
