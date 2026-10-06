# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a single-file Raiden-style vertical shoot-em-up game built with HTML5 Canvas, plain JavaScript, and the WebAudio API. The entire game is implemented in `index.html` (~1670 lines) with no external dependencies or build process.

- **Entry point**: `index.html`
- **Type**: Web-based game
- **Runtime**: Browser (no Node.js, no build tools)
- **Testing/Development**: Open `index.html` directly in a browser

## How to Run

There is no build process. Simply open `index.html` in any modern web browser:
- Desktop: Drag `index.html` into browser or `open index.html` (macOS)
- Local server: `python3 -m http.server 8000` then visit `http://localhost:8000/index.html`
- When developing, auto-reload: Use browser DevTools or a local HTTP server with watch tools

## Architecture

### Single-File Structure
All game code is in `index.html`. The file structure is:
1. **HTML** (lines 361-420): DOM elements (canvas, HUD, buttons, overlay UI)
2. **CSS** (lines 10-359): Styles for canvas rendering, UI, animations, responsive layout, accessibility
3. **JavaScript** (lines 422-1667): Main game logic in an IIFE (Immediately Invoked Function Expression)

### JavaScript Architecture

The game is organized as a single IIFE with nested functions and modular sections:

#### Key Subsystems

1. **Game State** (lines 863-869)
   - `player`: Position, weapon, health, invulnerability
   - `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`: Arrays of game objects
   - `boss`: Boss entity during boss fights
   - `score`, `lives`, `stage`, `multiplier`: Game progress tracking

2. **Input Handling** (lines 767-861)
   - **Keyboard**: Arrow keys, WASD, Space (fire), X/Shift (bomb), P/Esc (pause)
   - **Mouse**: Move plane toward cursor, click to fire
   - **Touch**: Joystick on left, Fire/Bomb buttons on right
   - Touch is detected with `'ontouchstart' in window` and enables mobile UI

3. **Game Loop** (lines 1640-1665)
   - `requestAnimationFrame(loop)` callback
   - Delta-time calculation to decouple from frame rate
   - Calls `update(dt)` if running/not paused, then `draw()`
   - Frame rate capped at ~50ms per frame to prevent huge dt jumps

4. **Update Logic** (lines 1148-1302)
   - Player movement (keyboard, joystick, or mouse steering)
   - Weapon fire (cooldown-based, vulcan or laser)
   - Enemy spawning and behavior (weaving, aiming)
   - Collision detection (player vs enemies, bullets vs enemies, etc.)
   - Bomb and power-up logic
   - Chain/combo tracking for score multiplier

