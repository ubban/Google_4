# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. The entire game—engine, graphics, audio, UI, and gameplay logic—is contained in `index.html` with no external dependencies.

The game features:
- Vertical scrolling bullet-hell gameplay with procedural enemy waves
- Boss encounters with distinct attack patterns (6 unique bosses)
- Dual weapon system (Vulcan spread fire vs. Laser beams) with power-up progression
- Difficulty levels (Easy, Normal, Hard) affecting enemy spawn rates and behavior
- Full accessibility support: reduced motion preferences, screen reader announcements, keyboard-friendly controls
- Touch and mouse/keyboard input (adaptive for mobile and desktop)
- Procedurally generated audio (Web Audio API synthesis—no audio files)
- Adaptive quality rendering (dynamically adjusts particle counts and effects based on frame rate)
- Persistent state (best score, mute preference, difficulty, reduced motion setting via localStorage)

## Architecture & Key Concepts

### High-Level Structure

The codebase is organized as a self-contained IIFE (Immediately Invoked Function Expression) containing:

1. **UI Layer** (HTML/CSS)
   - Main canvas for game rendering
   - HUD overlays (score, lives, weapon level, bombs)
   - Start/pause/game-over screens
   - Touch-friendly button layout for mobile
   - Boss health bar display

2. **Input System**
   - Multi-modal: keyboard (arrows + WASD, Space to fire, X/Shift for bomb, P/Esc to pause)
   - Mouse: continuous aiming with crosshair reticle (player glides toward pointer)
   - Touch: dual-zone layout with virtual joystick and action buttons
   - All input modalities can coexist; keyboard/joystick take priority over mouse to prevent fighting

3. **Game Loop** (`loop()` function)
   - Requestanimationframe-driven at display refresh rate
   - Delta-time-based updates to handle variable frame rates
   - Performance tracking: adaptive quality system monitors frame times and reduces particle/effect density if framerate drops below 40 FPS

4. **Game State Management**
   - Player entity: position, weapon, cooldown, invulnerability frames
   - Enemies: spawn pools with type-specific stats (grunt, interceptor, tank)
   - Boss: single-entity with telegraph phase and attack pattern cycling
   - Scoring: chain combo multiplier system (resets if player is hit)
   - Difficulty config: `DIFFICULTIES` object drives spawn rates, enemy speed, shoot chance, lives/bombs/boss HP

5. **Rendering** (`draw()` function)
   - Procedural background: layered clouds with parallax scrolling and theme-based color gradients
   - 2D canvas drawing only (no WebGL, no image assets)
   - Screen shake effect applied via canvas translate
   - Dynamic glow/shadow effects toggled by quality setting

6. **Collision & Physics**
   - AABB (axis-aligned bounding box) overlap checks: `rectsOverlap()`
   - Player movement: clamped to canvas bounds
   - Bullets: simple linear motion; enemy bullets support radial (dx/dy) or aimed velocity
   - Particle system: spawn explosion particles on kills/hits, decay over lifetime

7. **Audio** (`sfx` object)
   - All sounds synthesized via Web Audio API (no external files)
   - Beep/noise generation with envelope shaping, frequency sweeping, stereo panning
   - Spatial audio: sounds pan left/right based on on-screen X position
   - Mutable via UI button with localStorage persistence

### Data Structures

**Player Object:**
```javascript
player = {
  x, y,           // screen position
  w, h,           // hitbox dimensions
  speed,          // pixels/second movement speed
  cooldown,       // weapon fire delay timer
  weapon,         // 'vulcan' or 'laser'
  level,          // 1-3 progression level
  invuln          // invulnerability frames after hit
}
```

**Enemy Object:**
```javascript
// Stored in enemies[] array
{
  x, y, w, h,     // position and hitbox
  type,           // 'grunt', 'interceptor', 'tank'
  hp,             // hit points (bullets reduce this)
  speed,          // movement speed
  shootTimer,     // delay before next shot
  canShoot,       // whether this enemy fires at player
  pattern,        // 'spread' or 'aimed' bullet pattern
  weave,          // whether this type weaves left/right
  baseX, weaveSeed // for weaving motion
}
```

