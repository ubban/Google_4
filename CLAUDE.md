# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single self-contained HTML5 file. The entire game—including CSS styling, canvas rendering, game logic, and audio synthesis—is embedded in `index.html`. No build tools, bundlers, or external dependencies are required.

## Running the Game

- **Local development**: Open `index.html` directly in a web browser
- **Web server (optional)**: Serve the file over HTTP for consistency with deployment
- **Mobile**: Fully playable on touch devices with on-screen joystick and fire buttons

No build step, lint, or test commands exist. The game runs as-is in any modern browser.

## Code Architecture

The entire codebase is organized within a single IIFE (Immediately Invoked Function Expression) that starts at line 423. Key subsystems:

### Game State & Objects
- **Player**: Canvas position/velocity, current weapon, health (hearts), bomb count
- **Enemies**: Array of `{ x, y, w, h, hp, speed, type, ... }` objects with different patterns (weaving, shooting)
- **Bullets**: Player bullets (player-fired) and enemy bullets, tracked separately
- **Boss**: Single boss object (if active) with dedicated health bar and AI patterns
- **Power-ups**: Weapon upgrades, health, bombs—spawn on enemy death and float downward
- **Particles**: Visual effects (explosions) with lifetime decay

### Core Game Loop
The game runs at 60 FPS via `requestAnimationFrame`. Entry point at line 1640 (`loop(now)`):
1. Calculate delta time (`dt`)
2. Update game state (`update(dt)`)
3. Render scene (`draw()`)
4. Collision detection and score updates

### Key Functions

**Rendering** (line 1520 `draw()`):
- Canvas background with parallax clouds
- Player and enemy sprites
- Bullets and particles
- Boss health bar and HUD (score, multiplier, weapon info, hearts, bombs)
- Flash/vignette effects for hits and bomb use

**Update Logic** (line 1148 `update(dt)`):
- Player movement (keyboard/touch joystick)
- Enemy spawning with difficulty ramp (every ~2 frames, ramping spawn rate)
- Enemy AI: movement weaving, shooting patterns, bullet physics
- Collision detection: bullets vs enemies/boss, enemies vs player, pickups vs player
- Score multiplier: breaks on taking damage, increases on kills
- Stage progression: spawn boss after 18 kills, advance to next stage

**Boss Battles** (line 1411 `updateBoss(dt)`):
- Boss entrance animation
- Firing patterns vary by boss type (4 boss variants)
- Boss health is 3x more resistant per hit than regular enemy
- Screen shake and vignette effects during boss fights

**Audio System** (line 545 onwards):
- Web Audio API synthesis (no external sound files)
- `beep()`: Tone generation with filters, panning, envelopes
- `noiseBurst()`: Procedural noise for explosions
- Sound effects: shoot, hit, explode, bomb, level-up, lose-life, boss-hit
- Mute button controls audio context suspension/resumption

**Input Handling**:
- Keyboard: Arrow keys for movement, Z/Space for fire, X/Shift for bomb
- Touch: Joystick in bottom-left, fire button in bottom-right
- Pause: Pause button stops updates but not rendering
- Button state tracked in objects: `keys`, `firing`, `bombHeld`

### Difficulty System
Three presets (Easy, Normal, Hard) at line 485 define:
- Enemy spawn rate (base, min intervals)
- Enemy bullet speed
- Starting lives and bombs
- Score multiplier bonus per stage

### Weapons
Five weapon types upgrade through pickups:
1. **Vulcan** (basic): Single-stream bullets, 3-pierce laser upgrade
2. **Spread**: 3-bullet cone pattern
3. **Aimed**: Adjusted bullets angled toward player position
4. **Cannon**: Slower, heavier projectiles with pierce
5. **Rail**: Wide arcs of bullets, heavy fire support

Boss fire patterns at line 1324-1369 adapt per boss type (radial, spiral, aimed spreads, homing, double rings).

### Player State Persistence
Uses browser `localStorage`:
- `skyfighter.bestScore`: Best score across sessions
- `skyfighter.muted`: Mute preference
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion toggle state

### Visual & Motion Effects
- **Reduced motion mode** (line 529): Disables screen shake, vignette pulses, flash animations when user requests reduced motion or via button toggle
- **Combo pop & stage banner**: Floating text animations for milestones
- **Explosions**: Particle system with spread angle and color-coded by source (enemy, boss, player)
- **Screen shake**: Magnitude-based rumble effect (disabled in reduced-motion mode)

## Modifying Game Behavior

**Adjust spawn rate / difficulty**: Line 1201—modify `spawnBase` and `spawnMin` in `DIFFICULTIES` object.

**Change weapon stats**: Lines 1124-1147 in `fireWeapon()`—bullet speed, count, and pattern.

**Add/modify bosses**: Line 508 defines `BOSSES` array. Each boss has `name`, `baseHp`, `color`, and firing function. Boss patterns are lines 1324-1369.

**Tweak enemy types**: Line 502 `ENEMY_TYPES` defines stats (hp, score, type, shooting pattern). Spawn logic at line 1022.

**Audio adjustments**: Web Audio parameters in `beep()` and `noiseBurst()` (frequency, duration, gain).

**UI styling**: CSS grid, flexbox, and animation keyframes (lines 10-421). Responsive design uses `clamp()` for scaling across devices.

## Accessibility & Mobile

- **ARIA labels**: HUD elements and buttons have proper roles for screen readers
- **Keyboard nav**: All buttons keyboard-accessible; game state announced via `srLive` element
- **Touch controls**: On-screen joystick for movement, fire and bomb buttons
- **Responsive canvas**: Resizes to fit window/device; safe area insets for notched phones
- **Reduced motion**: Respects `prefers-reduced-motion` media query; button to toggle

## Browser Compatibility

Requires HTML5 canvas, Web Audio API, and ES6+ support (const, arrow functions, template literals). Tested on modern desktop and mobile browsers.
