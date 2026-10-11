# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented entirely in a single HTML file (`index.html`). The game features:

- **Self-contained**: All code, styling, and assets are in one file (no external dependencies)
- **Canvas-based rendering**: 2D graphics using HTML5 Canvas API
- **WebAudio synthesis**: Sound effects generated programmatically (no asset files)
- **Responsive design**: Works on desktop, tablet, and mobile with touch controls
- **Accessibility**: Reduced motion support, ARIA live regions for screen readers

## Architecture & Key Systems

### Game Loop (Lines 1640–1652)
The main `loop()` function runs on `requestAnimationFrame`, maintaining a 60 FPS target. It:
1. Computes delta time (`dt`) capped at 50ms
2. Calls `update(dt)` for game logic
3. Calls `draw()` for rendering
4. Tracks performance and adjusts quality dynamically

### Core Game Objects

**Player** (Lines 877): Ship with position, size, weapon state, cooldown, invulnerability timer
- **Weapons**: Vulcan (spreads more at higher levels) or Laser (pierce property, faster fire)
- **Movement**: Keyboard (WASD/Arrows), mouse steering, or joystick (touch)
- **Invulnerability**: After hit, player blinks for ~1.5s and ignores collisions

**Enemies** (Lines 1022–1041): Three types with distinct behaviors
- `grunt`: Red, basic enemies (1 HP)
- `interceptor`: Orange, fast weaving enemies (1 HP)
- `tank`: Purple, slow heavy tanks (4 HP, spawn at stage 2+)
- Each has optional shooting patterns (`spread` or `aimed`)

**Bosses** (Lines 508–515, 1372–1409): Six unique boss designs with stage-dependent spawning
- Boss patterns are arrays of firing functions; each pattern is triggered after a telegraph phase
- Bosses move side-to-side and telegraph attacks before firing
- Defeated when `hp <= 0`; dropping a bomb deals 24 damage

**Projectiles**: Bullets (player), enemyBullets (enemies), particles (explosions)
- Player bullets have `pierce` property for lasers
- Enemy bullets marked `radial: true` move via `dx/dy` instead of `speed`

**Power-ups** (Lines 1058–1067): Spawned on enemy death (6%, 8%, 20% chances)
- `life`: Restores one life (up to max)
- `bomb`: Restores one bomb
- `weapon`: Levels up weapon or swaps Vulcan ↔ Laser (resets level to 1)

### Collision Detection (Line 1069)
`rectsOverlap(a, b)` checks AABB collision using center distance and half-widths.

### Scoring & Multiplier System (Lines 1087–1101)
- `chain` increments on each kill; `chainTimer` resets to 2.2s per kill
- `multiplier` = 1 + floor(chain / 5), capped at 10x
- Chain breaks if no kills for 2.2s
- Kills every 10 enemies show combo pop

### Difficulty System (Lines 485–489)
Three presets control enemy spawn rate, speed, shooting frequency, lives, bombs, and boss HP scaling:
- `easy`: Slower spawning, slower enemies, fewer bombs
- `normal`: Default balance
- `hard`: Constant spawning, 1.9x enemy speed, 95% shoot chance, half bomb count

### Accessibility & UX (Lines 474–483, 529–540)
- **Reduced motion**: Disables animations, screen shake, flashing when enabled
- **Screen reader support**: `srLive` element announces game events via `aria-live`
- **Performance scaling**: `quality` variable (0=full, 1=medium, 2=low) adjusts particle counts and glow effects
- **Storage**: Game saves best score, mute state, difficulty, reduced-motion preference to localStorage

### Audio Synthesis (Lines 542–676)
All sounds are generated with the WebAudio API. Key functions:
- `beep()`: Oscillator-based tones with optional frequency sweep, filters, panning
- `noiseBurst()`: White/pink noise with envelope and filters
- Effects use envelope shaping (fast attack, exponential release) and panning for positional audio
- Compressor on master gain for dynamic control

## Development Workflow

### Testing the Game
```bash
# Open in browser (no build step needed)
open index.html
# Or use a local HTTP server
python3 -m http.server 8000
# Then visit http://localhost:8000/
```

The game auto-starts on page load. Use:
- **Mouse**: Hover to steer, click to fire
- **Keyboard**: Arrow keys or WASD to move, Space to fire, X/Shift to bomb, P/Esc to pause
- **Touch**: Drag left zone to move, tap FIRE/BOMB buttons

### Modifying Game Parameters

**Canvas Size** (Line 753): `BASE_W = 480, BASE_H = 720`

**Difficulty Tuning** (Lines 485–489): Adjust spawn timings, enemy speeds, boss HP multipliers

**Stage/Boss Content** (Lines 493–515):
- `THEMES[]`: Add/modify stage color schemes (sky gradient, cloud color, sun color, name)
- `BOSSES[]`: Add/modify boss designs (hull colors, core color, HP multiplier)
- `BOSS_PATTERNS[]`: Add/modify firing patterns (must match BOSSES array length)

**Enemy Types** (Lines 502–506): Adjust HP, speed multiplier, color, score value, spawn stage

**Weapon Balancing** (Lines 1124–1146):
- Vulcan: Spread angle at different levels
- Laser: Number of lanes, piercing depth, fire rate (cooldown in line 1177)

**Scoring & Multiplier** (Line 1090): Adjust kill threshold for multiplier increase

### Common Editing Tasks

**Add a new weapon type**:
1. Add to weapon check in `fireWeapon()` (line 1126)
2. Update `weaponLabel()` (line 871)
3. Adjust fire rate and spread in weapon logic

**Add a new boss attack pattern**:
1. Create a new firing function using helper functions like `fireRadial()`, `fireAimedSpread()`
2. Add a new array to `BOSS_PATTERNS[]` with 3+ attack functions
3. Ensure the index matches the `BOSSES[]` array index

**Change HUD layout**: Modify CSS in `<style>` block (lines 10–358) and HTML in `<div id="hud">` (lines 367–374)

**Add new sound effect**: Create function in `sfx` object (lines 624–676), call it in appropriate game events

## Important Conventions

**Frame-based vs. time-based animation**: Some systems use frame count (e.g., `frame * 0.1` for rotation), others use delta time (e.g., `speed * dt` for movement). Keep this distinction when modifying.

**Collision radius**: All objects use simple AABB collision. Radii are roughly half of width/height.

**Enemy bullet types**: 
- Standard bullets use `speed` and optional `vx` (horizontal component)
- Radial bullets (from bosses) use `dx/dy` components; these are NOT affected by `speed`

**Invulnerability period**: 1.5s after hit, with visual blink every 4 frames (`Math.floor(frame / 4) % 2`)

**Boss state machine**: Bosses have `entering` phase (moving to center), then `telegraph` phase (warning animation), then attack phase. Telegraph duration is 22 frames.

**Storage keys**: All localStorage keys are prefixed with `skyfighter.` to avoid conflicts

## Performance Notes

- Adaptive quality system reduces particles and disables glow at low FPS
- Screen shake is reduced if `reducedMotion` is enabled
- Canvas rendering uses `ctx.save()/restore()` for clean state management
- Cloud background layers move at different speeds for parallax effect
