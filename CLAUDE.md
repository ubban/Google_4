# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. A fully self-contained single-file web game with no external dependencies, implemented in vanilla HTML, CSS, and JavaScript.

## Architecture

The entire game runs in a single `index.html` file (~1700 lines):

### Top-Level Structure
- **HTML**: Minimal DOM structure with canvas and overlay UI
- **CSS**: Complete styling (layout, animations, controls, responsive sizing)
- **JavaScript**: Game loop, physics, rendering, input handling, audio synthesis

### Key Game Systems

**Input & Controls**
- Keyboard: Arrow keys + WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- Mouse: Move to steer, click to fire
- Touch: Joystick on left, action buttons on right (Fire/Bomb)
- Input systems never fight—keyboard/joystick takes priority over mouse

**Rendering Pipeline**
- Canvas-based immediate mode rendering (cleared and redrawn every frame)
- Adaptive quality system: tracks framerate, reduces particle/glow effects at low FPS
- Sky gradient, clouds, enemies, player, bullets, particles, UI HUD all canvas-drawn
- Theme system: 6 different color themes that cycle per stage

**Game State & Updates**
- Flat object structure: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Single `update(dt)` call per frame handles movement, collision, spawning
- Collision detection: axis-aligned `rectsOverlap()` used for all hit tests

**Weapon System**
- Two weapons: Vulcan (spread shots) and Laser (piercing shots)
- Each has 3 upgrade levels; at max level, switches to other weapon at level 1
- Firing rate differs per weapon (laser fires faster)
- Spread/pierce behavior controlled by level in `fireWeapon()`

**Enemy System**
- Three enemy types: grunt (basic), interceptor (fast), tank (slow, tough)
- Spawning difficulty ramps with frame counter
- Enemies weave side-to-side (configurable per type) and shoot on timers
- Two firing patterns: spread (predictable) or aimed (tracks player)

**Boss System**
- 6 different boss designs (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN)
- Each has unique attack patterns (3 patterns per boss, cycles through them)
- Boss telegraph animation (2.2s wind-up) before attacking
- Boss health bar shown in HUD when fighting
- Bosses spawn after 18 kills per stage

**Scoring & Progression**
- Kill combo system: consecutive kills within 2.2s multiplier (1.0x base, 2.0x at 5+ kills)
- Score increases per stage, enemies harder, new boss every stage
- 6 stages before cycling back (theme and boss rotate)

**Difficulty Modes**
- Easy, Normal, Hard: affects spawn rate, enemy speed, enemy accuracy, player lives, bomb count, boss HP
- Stored in localStorage, player can switch between games

**Audio**
- No external audio files; all sounds generated via WebAudio API (oscillators + noise)
- Synthesized SFX for every action: shoot, laser, explosions, hit, bomb, power-up, boss telegraph, boss down, stage clear, UI click, game over
- Master gain + compressor for dynamic range control
- Stereo panning based on world position
- Mute toggle persisted to localStorage

**Visual Effects**
- Particles: collision sparks, explosions (count scales with quality setting)
- Flash screen on hit/bomb (white overlay with animation)
- Vignette pulse (red screen edge) when health is critical
- Screen shake for impacts, magnitude configurable
- Combo text pop-up at kills milestone
- Stage banner with fade animation
- Reduced motion mode: removes all animations if user prefers (respects `prefers-reduced-motion`)

**Game Loop**
- `requestAnimationFrame(loop)` at 60 FPS (delta time clamped to 50ms max)
- Update → Draw → Request next frame
- Separate pause state pauses update but continues rendering overlay

## Common Development Tasks

### Running the Game
1. Open `index.html` in a web browser
2. Game starts with menu overlay; click "Start Game" to begin
3. Use keyboard/mouse/touch controls to play

