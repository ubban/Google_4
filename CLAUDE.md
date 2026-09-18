# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a self-contained HTML/Canvas application. The entire game logic, rendering, and audio synthesis is in a single 1669-line `index.html` file with embedded JavaScript.

**Key Features:**
- Player aircraft with switchable weapons and smart bombs
- Three difficulty levels (easy, normal, hard)
- Boss battles with unique visual designs
- Multiple visual themes (6 distinct theme sets)
- Touch and keyboard controls with adaptive joystick UI
- Procedural Web Audio synth system (no external audio files)
- Adaptive performance management based on frame timing
- LocalStorage persistence (best score, settings, difficulty)
- Accessibility features (screen reader support, reduced motion preference)

## Quick Start

### Running the Game
- Open `index.html` directly in a web browser
- Works on desktop, tablet, and mobile (touch controls auto-enable on touch devices)
- No build process, no dependencies, no server required

### Testing the Game
Manual testing is the primary approach (no automated test suite). Test paths:
- **New game start:** Click START button, verify game initializes to correct difficulty
- **Player movement:** WASD or arrow keys (desktop) / joystick (touch)
- **Weapons:** Press E/Z to cycle weapons, verify weapon name and level display updates
- **Difficulty levels:** Click difficulty buttons before game start, verify enemy spawn rates and health differ
- **Boss battles:** Survive to stage completion (18 kills), verify boss appears and bar fills correctly
- **Game over:** Lose all lives, verify score/best score display and new best score notification
- **Touch controls:** Test on mobile device or browser DevTools device emulation
- **Audio:** Verify sound effects play; test mute button and audio context suspension recovery
- **LocalStorage:** Close browser and reopen, verify best score and settings persist

## Architecture

The game is structured as a single IIFE (Immediately Invoked Function Expression) encapsulating all game state and logic. Understanding the code requires knowing these major sections:

### 1. **DOM Setup & Constants** (lines 426–527)
- All HTML element references (canvas, buttons, HUD elements)
- LocalStorage keys for persistence
- Difficulty configurations with spawn rates and balance parameters
- Enemy type definitions (grunt, interceptor, tank) with visual and gameplay attributes
- Boss definitions (6 boss types) with unique color schemes and HP multipliers
- Visual themes (6 themes) with color palettes and shader values

### 2. **Audio System** (lines 542–654)
- WebAudio API synth engine: generates all sound effects procedurally (no asset files)
- Two main functions:
  - `beep(freq, dur, type, vol, opts)`: Oscillator-based tones with frequency sweep and filtering
  - `noiseBurst(dur, vol, opts)`: Filtered noise for percussion effects
- Supports spatial audio: pan values computed from screen X position
- `ensureAudio()`: Lazy-initializes AudioContext (required by autoplay policy)
- Master gain and dynamics compressor for clean output

### 3. **Game State** (scattered, but centralized around `update()` and `draw()`)
- Player: `player` object with `x, y, w, h, hp, lives, bombs, weapon, weaponLvl, chargeTime`
- Enemies: `enemies` array; boss: `boss` object
- Projectiles: `playerBullets` and `enemyBullets` arrays
- Power-ups: `powerUps` array (health, bombs, weapons)
- Particles: `particles` array for visual effects
- Multiplier system: `multiplier` tracks kill streak, affects score and weapon XP

### 4. **Rendering** (lines 1544–1637)
- Single `draw()` function handles all canvas rendering
- Background: gradient sky with parallax cloud layers
- Objects: custom `drawPlane()` for player/enemies, `drawBossShip()` for bosses
- HUD: Separate HTML elements updated each frame, not canvas-rendered
- Performance optimizations: particle/glow rendering omitted at lower quality levels

### 5. **Game Loop** (lines 1640–1652)
- `loop(now)` runs via `requestAnimationFrame`
- Computes delta time and clamps to max 50ms to prevent spiral on lag
- Calls `update(dt)` if game is running and not paused
- Calls `draw()` after each update
- Performance tracking: measures frame times to adapt quality level

### 6. **Update Logic** (referenced in `update()` function, lines ~700–1500)
- Enemy spawning: Controlled by difficulty-dependent spawn rates; enemy type probability increases by stage
- Collision detection: AABB (axis-aligned bounding box) for bullets, simpler circle checks for power-ups
- Boss AI: Moves predictably, shoots in patterns, enters damage state briefly after hit
- Weapon mechanics: 3 weapon types with level progression; charging increases fire rate and damage
- Combo/multiplier: Increases with consecutive kills, resets on miss or hit taken
- Stage progression: 18 kills advance to next stage; boss appears on stage completion
- Game over: Triggered when lives reach 0; displays stats and best score comparison

### 7. **Input Handling** (touch and keyboard)**
- Keyboard: WASD/arrows for movement, space/click to fire, E/Z to cycle weapons, Space for bombs
- Touch: Joystick in bottom-left, fire button bottom-right, bomb button top-right
- Joystick behavior: radius-based, centered in touch zone, updates continuously
- Pause: P key or pause button; can resume from pause overlay

## Modifying the Game

