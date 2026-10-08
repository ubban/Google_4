# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up arcade game built as a single HTML5 file with no external dependencies. The entire game engine, audio synthesis, rendering, and UI are self-contained within `index.html`.

## How to Run

- **Web Browser**: Open `index.html` directly in any modern browser (no server required)
- **Development**: Use any text editor to modify index.html and refresh the browser to see changes
- **No Build System**: This project requires no build tools, npm packages, or compilation

## Architecture Overview

### Single-File Design
The entire game is contained in `index.html` (~1670 lines):
- **Lines 1-359**: HTML structure and CSS styling
- **Lines 422-1667**: JavaScript game engine (wrapped in IIFE)

### Key Game Systems

**State Management** (lines 863-870)
- Player object: position, speed, weapon, level, invulnerability state
- Bullets array: player projectiles with piercing mechanics
- Enemies array: various enemy types with AI
- Enemy bullets array: incoming projectiles from enemies
- Particles: explosion effects
- Power-ups: drops from defeated enemies
- Boss: single boss entity with health and attack patterns

**Rendering Pipeline** (lines 1520-1638)
- Canvas setup: fixed 480x720 base resolution, scaled to fit viewport
- Three-layer drawing: sky gradient + sun glow, game objects, effects
- Screen shake effect for impacts
- Adaptive rendering: quality levels (0=full, 1=medium, 2=low) based on frame time

**Input Handling** (lines 768-862)
- Keyboard: Arrow keys + WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
- Mouse: Click to fire, move to steer (glides toward cursor)
- Touch: Left zone for joystick movement, right zone with FIRE/BOMB buttons
- Priority: Keyboard/joystick input overrides mouse to prevent conflicts

**Game Loop** (lines 1640-1653)
- Runs at ~60 FPS via requestAnimationFrame
- Updates game state if running and not paused
- Applies screen shake and draws each frame
- Tracks performance to adaptively reduce rendering quality

### Enemy System (lines 502-506, 1022-1041, 1210-1228)

**Three Enemy Types:**
- **Grunt** (w:34 h:34, 1 HP): Basic red enemy, score 10
- **Interceptor** (w:26 h:26, 1 HP): Fast orange enemy with weaving, score 15, appears stage 1+
- **Tank** (w:46 h:46, 4 HP): Slow purple heavy, score 35, appears stage 2+

**Enemy AI:**
- Spawn rate decreases over time (difficulty scaling)
- Two firing patterns: "spread" (5-bullet spray) and "aimed" (homing toward player)
- Weaving enemies use sine wave motion
- Max 18 kills per stage triggers boss spawn

### Boss System (lines 508-515, 1043-1056, 1372-1439)

**Six Unique Bosses** (one per stage, then loop):
1. **VANGUARD**: Radial burst, aimed fan, wide V-fan
2. **BEHEMOTH**: Heavy columns, slow homing, dense radial walls
3. **SPECTER**: Teleport dash, tight spiral, double aimed shots
4. **DREADNOUGHT**: 8-way cross fire, homing missiles, dense walls
5. **WRAITH**: Teleport barrage, expanding double ring, cross-spread
6. **TITAN**: Heavy columns, full ring, relentless homing

Each boss has:
- Telegraph phase (22 frames of warning) before firing
- Attack timer determines pattern rotation
- Horizontal movement with boundary bouncing
- Health scaling by difficulty (×0.8 easy, ×1.0 normal, ×1.3 hard)

### Weapon System (lines 871-901, 1124-1146)

**Vulcan Gun:**
- Cooldown: 0.15s between shots
- Level 1: Single narrow bullet
- Level 2: 2-bullet spread
- Level 3: 3-bullet spread

**Laser:**
- Cooldown: 0.1s between shots
- Pierce mechanic: can damage multiple enemies (pierce value decreases)
- Level 1: Single lane
- Level 2: 2 lanes
- Level 3: 3 lanes (center + 2 sides)

**Power-up System** (lines 1058-1067, 1304-1322):
- 6% chance: Extra life (up to max)
- 8% chance: Extra bomb (up to max)
- 20% chance: Weapon upgrade (level up or switch to laser)

### Difficulty System (lines 485-489, 741-750)

Three preset difficulty levels stored in localStorage:
- **Easy**: Slower spawns (75→40), slower enemies (1.0×), 45% shoot chance, 4 lives, 3 bombs, 0.8× boss HP
- **Normal**: Medium spawns (58→26), medium enemies (1.4×), 70% shoot chance, 3 lives, 2 bombs, 1.0× boss HP
- **Hard**: Fast spawns (40→18), fast enemies (1.9×), 95% shoot chance, 3 lives, 2 bombs, 1.3× boss HP

### Visual Themes (lines 493-500)

Six parallax-scrolling sky backgrounds, each with unique sky gradient, cloud color, and sun tint. Themes rotate per stage: Daylight Front → Sunset Corridor → Night Approach → Storm Front → Arctic Wastes → Volcanic Ridge → (repeat).

### Audio System (lines 543-676)

**Web Audio API Synthesizer** (no external audio files):
- Master gain (0.9) → compressor → destination
- `beep(freq, dur, type, vol, opts)`: Tone synthesis with optional frequency sweep, filter, panning, delay
- `noiseBurst(dur, vol, opts)`: White noise generation with filter
- Spatial audio: Pan value calculated from screen position using `panFor(x)`
- Mute toggle persists in localStorage

