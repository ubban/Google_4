# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 web application. The game features:
- Procedurally spawned enemies with distinct types and AI patterns
- 6 unique boss encounters with complex attack patterns
- Weapon progression system (Vulcan and Laser cannons)
- Difficulty levels (Easy, Normal, Hard) with distinct parameter tuning
- Touch and keyboard/mouse input support
- WebAudio API synthesized sound effects (no external audio files)
- Particle effects and screen shake for visual feedback
- Multi-stage gameplay with increasing difficulty
- Score multiplier chain system for skilled play

## File Structure

```
index.html          # Single-file application containing:
                    # - HTML structure (canvas + UI overlays)
                    # - CSS styling (responsive, dark theme)
                    # - Vanilla JavaScript game logic (~1200 lines)
README.md           # Project metadata
CLAUDE.md           # This file
```

## Architecture & Key Systems

### Game Loop (lines 1640–1665)
- Uses `requestAnimationFrame` for 60 FPS rendering
- Delta-time based updates for frame-rate independence
- Three main phases per frame: update logic, screen shake, rendering
- Performance tracking: adaptive quality system downscales particle effects on slower devices

### Game State Management (lines 864–869)
Central mutable state arrays:
- `player`: Position, velocity, weapon, cooldown, invulnerability
- `bullets`: Player projectiles (supports piercing laser rounds)
- `enemies`: Spawned enemies with AI and shooting behavior
- `enemyBullets`: Enemy projectiles
- `boss`: Active boss encounter state
- `particles`: Explosion and visual effects
- `powerUps`: Collectible items

### Input System (lines 767–862)
- **Keyboard**: WASD/Arrows for movement, Space/X/Shift for firing/bombs
- **Mouse**: Position drives player movement toward cursor when canvas is hovered; click to fire
- **Touch**: Left side drag for movement, right-side buttons for fire/bomb
- Joystick visualization for touch input at `#moveZone`
- `mouseActive` flag prevents keyboard/joystick interference when mouse is used

### Collision Detection (line 1069)
- `rectsOverlap(a, b)`: Simple AABB collision using distance formula on centers
- Used for: bullet-enemy hits, enemy-player collisions, power-up collection

### Rendering Pipeline (lines 1520–1637)
1. **Screen shake**: Random translation if `shakeTime > 0`
2. **Background**: Gradient sky with animated sun and scrolling clouds
3. **Game objects**: Player, enemies, bullets, boss, particles
4. **HUD**: Overlaid via HTML (not canvas) for performance
   - Score, best, multiplier, weapon status, hearts, bombs displayed via `#hud`
   - Boss health bar at `#bossBarWrap`

### Audio System (lines 542–676)
- Lazy initialization: WebAudio context created on first user interaction
- No external assets; all sounds synthesized via oscillators and noise generation
- `beep(freq, dur, type, vol, opts)`: Generates tones with sweeps and filters
- `noiseBurst(dur, vol, opts)`: White noise with envelope and filtering
- `sfx` object: Pre-defined sound profiles for game events (shoot, explode, powerup, etc.)
- Panning based on screen position: `panFor(x)` maps x-coordinate to stereo pan value
- Master gain with compressor for dynamic range control

### Difficulty System (lines 485–489)
Three presets control spawn rates, enemy speed, shooting chance, lives, bombs, and boss HP:
- **Easy**: Slower enemies, fewer shots, 4 lives, 3 bombs, 0.8× boss HP
- **Normal**: Balanced, 3 lives, 2 bombs, 1.0× boss HP
- **Hard**: Fast enemies, dense fire, 3 lives, 2 bombs, 1.3× boss HP

Difficulty is persisted in localStorage and can be changed before each game.

### Weapon System (lines 871–873, 1124–1145)
- **Vulcan** (default): Straight shots, spread pattern increases at levels 2–3
  - Level 1: Single center bullet
  - Level 2: Two angled bullets
  - Level 3: Three bullets (wide spread)
- **Laser**: Faster, piercing rounds that reduce `pierce` count instead of destroying
  - Level 1–2: Center lane(s)
  - Level 3: Three lanes (center + two offset)
- Weapon switching: Collect weapon power-up at max level to toggle between Vulcan ↔ Laser

### Scoring & Chain Multiplier (lines 1087–1101)
- Base score: Enemy kills grant `type.score` points × `multiplier`
- Chain: Each kill extends timer (`chainTimer = 2.2s`) and increments `chain` counter
- Multiplier: 1 + floor(chain / 5), capped at 10×
- Breaking chain: No enemy killed within window or taking damage resets to 1×

### Enemy Spawning (lines 1022–1040)
- Spawn timer decrements; when it reaches 0, spawn one enemy and reset timer
- Difficulty ramps: `spawnTimer = max(spawnMin, spawnBase - difficultyRamp)`
- Enemy types unlock at stages 2+ (tank, interceptor) via `minStage` check
- Weaving enemies (interceptors) use sine wave pattern relative to spawn point

### Boss System (lines 1043–1056, 1411–1439)
- Spawns after player kills 18 enemies (`KILLS_PER_STAGE = 18`)
- Boss entrance animation: Y position eases from off-screen to Y=120
- **Boss patterns** (lines 1372–1408): Each of 6 bosses has 3 attack patterns:
  - VANGUARD: Radial bursts, aimed spreads, V-formation fans
  - BEHEMOTH: Heavy columns, homing spreads, dense radial walls
  - SPECTER: Teleport dash, spiral bursts, double aimed shots
  - DREADNOUGHT: 8-way cross, homing missiles, dense walls
  - WRAITH: Teleport barrages, expanding rings, aimed cross-spreads
  - TITAN: Heavy columns, full rings, relentless homing volleys