### Balancing Enemy Difficulty
Edit the `DIFFICULTIES` object (line 485–489):
- `spawnBase` and `spawnMin`: Adjust overall enemy frequency
- `enemySpeed`, `enemyShootChance`: Tweak attack patterns
- `lives` and `bombs`: Change starting resources
- `bossHpMul`: Scale boss health across all bosses

Example: To make hard mode even harder, lower `spawnMin` or raise `enemyShootChance`.

### Adding a New Visual Theme
Add an entry to the `THEMES` array (lines 493–500):
```javascript
{ sky: ['#XXXXXX', '#XXXXXX', '#XXXXXX'], cloud: 'R,G,B', sun: 'R,G,B', name: 'Theme Name' }
```
- `sky`: 3 color stops for the radial gradient background
- `cloud` and `sun`: RGB values for parallax cloud and sun effects
- Theme is randomly selected at game start

### Adding a New Enemy Type
Add an entry to `ENEMY_TYPES` (lines 502–506):
```javascript
newtype: { w: SIZE, h: SIZE, hp: HP, speedMul: SPEED, color: '#HEX', dark: '#HEX', score: PTS, minStage: STAGE, weave: BOOL }
```
- `weave`: If true, enemy zigzags while moving
- `minStage`: First stage this enemy appears (1-indexed)
- Color pair: `color` for main body, `dark` for shading/highlights

### Adding a New Boss
Add an entry to `BOSSES` (lines 508–515):
```javascript
{ name: 'BOSSNAME', hullTop: '#HEX', hullBottom: '#HEX', wing: '#HEX', core: '#HEX', hpMul: MULT }
```
- Colors define the boss appearance; core color is weak point glow
- `hpMul`: Multiplies base HP from difficulty settings
- Boss behavior (movement, firing pattern) is hard-coded in the `update()` function; search for "boss" to customize

### Weapon Behavior
Player has 3 fixed weapons; behavior is in `update()` under bullet firing logic:
- Weapon 0: Narrow spread, fast
- Weapon 1: Medium spread, balanced
- Weapon 2: Wide spread, slow
- Level progression handled by `weaponLvl`; check `update()` for damage scaling

### Audio Tuning
Adjust `masterGain.gain.value` (line 556) to change overall volume. Individual sounds call `beep()` or `noiseBurst()` with specific parameters; search for these to adjust sound design.

### Accessibility & Settings
- Screen reader support: `announce()` function (line 468) updates `#srLive` ARIA-live region
- Reduced motion preference: Detected from `prefers-reduced-motion` media query; controlled by toggle button
- Mute toggle: Stored in LocalStorage; checked before each audio call

## Performance & Optimization

### Adaptive Quality System
The game monitors frame time (lines 476–483) and adjusts rendering:
- `quality` level 0 (full): All particles and glow effects
- `quality` level 1 (medium): Reduced particles
- `quality` level 2 (low): Minimal particles, no glow

Targets 40 FPS lower bound; throttles up to 56 FPS if headroom exists.

### Canvas Rendering
- Single `canvas` 2D context; no WebGL or layers
- Drawing order: sky → clouds → player → enemies → boss → bullets → power-ups → particles
- No double-buffering needed (canvas handles it)

### Collision Performance
- Enemies use simple AABB checks; no spatial partitioning (acceptable for ~30 concurrent objects)
- Bullet-enemy checks are O(n²) but rarely a bottleneck given spawn limits

## Common Tasks

### Finding Key Game State Variables
Search for these in `update()` and event listeners:
- `running`, `paused`: Game state flags
- `player`: Main character object
- `enemies`, `boss`: Actor arrays
- `playerBullets`, `enemyBullets`: Projectile tracking
- `powerUps`: Collectible drop list
- `multiplier`, `score`: Score/combo state
- `stage`, `killsThisStage`: Progression tracking

### Debugging
- Open browser DevTools (F12)
- Check Console for errors (none should appear in normal play)
- Break in `loop()` to inspect game state: `player.x`, `enemies.length`, etc.
- Modify constants (like `KILLS_PER_STAGE`) in DevTools console to test quickly

### Browser Compatibility
- Requires `canvas` 2D context (IE9+, all modern browsers)
- Web Audio API optional (game runs silent if AudioContext unavailable)
- Touch API for mobile; falls back to keyboard on desktop
- LocalStorage optional (game runs without persistence if blocked)

## File Structure

```
.
├── CLAUDE.md          ← You are here
├── README.md          ← Brief project description
└── index.html         ← Complete game (1669 lines)
    ├── HTML (head, body structure with canvas and overlay UI)
    └── JavaScript (full game engine in <script> IIFE)
```

## Future Development Considerations

- **Code splitting**: As the game grows, move game logic to separate modules (WebWorkers or module scripts)
- **Asset pipeline**: To use external audio/graphics, would need a build step and asset loader
- **Save system**: Currently only best score; could extend to level select, ship skins, etc.
- **Networking**: Scoreboard or leaderboard would require backend
- **Mobile app**: Could wrap in Cordova/Capacitor for app stores
