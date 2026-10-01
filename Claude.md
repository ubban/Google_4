# Sky Fighter — Codebase Guide

## Project Overview

Sky Fighter is a **Raiden-style vertical shoot-'em-up game** built entirely in a single HTML5 file with embedded CSS and JavaScript. It runs in any modern web browser with no external dependencies.

### Key Features
- **Vertical scrolling gameplay**: Player flies up the screen, enemies descend
- **Multiple weapons**: Vulcan (spread shots) and Laser (piercing beams)
- **Progressive difficulty**: 6 themed stages with unique boss fights
- **Power-ups**: Life recovery, bombs, and weapon upgrades
- **Responsive design**: Scales to mobile/tablet with touch controls
- **Web Audio synthesis**: All sound effects generated procedurally (no audio files)
- **Adaptive performance**: Quality scaling based on frame time

---

## Architecture & File Structure

The entire game is contained in **`index.html`** (1670 lines):
- **Lines 1–359**: HTML structure + CSS styling (layout, animations, theme variables)
- **Lines 360–420**: DOM elements (canvas, HUD, overlay menu, touch controls)
- **Lines 422–1667**: JavaScript game loop and logic (all game systems)

### No build step required
- Single self-contained file
- Can be served as-is or run locally in a browser
- localStorage for persistence (best scores, settings)

---

## Game Systems

### 1. **Player (`player` object)**
- **Position**: `x`, `y` on canvas
- **Weapons**: 'vulcan' or 'laser' with levels 1–3
  - **Vulcan** (default): Rapid-fire spread shots
    - Level 1: Single center shot
    - Level 2: Two shots (left/right)
    - Level 3: Three shots (wide spread)
  - **Laser**: Piercing beams that can hit multiple enemies
    - Level 1: Two lanes (left/right of ship)
    - Level 2: Three lanes (added center)
    - Level 3: Increased pierce multiplier
- **Controls**:
  - Keyboard: Arrow keys or WASD to move, Space to fire, X/Shift to bomb
  - Mouse: Move to steer, click to fire
  - Touch: Left side drag to move, FIRE/BOMB buttons (bottom-right)
- **Stats**: HP (lives), invulnerability window post-hit, cooldown between shots

### 2. **Enemies (`enemies` array)**
- **Three types**:
  - **Grunt** (red): 1 HP, basic drop speed, score 10
  - **Interceptor** (orange): 1 HP, 1.7× speed, weaving flight, score 15, stage 1+
  - **Tank** (purple): 4 HP, slow, high score (35), stage 2+
- **Behavior**:
  - Spawn at random x-position, enter from top
  - Shoot toward player (aimed) or in spread pattern
  - Weaving interceptors follow sinusoidal path
  - HP bar drawn above tanks
- **Death**: Explosion particles + power-up drop chance

### 3. **Bosses (`boss` object)**
- **Six boss designs** (cycle per stage, difficulty varies):
  1. **VANGUARD**: Radial bursts, aimed fans, V-spreads
  2. **BEHEMOTH**: Heavy columns, slow homing, dense radial
  3. **SPECTER**: Teleport + spiral, tight aimed shots
  4. **DREADNOUGHT**: 8-way cross, homing volleys
  5. **WRAITH**: Teleport barrage, expanding rings
  6. **TITAN**: Columns, full radial ring, relentless homing

- **Mechanics**:
  - Spawn after 18 kills per stage
  - Enter from top (500px travel)
  - Patrol side-to-side once settled
  - Telegraph wind-up (22 frames) before each attack pattern
  - HP scales with difficulty × boss multiplier
  - Bullets marked `radial: true` (different velocity model)

### 4. **Weapons System**
- **Vulcan** (fire-and-forget):
  - Creates bullet objects with `vy: -500`, optional `vx` for spread
  - Spread angle varies by level
- **Laser** (pierce mechanic):
  - Marked with `laser: true` and `pierce` counter
  - Bullets continue until pierce count reaches 0
  - Bypasses some collision logic