- Boss health bar fills at `#bossBarFill` by fraction of remaining HP
- Telegraph phase (22 frames) before attack fires: Color flashes white
- Boss movement: Bounces horizontally between screen edges

### Theme System (lines 493–500)
- 6 rotating themes per stage: sky gradients, cloud colors, sun glow
- Themes cycle and repeat every 6 stages

## Common Development Tasks

### Run the Game
1. Open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge)
2. Game runs immediately with no build step or dependencies
3. Responsive layout: scales canvas to fit window while maintaining 480×720 aspect ratio

### Testing & Debugging
- **LocalStorage**: Best score, difficulty, mute, reduced-motion settings stored under keys: `skyfighter.bestScore`, `skyfighter.difficulty`, `skyfighter.muted`, `skyfighter.reducedMotion`
- **Performance profiling**: Open DevTools → Performance tab, record game loop, inspect frame timing
- **Audio issues**: Check browser console for WebAudio errors; audio only initializes on user gesture
- **Touch testing**: Use DevTools device emulation or test on physical device

### Adding or Modifying Content
- **New enemy type**: Add entry to `ENEMY_TYPES` (line 502), set `minStage`, define visual in `drawPlane`
- **New boss**: Add design to `BOSSES` array (line 508), add 3 attack patterns to `BOSS_PATTERNS` (line 1372)
- **New theme**: Add sky gradient colors, cloud color, sun glow to `THEMES` (line 493)
- **Sound effect**: Add function to `sfx` object (line 624) using `beep()` and `noiseBurst()`
- **Difficulty tuning**: Modify `DIFFICULTIES` values (line 485) for spawn rates, speeds, health

### Code Style Conventions
- Strict mode enabled at script start (line 424)
- No external dependencies; all DOM queries cached at top of IIFE
- Event listeners attached early; state mutations happen during update phase only
- Physics: Delta-time multiplied by speed constant; coordinates in canvas pixels
- Colors: Hex strings or rgba() for transparency
- Animation frames: Use `frame` counter (incremented each update) for deterministic animations

## Important Implementation Details

### Player Movement Priority
Input processing (lines 1152–1170):
1. Keyboard/joystick input has highest priority (checked first)
2. If neither active (magnitude ~0), mouse steering activates
3. Movement speed: 260 px/s
4. Boundaries: Player clamped within canvas ±w/2, h/2

### Invulnerability Mechanics
- After taking damage: 1.5s invulnerability window
- Player blinks (every 4 frames) during invulnerability
- Chain multiplier resets on damage

### Enemy Shooting Behavior
- Interceptors inherit `weave: true` flag; follow sine-wave path
- Two shot patterns: 'spread' (3-shot fan) and 'aimed' (single homing shot at player)
- Shot timer randomized: 60–150 frames between shots

### Radial Bullet Movement
- Boss and some enemy patterns fire "radial" bullets: use `dx`, `dy` velocity components
- Radial bullets moved each frame: `x += dx * dt; y += dy * dt`
- Standard bullets moved via `y -= speed * dt` and optional horizontal vx

### Adaptive Quality (lines 474–483)
- Tracks last 45 frame times; calculates average
- If average > 1/40 (25ms) and quality < 2: step down to reduce particles
- If average < 1/56 (18ms) and quality > 0: step up to increase visual fidelity
- Quality affects particle counts and shadow/glow effects in rendering

### Reduced Motion Preference
- Respects OS preference: `prefers-reduced-motion: reduce`
- Can be overridden by user toggle (🎆 button)
- Disables: screen shake, animations, particle effects, vignette pulse, flash effects
- CSS class `reduced-motion` applied to body for animation media queries

### Storage Keys
- `skyfighter.bestScore`: Persistent high score
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.muted`: Audio mute state
- `skyfighter.reducedMotion`: User preference override (overrides OS setting)

## Key Code Patterns

### Particle Spawning
```javascript
spawnExplosion(x, y, color, count);  // Creates count particles at (x,y) with random velocity
```

### Adding Score
```javascript
addScore(base);  // Adds base × multiplier to score, extends chain timer
```

### Drawing Game Objects
- Player plane: `drawPlane(x, y, w, h, color, darkColor, flame)`
- Boss: `drawBossShip(b)` uses design colors and telegraphing visual state
- All drawn relative to translated canvas context

### Event Handling Pattern
- Input events enable audio context immediately: `ensureAudio()`
- Audio context suspended on some browsers until user gesture
- Mute state (`muted` boolean) checked in every `beep()` and `noiseBurst()` call

## Future Enhancement Considerations

- **Particle pooling**: Reuse particle objects instead of creating/destroying to reduce GC pressure
- **WebGL rendering**: Would improve performance on older devices
- **Mobile optimization**: Further refine touch controls, landscape orientation lock
- **Leaderboard**: Server integration for global high scores
- **Level editor**: Allow creation of custom boss patterns and enemy waves
- **Story mode**: Narrative elements, cutscenes between stages

---

Last updated: 2026-10-14