### Testing a Specific System
- **Weapons**: Lines 1124–1146 (`fireWeapon()`) — adjust spread/pierce in level conditions
- **Enemies**: Lines 1022–1041 (`spawnEnemy()`) — modify `ENEMY_TYPES` constants (lines 502–506) to change stats
- **Bosses**: Lines 1043–1056 (`spawnBoss()`) and 1411–1439 (`updateBoss()`) — edit `BOSSES` array (508–515) or `BOSS_PATTERNS` (1372–1409)
- **Difficulty**: Lines 485–489 — tweak `DIFFICULTIES` object spawn rates, enemy speed, HP multipliers
- **Audio**: Lines 624–676 — adjust frequency/duration/volume in the `sfx` object methods

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` object (line 502+): define `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add to `choices` filter in `spawnEnemy()` line 1026 if stage-gated
3. Add spawn chance in random roll (lines 1028–1030)
4. If it has unique death SFX, add method to `sfx` object and add entry to `EXPLODE_SFX` map (line 678)

### Adding a New Boss Attack Pattern
1. Add function to fire bullets (see `fireRadial`, `fireAimedSpread`, `fireCross`, etc. starting line 1324)
2. Create pattern array: 3 attack functions, add to `BOSS_PATTERNS` array (line 1372)
3. Each pattern receives boss object `b`; fire bullets into `enemyBullets` array
4. Patterns are called in `updateBoss()` after telegraph finishes (line 1427)

### Adjusting Game Balance
- **Enemy spawning too fast/slow**: Adjust `spawnBase`, `spawnMin` in `DIFFICULTIES` (line 485+)
- **Boss too hard**: Increase `bossHpMul` or reduce `hpMul` in specific `BOSSES` entry (line 508+)
- **Player movement too slow**: Change `player.speed` in `resetGame()` line 877
- **Weapon fire rate**: Adjust `fireDelay` in input handling line 1177

### Modifying Visual Appearance
- **Theme colors**: Edit `THEMES` array (line 493+) — `sky` is gradient stops, `cloud` and `sun` are RGB strings
- **Player plane color**: Change `drawPlane()` call in draw function (line 1558) — first color is primary, second is dark
- **Enemy colors**: Edit `ENEMY_TYPES` `color` and `dark` fields (line 502+)
- **Boss colors**: Edit specific `BOSSES` entry `hullTop`, `hullBottom`, `wing`, `core` (line 508+)

### Canvas Sizing & Responsive Layout
- Base canvas size: 480×720 (defined line 753)
- Responsive scaling: `resize()` function (line 754–765) scales canvas up to 1.6× if screen is larger
- All responsive UI (HUD, buttons, text) uses `clamp()` CSS units for scaling
- Safe area insets for notched phones: `env(safe-area-inset-*)`

## Key Code Locations

- **Game loop entry**: Line 1665 (requestAnimationFrame start)
- **Update logic**: Line 1148 (update function)
- **Draw logic**: Line 1520 (draw function)
- **Collision detection**: Line 1232–1295 (enemy/bullet, enemy/player, boss/player, powerup/player)
- **Input handling**: Lines 768–861 (keyboard, mouse, touch, joystick)
- **HUD rendering**: Lines 680–696 (hearts, bombs display)
- **Pause/Game Over**: Lines 1469–1504 (endGame, togglePause)
- **localStorage persistence**: Lines 463–467 (storage key constants), used throughout for best score, mute, difficulty, reduced motion

## Performance Considerations

- Adaptive quality system (line 474–483): monitors average frame time, reduces particle/glow counts and shadow blur at low FPS
- Particle count scales with quality: full (14 particles), medium (8), low (5)
- Shadow effects disabled at low quality to reduce draw calls
- Collision detection uses simple AABB (axis-aligned bounding box)—O(n²) but acceptable for small enemy counts
- requestAnimationFrame naturally throttles to screen refresh rate (usually 60 FPS on desktop, variable on mobile)

## Browser Compatibility

- Requires: Canvas 2D, requestAnimationFrame, localStorage, CSS Grid/Flexbox, Web Audio API
- Tested on: modern Chrome, Firefox, Safari, Edge
- Mobile: Touch controls automatically enabled on touch devices (`ontouchstart` or `maxTouchPoints`)
- Accessibility: Reduced motion support, screen reader announcements via `#srLive` div, semantic button labels

## Notes for Future Work

- To add a new stage mechanic, update stage progression in `killBoss()` (line 1441+)
- To add persistent progression (character upgrades across games), extend localStorage usage
- To extract into modular components, separate `spawnEnemy`, boss patterns, and drawing functions into objects
- Audio context is lazy-initialized on first user input (`ensureAudio()` line 545) to comply with autoplay policies