### 5. **Power-Ups (`powerUps` array)**
Spawn on enemy death with probability:
- **Life** (6%): Restore 1 HP (capped at max)
- **Bomb** (8%): +1 bomb (capped at max)
- **Weapon** (20%): Level up weapon or swap (Vulcan ↔ Laser)

### 6. **Scoring & Chain Multiplier**
- **Score formula**: `baseScore × multiplier`
- **Multiplier**: 1 + floor(chain / 5), max 10
- **Chain**: Increments on each kill, timer resets (2.2s grace period)
- Break chain on player hit or inactivity

### 7. **Difficulty Modes**
Three presets in `DIFFICULTIES` object (lines 485–489):
- **Easy**: Slower spawns, weak enemies, 4 lives, 3 bombs, 0.8× boss HP
- **Normal** (default): Balanced
- **Hard**: Rapid spawns, aggressive enemies, 1.3× boss HP

---

## Key Constants & Data

### Screen & Physics
```javascript
const BASE_W = 480, BASE_H = 720;  // Canvas dimensions
const KILLS_PER_STAGE = 18;         // Enemies before boss
const STORAGE_BEST = 'skyfighter.bestScore';  // localStorage key
```

### Enemy Types (`ENEMY_TYPES`)
- `w`, `h`: Pixel size
- `hp`: Hit points
- `speedMul`: Speed multiplier vs difficulty setting
- `score`: Points on kill
- `minStage`: First stage they appear

### Themes (`THEMES` array)
- 6 stages with unique sky gradients, cloud colors, sun colors
- Cycles: Arctic Wastes → Volcanic Ridge → repeat

### Bosses (`BOSSES` array)
- `name`: Displayed during fight
- `hullTop`, `hullBottom`, `wing`, `core`: Colors
- `hpMul`: HP multiplier (0.8 to 1.7)

### Boss Attack Patterns (`BOSS_PATTERNS`)
- Array of 6 pattern lists (one per boss type)
- Each list contains 3 functions: `b => { /* fire bullets */ }`
- Functions called in sequence during fight, cycling

---

## Audio (Web Audio API)

All sounds synthesized at runtime (no external assets):
- **Oscillators**: Square, sawtooth, sine, triangle waves
- **Filters**: Low/high-pass for tonal shaping
- **Noise bursts**: Procedural white noise with envelopes
- **Spatial**: Stereo panning based on bullet x-position

Sound effects tied to events:
- `sfx.shoot()`, `sfx.laser()` — player fire
- `sfx.explodeGrunt()`, `sfx.explodeTank()` — enemy death
- `sfx.bomb()` — smart bomb used
- `sfx.bossDown()` — boss defeated
- `sfx.gameover()`, `sfx.stageClear()` — game transitions

All sounds respect mute toggle and master gain.

---

## Input Handling

### Keyboard
- Arrow keys / WASD: Move
- Space: Fire
- X / Shift: Bomb
- P / Esc: Pause

### Mouse
- Move over canvas: Steer plane toward cursor
- Click: Fire (continuous while pressed)
- Leave canvas: Disable mouse steering

### Touch (Mobile)
- Left zone (55% width): Joystick input
  - Visual feedback: base circle + stick position
  - Clamped to 50px radius
- Right zone (45% width): Fire/Bomb buttons
- Both zones support multi-touch

---

## Rendering & Animation

### Screen Shake
- Triggered by hits/explosions
- Applied as random translation offset
- Disabled in reduced-motion mode

### Particle System
- Created on explosions
- Velocity-based movement
- Fade-out with shrinking circles
- Quality scaled (full, medium, low particle counts)

### Clouds
- Procedural parallax scrolling
- Two layers (speeds 0.5×, 1.2×)
- Repositioned each frame off-screen for infinite scroll

### Vignette (Red Border)
- Shows when player has 1 HP
- Pulsing animation (disabled in reduced-motion mode)

---

## Game Loop & Frame Timing

```javascript
function loop(now) {
  dt = (now - lastTime) / 1000;        // Frame delta (capped at 50ms)
  trackPerf(dt);                       // Adaptive quality check
  if (running && !paused) {
    update(dt);                        // Game logic
    draw();                            // Render
  }
  requestAnimationFrame(loop);
}
```