**Sound Effects:**
- `shoot`: 760Hz square wave swept to 480Hz
- `laser`: 1500Hz sawtooth swept to 2400Hz
- `explodeGrunt/Tank/Interceptor`: Unique multi-voice explosions
- `hit`: Triple hit with sawtooth + noise
- `bomb`: Heavy explosion sound
- `powerup`: Ascending arpeggio
- `bossHit`: Sharp single beep
- `bossTelegraph`: Rising sine sweep
- `bossDown`: Descending 5-note sequence
- `stageClear`: Ascending arpeggio
- `gameover`: Descending sawtooth sequence

### Scoring & Combo System (lines 1087-1101, 1088-1094)

- Kill chain extends timer (2.2s) when you get a kill
- Multiplier increases every 5 kills (max ×10 at 50+ chain)
- Breaking chain (2.2s without kills) resets multiplier to 1×
- Score milestones trigger "X KILLS!" pop-ups at multiples of 10

### UI Elements & HUD (lines 34-359, 367-419)

**Top HUD:** Score, Best Score, Multiplier, Weapon name/level, Hearts (lives), Bombs
**Boss Bar:** Visible only when boss present, shows health as percentage
**Overlays:** Start menu, pause screen, game over screen
**Buttons:** Mute, motion toggle, pause
**Touch Controls:** Joystick on left, FIRE/BOMB buttons on right (mobile only)

### Accessibility Features

- `prefers-reduced-motion` media query support: Disables all animations if OS reduces motion
- Screen reader announcements via `srLive` element
- Aria-live region for important status updates
- Safe-area insets for notched devices

### Performance Optimization (lines 473-483)

Adaptive quality system tracks frame times:
- Quality 0 (full): Glow effects, shadows, high particle count
- Quality 1 (medium): Reduced shadow blur, 60% particle count
- Quality 2 (low): No shadows, 35% particle count
- Adjusts automatically when frame time averages >1/40s (lower quality) or <1/56s (higher quality)

### Data Persistence

Uses localStorage for:
- Best score: `skyfighter.bestScore`
- Mute state: `skyfighter.muted`
- Difficulty selection: `skyfighter.difficulty`
- Reduced motion preference: `skyfighter.reducedMotion`

## Common Development Tasks

### Modify Enemy Behavior
Find `ENEMY_TYPES` (line 502) to adjust stats, `spawnEnemy()` (line 1022) to change spawn logic or patterns, `enemies.forEach` (line 1210) to modify per-frame behavior.

### Tweak Boss Attacks
Edit `BOSS_PATTERNS` array (line 1372) to change attack sequences. Each boss has its own pattern array with functions like `fireRadial()`, `fireAimedSpread()`, etc. (lines 1324-1370).

### Add/Change Visual Theme
Add new entry to `THEMES` array (line 493) with sky gradient colors, cloud/sun colors, and theme name. Game automatically cycles through stages.

### Adjust Difficulty
Modify `DIFFICULTIES` object (line 485) spawn rates, enemy speeds, shoot chances, and HP multipliers.

### Change Weapon Mechanics
Edit `fireWeapon()` (line 1124) for bullet generation, `VULCAN` vs `LASER` spread patterns in weapon levels, cooldowns, or piercing behavior.

### Modify Audio
Adjust existing calls to `beep()` or `noiseBurst()` in the `sfx` object (line 624), or create new audio functions and call them from gameplay events.

### Add Particle Effects
Use `spawnExplosion(x, y, color, count)` (line 1073) anywhere in the code, or create custom particle objects in the `particles` array.

### Optimize Further
- Increase quality levels: Modify `quality` threshold values (line 480-481)
- Reduce particle density: Adjust multiplier in line 1074
- Disable glow effects: Set `const glow = false;` at line 1579

## Key Variables & Constants

- `BASE_W=480, BASE_H=720`: Canvas dimensions
- `KILLS_PER_STAGE=18`: Enemies needed to trigger boss
- `frame`: Increments every update, used for timing and animations
- `quality`: 0=full, 1=medium, 2=low (auto-adjusting)
- `chain`: Current kill streak counter
- `multiplier`: Score multiplier based on chain
- `stage`: Current stage number (auto-incrementing)

## Collision Detection

Simple AABB (axis-aligned bounding box) via `rectsOverlap()` (line 1069):
```javascript
Math.abs(a.x - b.x) < (a.w + b.w) / 2 && Math.abs(a.y - b.y) < (a.h + b.h) / 2
```
Used for: player-enemy, player-bullets, enemy-bullets, powerups, and boss collision.

## Testing in Browser

1. Open index.html in any modern browser
2. Select difficulty (Easy/Normal/Hard)
3. Click "Start Game"
4. Use keyboard (arrows + space), mouse (move + click), or touch controls
5. Press P or Esc to pause/resume
6. Use top-right buttons for mute and motion toggle
7. Open browser DevTools (F12) to check console for any errors

## Browser Compatibility

- Requires ES6 support (let, const, arrow functions, template literals)
- Uses Web Audio API (all modern browsers support this)
- Canvas 2D context
- LocalStorage
- requestAnimationFrame
- Tested on Chrome, Firefox, Safari, Edge (desktop and mobile)