**Boss Object:**
```javascript
{
  x, y, w, h,      // position and hitbox
  hp, maxHp,       // health tracking
  design,          // reference to BOSSES[n] (name, colors, hpMul)
  entering,        // boolean: in entry animation
  telegraph,       // countdown timer: >0 means telegraphing next attack
  attackTimer,     // countdown until next attack
  patternIndex,    // which pattern in BOSS_PATTERNS to use
  vx               // horizontal velocity (-1 or +1)
}
```

**Bullet Objects:**
```javascript
// Player bullets: in bullets[] array
{ x, y, w, h, speed, vx, pierce, laser }

// Enemy bullets: in enemyBullets[] array  
{ x, y, w, h, speed, vx } // or radial: { x, y, w, h, dx, dy, radial: true }
```

### Critical Game Flow

1. **Start Screen** → player selects difficulty → clicks "Start Game"
2. **Stage Begin** → display stage banner; spawn initial enemy wave
3. **Enemy Spawning** → `spawnTimer` decrements; when ≤0, spawn random enemy; difficulty ramps spawn rate over time
4. **Stage Progression** → after `KILLS_PER_STAGE` (18 kills) → boss warning banner → boss spawn
5. **Boss Phase** → boss enters with telegraph/attack cycle; unique patterns per boss
6. **Stage Clear** → boss defeated → score bonus → next stage
7. **Game Over** → lives exhausted → show final score and best score; allow restart

### Boss Attack Patterns

Each boss (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) has 3 attack patterns defined in `BOSS_PATTERNS`:

- **Radial Burst**: bullets spread outward in a circle
- **Aimed Spread**: bullets fan out around player direction
- **Column Fire**: vertical lines of bullets
- **Spiral Burst**: 8-way rotating pattern
- **Cross Fire**: 8-direction cardinal + diagonal
- **Homing Seeded**: bullets aimed with spread variance
- **Double Ring**: two concentric circles

Bosses telegraph 22 frames before firing (visual core flash, audio cue). Player can evade during this window.

### Scoring & Multiplier System

- Base kill scores: grunt (10), interceptor (15), tank (35), boss (500)
- **Chain Multiplier**: each kill increments `chain` counter; multiplier = 1 + floor(chain/5) (cap 10x)
- **Chain Timeout**: `chainTimer` = 2.2 seconds; if no kill within timeout, chain resets to 1x
- Use case: encourages consecutive kills without gaps for higher scores

### Difficulty Scaling

Three presets in `DIFFICULTIES` object:
- **Easy**: spawnBase 75, enemySpeed 1.0, 4 lives, 3 bombs, boss HP 0.8x
- **Normal**: spawnBase 58, enemySpeed 1.4, 3 lives, 2 bombs, boss HP 1.0x
- **Hard**: spawnBase 40, enemySpeed 1.9, 3 lives, 2 bombs, boss HP 1.3x

Difficulty persists in localStorage; dynamically ramps spawn rate further: `spawnTimer = max(spawnMin, spawnBase - floor(frame/300)*4)`

### Visual Themes

Six procedurally generated environment themes (sky gradient, cloud color, sun color) cycle through stages:
- Daylight Front
- Sunset Corridor
- Night Approach
- Storm Front
- Arctic Wastes
- Volcanic Ridge

Themes are purely cosmetic; no gameplay impact.

## Development Workflow

### Running the Game

Simply open `index.html` in a web browser. No build step, no server required.

```bash
# Option 1: Direct file open
open index.html

# Option 2: Local server (for testing cross-origin features if needed)
python -m http.server 8000
# Then visit http://localhost:8000/index.html
```

### Making Changes

1. **Edit `index.html`** in your editor
2. **Reload browser** (Cmd+R / Ctrl+R) to see changes immediately
3. **Use browser DevTools** (F12) to debug:
   - Canvas is 480x720 (internal) but scales to fit window
   - Console logs available; add `console.log()` statements as needed
   - Inspect game state by adding breakpoints in the `update()` or `draw()` functions

### Key Areas for Common Edits