### Performance Tracking
- Samples last 45 frame times
- Adjusts quality (0=full, 1=medium, 2=low) if average FPS drops/recovers
- Affects particle count and glow effects

---

## Accessibility Features

- **Screen Reader Support**:
  - `#srLive` (aria-live) announces score, stage transitions, game-over status
  - `announce(text)` function posts messages with 30ms delay for TTS

- **Reduced Motion**:
  - Checkbox in pause menu (`motionBtn`)
  - Disables: screen shake, vignette pulse, flash effects, animations
  - Defaults to system `prefers-reduced-motion` preference
  - Stored in localStorage

---

## Save & Persist

Uses browser **localStorage** with keys:
- `skyfighter.bestScore` — High score
- `skyfighter.muted` — Audio mute state ('0'/'1')
- `skyfighter.difficulty` — Selected difficulty
- `skyfighter.reducedMotion` — Motion preference ('0'/'1')

---

## Development Notes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (lines 502–506)
2. Define spawn logic in `spawnEnemy()` if conditional
3. Create explosion SFX in `EXPLODE_SFX` map if different sound needed
4. Optional: Create drawEnemy function or use generic `drawPlane()`

### Adding a New Boss
1. Add to `BOSSES` array (lines 508–515) with colors and hpMul
2. Create attack pattern array in `BOSS_PATTERNS` (one per boss type)
3. Patterns are called sequentially during fight based on `patternIndex`

### Tweaking Difficulty
Edit `DIFFICULTIES` object (lines 485–489):
- `spawnBase`/`spawnMin`: Frame delays between enemy spawns
- `enemySpeed`, `enemyShootChance`: AI aggression
- `lives`, `bombs`: Starting counts
- `bossHpMul`: Boss health scale

### Canvas Scaling
- Base size: 480×720
- Scales up to 1.6× native size max via CSS
- Aspect ratio locked (no stretching)
- Touch controls adjust zones dynamically

---

## Testing Checklist

- [ ] All three difficulty modes play correctly
- [ ] Boss patterns execute (telegraph, bullets spawn)
- [ ] Weapon upgrades and swaps work
- [ ] Power-ups pick up correctly
- [ ] Chain multiplier builds and resets
- [ ] High score persists across page reloads
- [ ] Mute, motion, and difficulty settings persist
- [ ] Mobile touch controls responsive
- [ ] Screen reader announces score/stage/game-over
- [ ] Reduced motion disables all animations
- [ ] Audio context suspends/resumes correctly

---

## Performance Considerations

1. **Canvas rendering**: Clear + redraw every frame (no dirty rects)
2. **Collision**: Simple AABB (axis-aligned bounding box) checks
3. **Particles**: Culled when life ≤ 0; max count adaptive
4. **Boss bullets**: Radial bullets recalc position per frame (no pre-computed paths)
5. **Audio**: Oscillators + filters created per sound (low overhead due to short durations)

No WebGL, no heavy libraries—pure 2D Canvas API.

---

## Common Patterns

### Spawning objects
```javascript
enemies.push({ x, y, w, h, speed, hp, ... });
particles.push({ x, y, vx, vy, life, maxLife, color });
```

### Collision detection
```javascript
function rectsOverlap(a, b) {
  return Math.abs(a.x - b.x) < (a.w + b.w) / 2 &&
         Math.abs(a.y - b.y) < (a.h + b.h) / 2;
}
```

### Scoring
```javascript
addScore(baseAmount);  // Applies multiplier, updates display, extends chain
```

### Effects
```javascript
screenShake(magnitude, duration);   // Visual wobble
spawnExplosion(x, y, color, count); // Particles + optional SFX
```

---

## Browser Compatibility

- **Target**: Modern browsers (Chrome, Firefox, Safari, Edge)
- **Requirements**: ES6 syntax, Canvas 2D, Web Audio, localStorage
- **No transpilation** (uses arrow functions, const/let, template literals)
- **Touch events** feature-detected, controls hidden on non-touch devices

---

Good luck with Sky Fighter!
