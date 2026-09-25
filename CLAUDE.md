# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based, Raiden-style vertical shoot-'em-up game. The entire game is implemented as a single HTML file (index.html) with embedded CSS and JavaScript. It features:
- Player-controlled aircraft that can switch between two weapons (Vulcan and Laser)
- Multiple enemy types with varied behaviors and attack patterns
- 6 unique boss encounters with procedurally-generated attack patterns
- A scoring system with combo multipliers
- Procedurally-synthesized sound effects via WebAudio API
- Full support for keyboard, mouse, and touch input on all platforms
- Difficulty scaling and progressive challenge through 6+ stages

## Getting Started

### Running the Game
The game runs directly in any modern web browser. No build step or server required:
- **Local development**: Open `index.html` directly in a browser
- **Testing**: Use any HTTP server (e.g., `python3 -m http.server`, VS Code Live Server)
- **Playing**: Point any browser to the file path or URL

### Key Files
- `index.html` — Complete game implementation (HTML + CSS + JavaScript, ~1670 lines)
- `README.md` — Minimal placeholder documentation

## Architecture & Code Organization

### Single-File Structure
The game is contained in one HTML file for simplicity and portability. The file is divided into logical sections:

1. **HTML Head (lines 1-359)**: Meta tags, favicon, comprehensive CSS for UI and game styling
2. **HTML Body (lines 361-420)**: Game container, canvas element, HUD elements, overlay UI, touch controls
3. **JavaScript (lines 422-1667)**: Complete game logic in an IIFE (Immediately Invoked Function Expression)

### Core JavaScript Sections (IIFE, order matters)

**DOM References & Configuration (lines 426-527)**
- Caches all DOM elements for performance
- localStorage constants for persistence (best score, mute state, difficulty, reduced motion)
- Difficulty configurations with spawn rates, enemy stats, and multipliers

**Adaptive Performance System (lines 473-483)**
- Tracks frame times and dynamically reduces visual quality (3 levels: full/medium/low)
- Used to disable shadow effects and particle counts on lower-end devices

**Audio System (lines 542-676)**
- WebAudio Context initialization with compressor
- Procedural synthesizer: `beep()` for tonal sounds, `noiseBurst()` for noise-based effects
- Pan support for spatial audio feedback
- SFX object with pre-defined sounds for game events (shoot, explode, hit, bomb, powerup, etc.)

**Input Handling (lines 767-861)**
- Keyboard input via event listeners (WASD/Arrows for movement, Space/Shift+X for bomb)
- Touch input with joystick implementation for mobile
- Mouse steering: smooth lerp toward cursor position when hovering canvas
- Input priority: keyboard/joystick > mouse (never interfere with each other)

**Game State Variables (lines 864-869)**
- `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Score, lives, bombs, wave/stage counters, combo chain state

**Game Loop (lines 1640-1652)**
- Uses `requestAnimationFrame` for smooth 60fps rendering
- Delta-time capped at 50ms to prevent spiral of death
- Three phases: `trackPerf()`, `update(dt)`, `draw()`

### Key Game Systems

#### Player System
- Position: center-bottom of screen, bounded to playfield
- Movement: 260 px/s, input-driven or mouse-guided
- Weapons: Vulcan (spread shots) and Laser (piercing) with 3 levels
- Invulnerability frames after taking damage (1.5s)
- Cooldown between shots (0.15s for Vulcan, 0.1s for Laser)

#### Enemy System
Three enemy types defined in `ENEMY_TYPES`:
- **Grunt** (red): 1 HP, 1x speed, 10 points
- **Interceptor** (orange): 1 HP, 1.7x speed, weaving motion, 15 points
- **Tank** (purple): 4 HP, 0.55x speed, 35 points
- Ships spawn at random x positions and scroll down
- Shooting patterns: "spread" (3-way) or "aimed" (toward player)
- Enemies with weave behavior oscillate horizontally

#### Boss System (lines 1043-1440)
- 6 boss designs with unique HP multipliers and color schemes
- Boss spawns after 18 enemy kills per stage
- Boss attacks follow pre-defined pattern sequences:
  - Pattern 0 (VANGUARD): Radial burst, aimed fan, V-fan spray
  - Pattern 1 (BEHEMOTH): Heavy columns, slow homing, dense radial
  - Pattern 2 (SPECTER): Teleport dash, spiral, double aimed shots
  - Pattern 3 (DREADNOUGHT): 8-way cross, homing volley, dense radial
  - Pattern 4 (WRAITH): Teleport barrage, double ring, cross-spread
  - Pattern 5 (TITAN): Heavy columns, full ring, relentless homing
- Boss telegraph phase (22 frames) before attack fires
- Boss health bar at top of screen

#### Scoring & Combo System
- Base score per enemy: 10-35 points (varies by type)
- Bomb kill: 5 points each
- Boss kill: 500 points
- Combo multiplier: starts at 1x, increases to 10x over time (1 point per 5 consecutive kills)
- Combo chain resets if player takes damage or waits >2.2 seconds between kills
- Every 10 kills, a "X KILLS!" popup appears

#### Power-Up System
Three power-up types (spawn on enemy death, probability 6%-34%):
- **Life**: Restore one heart (up to max)
- **Bomb**: Add one bomb (up to max)
- **Weapon**: Upgrade weapon level (1→2→3) or swap weapon (back to level 1)

#### Difficulty Levels
Three difficulty presets stored in `DIFFICULTIES`:
- **Easy**: Spawn rate 75→40, 1.0x speed, 45% shoot chance, 4 lives, 3 bombs, 0.8x boss HP
- **Normal**: Spawn rate 58→26, 1.4x speed, 70% shoot chance, 3 lives, 2 bombs, 1.0x boss HP (default)
- **Hard**: Spawn rate 40→18, 1.9x speed, 95% shoot chance, 3 lives, 2 bombs, 1.3x boss HP

#### Visual Themes
6 themes cycle through stages in `THEMES`:
1. Daylight Front (blue sky)
2. Sunset Corridor (orange/purple)
3. Night Approach (dark blue)
4. Storm Front (green)
5. Arctic Wastes (cyan)
6. Volcanic Ridge (red/orange)

#### Collision & Physics
- Simple rectangular AABB collision via `rectsOverlap()`
- Bullets: instant removal on enemy hit or screen exit
- Enemy bullets: radial movement for boss attacks, linear for regular enemies
- Particles: physics-based (vx/vy with gravity/drag effects)

#### UI & HUD
- Top HUD shows: Score, Best Score, Combo multiplier, Weapon name/level, Hearts (lives), Bombs
- Boss health bar slides in when boss spawns
- Combo popups show at 10-kill milestones
- Stage banners announce stage transitions and boss warnings
- Pause overlay with ability to change difficulty before game start

#### Accessibility Features
- Screen reader support via `#srLive` live region
- Reduced motion preference detection and override toggle
- Animations disabled for users with `prefers-reduced-motion`
- Vignette effect (critical low health) respects reduced motion setting