- **Spawn rates & difficulty**: Edit `DIFFICULTIES` object (lines 485–489)
- **Enemy types**: Edit `ENEMY_TYPES` object (lines 502–506)
- **Boss definitions**: Edit `BOSSES` array (lines 508–515) and `BOSS_PATTERNS` array (lines 1372–1409)
- **Audio SFX**: Edit `sfx` object (lines 624–676); function signatures are `beep(freq, dur, type, vol, opts)` and `noiseBurst(dur, vol, opts)`
- **UI text / controls hint**: Edit the HTML in the `#overlayCard` section (lines 401–419)
- **Weapon behavior**: Edit `fireWeapon()` function (lines 1124–1146)
- **Collision response**: Search for `rectsOverlap()` calls; modify enemy/bullet/powerup collision handlers (lines 1232–1294)
- **Visual style**: Edit CSS in `<style>` block (lines 10–358) for colors, animations, layout
- **Themes**: Edit `THEMES` array (lines 493–500) for sky gradients and cloud colors
- **Scoring**: Modify `addScore()` logic (lines 1087–1101) and base score values in enemy type definitions

### Testing & Verification

- **Difficulty Balance**: Play through all three difficulties; adjust `DIFFICULTIES` spawn rates and boss HP multipliers as needed
- **Touch Controls**: Test on mobile device or use browser device emulation (DevTools → Device Toolbar)
- **Performance**: Monitor frame rate using browser DevTools Performance tab; if dropping below 40 FPS, the quality adapter will kick in (reduce particles)
- **Accessibility**: Test keyboard-only play; verify screen reader announcements; toggle reduced motion and confirm effects dim/disable
- **Collision**: Visually verify enemy/bullet hitboxes overlap detection; adjust dimensions in enemy type definitions if needed

### Browser Compatibility

Target modern browsers (Chrome, Firefox, Safari, Edge) with Canvas API, Web Audio API, and ES6 support. Touch support detected with `('ontouchstart' in window) || navigator.maxTouchPoints > 0`.

## Important Implementation Details

### Weapon System

- **Vulcan** (default, rapid fire):
  - Level 1: single center bullet
  - Level 2: two bullets (left/right spread)
  - Level 3: three bullets (left/center/right)
  - Fire rate: 0.15s between shots

- **Laser** (higher damage, tighter fire):
  - Level 1: single center laser
  - Level 2: two lasers (left/right)
  - Level 3: three lasers (left/center/right)
  - Fire rate: 0.1s between shots
  - Pierces enemies: level 1 (1 pierce), level 2+ (1 pierce), level 3+ (2 pierce)

Weapon power-ups cycle: Vulcan → up levels 1→2→3 → Laser level 1 → back to Vulcan.

### Input Priority & Mouse Behavior

The player's movement input has a priority order:
1. Keyboard (arrows/WASD) and joystick input take full effect
2. If no keyboard/joystick input, mouse position becomes a target and player glides toward it
3. Mouse cursor is not shown when keyboard/joystick active

This prevents mouse and keyboard from "fighting" each other.

### Reduced Motion Support

When reduced motion is enabled:
- Screen shake magnitude reduced to 15% and clamped to 0.1s max duration
- Animations disabled (CSS `animation: none`)
- Glow/shadow effects disabled in render
- User preference for `prefers-reduced-motion` is respected on first load

### Local Storage Keys

- `skyfighter.bestScore` — best score (number)
- `skyfighter.muted` — audio muted (0 or 1)
- `skyfighter.difficulty` — selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion` — reduced motion enabled (0 or 1)

### Performance Adaptive Quality System

Frame times are tracked in a 45-frame rolling window. If average FPS drops below ~40, quality level increments:
- Quality 0: Full effects (glow, shadow, max particle count)
- Quality 1: Medium (60% particles, reduced shadow/glow)
- Quality 2: Low (35% particles, no shadow/glow)

Quality automatically downgrades as framerate drops and upgrades as it recovers.

### Boss Telegraph Mechanic

Bosses have a two-phase attack cycle:
1. **Telegraph Phase** (22 frames): boss core flashes white with glow, audio cue plays
2. **Attack Phase** (wait attackTimer frames): boss fires pattern, resets to telegraph

This gives players a reaction window to evade incoming fire.

## Notes for Future Development

- The single-file structure is intentional for instant deployment and no build complexity
- Adding new enemy types: add to `ENEMY_TYPES`, update `spawnEnemy()` filter logic
- Adding new boss: add to `BOSSES`, add pattern array to `BOSS_PATTERNS`, update boss selection in `spawnBoss()`
- Adding new stages: themes auto-cycle; just increment stage counter
- Persistent modding: changes are confined to JavaScript variables and Canvas 2D API; no server/database needed
- Audio is fully procedural; no sound asset pipeline to manage