5. **Boss System** (lines 1411-1454)
   - `updateBoss(dt)`: Position, telegraphing attacks, pattern execution
   - 6 unique boss designs with unique attack patterns (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
   - Attack patterns rotate through arrays defined in `BOSS_PATTERNS` (lines 1372-1409)

6. **Rendering** (lines 1520-1638)
   - Canvas background: Sky gradient and sun glow based on theme
   - Cloud parallax animation
   - Plane rendering: Both player and enemy planes use procedural drawing
   - Boss rendering: Complex multi-part design with animated elements
   - Particle effects: Simple circles with fade
   - HUD updates: Score, lives, bombs, weapon level (DOM manipulation)

7. **Audio** (lines 542-676)
   - WebAudio synth with no external files
   - `ensureAudio()`: Lazy-init AudioContext (can be suspended)
   - `beep()`: Oscillator-based tones with sweep, filter, pan
   - `noiseBurst()`: Procedural noise for explosion/bomb sounds
   - Individual sound functions in `sfx` object: shoot, laser, explode (variants), hit, bomb, powerup, bossHit, stageClear, etc.
   - Mute toggle persisted to localStorage

8. **Performance Adaptation** (lines 473-483)
   - `quality` variable: 0 = full, 1 = medium, 2 = low
   - Tracks last 45 frame times, auto-degrades if avg > 25ms, upgrades if < ~18ms
   - Used to reduce particle count, glow effects, shadow blur

### Data Structures

#### Enemy Types (`ENEMY_TYPES`, lines 502-506)
- `grunt`: Red, 1 HP, baseline
- `interceptor`: Orange, 1 HP, weaves, appears stage 1+
- `tank`: Purple, 4 HP, slow, tanky, appears stage 2+

Each type has: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`, optional `weave`

#### Boss Designs (`BOSSES`, lines 508-515)
Array of 6 bosses cycled by stage. Each has: `name`, `hullTop`/`hullBottom` colors, `wing`, `core`, `hpMul`

#### Themes (`THEMES`, lines 493-500)
6 sky themes cycling by stage. Each has: `sky` (3-stop gradient), `cloud`, `sun` (RGB), `name`

#### Difficulties (`DIFFICULTIES`, lines 485-489)
Tuneable parameters per difficulty: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier

### Storage Keys

Persisted to localStorage:
- `skyfighter.bestScore`: Best score across sessions
- `skyfighter.muted`: Mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Reduced motion preference

## Gameplay Mechanics

1. **Weapons**: 
   - Vulcan (default): Spread at level 3, faster fire rate
   - Laser: Piercing shots, slower fire rate
   - Cycle via weapon power-ups; level 3 swaps weapon type

2. **Scoring**:
   - Base score per enemy type (grunt 10, interceptor 15, tank 35)
   - Kills increase chain counter; multiplier = 1 + floor(chain/5), capped at 10x
   - Chain breaks after 2.2 seconds without a kill
   - Boss kill = 500 base score (multiplied)

3. **Progression**:
   - 18 kills per stage trigger boss warning (1.2s delay)
   - Boss defeated → stage++, enemies reset
   - Stage determines theme, boss design, enemy difficulty ramp

4. **Boss Patterns**:
   - Each boss has 3 attack patterns that repeat
   - Telegraph phase: 22 frames (player sees warning glow)
   - Boss moves left/right, bounces at canvas edges

## Common Development Tasks

### Adding a New Enemy Type
1. Add object to `ENEMY_TYPES` with properties: `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add draw logic in `enemies.forEach()` in `draw()` (lines 1603-1613); e.g., health bar for tanks
3. Optional: Add sound effect to `sfx` object if explosion sound differs
4. Add explosion SFX key to `EXPLODE_SFX` mapping (line 678)

### Adding a New Boss
1. Add object to `BOSSES` array with: `name`, `hullTop`, `hullBottom`, `wing`, `core`, `hpMul`
2. Add attack pattern array to `BOSS_PATTERNS` (lines 1372-1409)
3. Boss rendering is in `drawBossShip()` (lines 970-1020); adjust colors/shape if needed

### Tuning Difficulty
Edit `DIFFICULTIES` object (lines 485-489):
- `spawnBase` / `spawnMin`: Enemy spawn rate (frames between spawns)
- `enemySpeed`: Multiplier on enemy movement
- `enemyShootChance`: Probability enemy fires (0.0–1.0)
- `lives`, `bombs`: Starting player health/bombs
- `bossHpMul`: Boss health multiplier

### Adding Visual Effects
- **Screen shake**: Call `screenShake(magnitude, duration)` (lines 726-730)
- **Flash**: Call `flashHit()` or `flashBomb()` (lines 698-709)
- **Combo popup**: Call `showComboPop(text)` (lines 712-717)
- **Stage banner**: Call `showStageBanner(text)` (lines 719-724)

### Adding Audio
1. Add function to `sfx` object (lines 624-676)
2. Call `beep(freq, dur, type, vol, opts)` for tones or `noiseBurst(dur, vol, opts)` for noise
3. Use `opts.pan` for stereo positioning (expects x coord, uses `panFor()` to convert)
4. Mute state is checked inside each function; `ensureAudio()` must be called before any beep/burst

### Modifying UI/Overlay
- Overlay is shown/hidden by toggling `overlay.hidden`
- Update `overlayTitle`, `overlaySub`, `statsRow` content as needed
- HUD stats update via DOM elements: `scoreEl`, `heartsEl`, `bombsEl`, etc.
- Responsive: CSS uses `clamp()` for fluid font sizing

## Testing Strategy

No formal test suite. Validate by:
1. **Manual testing**: Open in browser, play through stages
2. **Difficulty sweep**: Test all three difficulties, verify spawn/health scaling
3. **Edge cases**: 
   - Low frame rate: Browser DevTools → throttle CPU/network
   - Touch vs mouse vs keyboard input
   - Mute/motion toggle persistence (check localStorage)
   - Boss patterns trigger correctly
4. **Performance**: DevTools → Performance tab, record and check frame rate, ensure adaptive quality adjusts

## Key Constants & Tuning

- `BASE_W = 480, BASE_H = 720`: Canvas size (scales to fit window)
- `KILLS_PER_STAGE = 18`: Enemy kills before boss appears
- `frame`: Global frame counter (incremented each update; used for animation)
- Quality thresholds (lines 476-482): Avg frame time > 25ms degrades, < ~18ms upgrades

## Notes for Future Work

1. **Code Organization**: File is large; consider extracting subsystems into modules if splitting multi-file (e.g., `game.js`, `audio.js`, `rendering.js`) becomes necessary
2. **Accessibility**: Screen reader annotations exist (`#srLive`); enhance with more live region announcements
3. **Mobile Optimization**: Touch controls work; test on actual devices for latency/lag
4. **Sound Improvements**: WebAudio synth is functional but minimal; could add more complex instruments or load external audio (currently none)
5. **Save/Resume**: Currently only high score persists; could add mid-game save state