#### Settings & Persistence
Four localStorage entries:
- `skyfighter.bestScore` — Best score across sessions
- `skyfighter.muted` — Audio mute state
- `skyfighter.difficulty` — Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion` — Motion reduction preference

### Canvas Sizing
- Game resolution: 480×720 (portrait aspect)
- Scales up to fit window (max 1.6x scale factor)
- Responsive to window resize and orientation change

### Rendering Pipeline
1. Clear canvas and apply screen shake (if active)
2. Draw background (theme-based gradient sky)
3. Draw sun glow (radial gradient)
4. Draw parallax clouds (two layers at different speeds)
5. Draw player (plane graphic, flame when firing)
6. Draw player bullets (with glow if quality = 0)
7. Draw enemy bullets (red circles with glow)
8. Draw enemies (plane graphics with rotation)
9. Draw boss (if active)
10. Draw power-ups (rotating squares)
11. Draw particles (expanding/fading circles)

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (lines 502-506)
2. Set w, h, hp, speedMul, color, dark, score, minStage
3. Optional: set `weave: true` for horizontal oscillation behavior
4. Add explosion SFX mapping in `EXPLODE_SFX` if unique (line 678)
5. Update spawn chances in `spawnEnemy()` (lines 1022-1041)

### Adding a New Boss
1. Add entry to `BOSSES` array (lines 508-515)
2. Create new attack pattern functions (e.g., `fireWhatever()`)
3. Add pattern array to `BOSS_PATTERNS` (lines 1372-1409)
4. Boss will be selected by `(stage - 1) % BOSSES.length`

### Adding a New Attack Pattern
1. Write helper function that calls `enemyBullets.push()` (pattern functions at lines 1324-1370)
2. Helper receives boss object `b` as parameter
3. Bullets have: x, y, w, h, radial (boolean), dx, dy (for radial) or speed/vx (for linear)
4. Add function call to `BOSS_PATTERNS` array

### Adjusting Difficulty
1. Modify `DIFFICULTIES` object (lines 485-489)
2. Spawn rates: `spawnBase` (normal spawn interval) and `spawnMin` (minimum as game progresses)
3. Enemy stats: `enemySpeed`, `enemyShootChance`
4. Player resources: `lives`, `bombs`, `bossHpMul`

### Adding Visual Polish
- Screen shake: call `screenShake(magnitude, duration)` on impact events
- Flash overlay: call `flashHit()` or `flashBomb()` for hit feedback
- Combo popup: call `showComboPop(text)` for milestones
- Animations: use CSS `@keyframes` in `<style>` section; check `reduce-motion` for accessibility

## Performance Considerations

- **Quality scaling**: Shadows, glow effects, particle count reduced at quality > 0
- **Object pooling**: Not used (creates fresh arrays per frame); acceptable for this game's scale
- **Collision checks**: O(n²) naive approach; acceptable for ~20 enemies
- **Canvas draw calls**: Batched by entity type for efficiency
- **Audio**: Lazy initialization; AudioContext created on first input to avoid browser blocking

## Testing Tips

- **Browser DevTools Console**: Modify game variables directly (e.g., `score = 5000`, `stage = 6`)
- **Difficulty testing**: Change `DIFFICULTIES[difficulty]` or modify localStorage
- **Performance**: Toggle quality in browser DevTools → reduce screen size to test low-quality rendering
- **Touch debugging**: Simulate touch in DevTools (Device Toolbar → Touch events)
- **Audio debugging**: Check AudioContext.state; mute button stores state in localStorage

## Future Enhancement Ideas

- Enemy movement patterns (zigzag, circular orbits)
- Special weapon pickups (spread, homing, beam)
- Multi-stage boss fights with phase transitions
- Leaderboard integration
- Mobile app wrapper (Cordova, React Native)
- Progressive difficulty curve over more stages
- Procedural level generation
- Multiplayer co-op
