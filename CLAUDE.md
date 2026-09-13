# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game built as a single-file HTML5 Canvas game. It's a fully self-contained game with no external dependencies — all rendering, audio synthesis, and game logic are embedded in `index.html`.

### Quick Start

- **Play the game**: Open `index.html` in a web browser
- **No build step required** — the game is standalone and ready to run immediately
- **No external assets** — sprites are drawn procedurally, music/SFX generated via Web Audio API

## Architecture Overview

The entire game is contained in a single `index.html` file (~1670 lines). The structure is:

1. **HTML/Markup** (lines 1-420)
   - Canvas element for rendering
   - HUD elements (score, lives, bombs, weapon status)
   - Boss health bar
   - UI overlays (title screen, pause, game over)
   - Touch controls for mobile
   - Accessibility elements (screen reader support, reduced motion toggle)

2. **CSS Styling** (lines 10-358)
   - Responsive design with `clamp()` for scaling across devices
   - Dark theme with subtle blur effects
   - Animations: heart beat (critical health), combo pop-ups, stage banners, flashes
   - Touch-safe buttons and controls
   - Safe-area support for notched devices

3. **JavaScript Game Logic** (lines 422-1666)
   - Single IIFE module pattern (everything is scoped to avoid globals)
   - State management (game state, player, enemies, bullets, etc.)
   - Update/render loop using `requestAnimationFrame`

## Key Game Systems

### Difficulty System (lines 485-489)
Three difficulty presets that scale enemy spawn rate, speed, shoot chance, health, and bombs:
- **Easy**: Slower enemies, more starting lives/bombs, weaker bosses
- **Normal**: Balanced difficulty (default)
- **Hard**: Aggressive spawning, fast enemies, more powerful bosses

### Enemy Types (lines 502-506)
Three enemy archetypes with different stats and behaviors:
- **Grunt** (red, 1 HP): Basic enemy, straight movement
- **Interceptor** (orange, 1 HP): Fast, weaving movement pattern
- **Tank** (purple, 4 HP): Slow, heavy, shows health bar

### Boss System (lines 508-514, 1372-1409)
Six bosses (VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN) with unique visual designs and firing patterns:
- Each boss has 3 distinct attack patterns (radial, aimed, spiral, columns, etc.)
- Bosses telegraph attacks with a glowing core
- Health scales with difficulty and stage progression
- Patterns cycle through on cooldown

### Weapon System (lines 871-872, 1124-1146, 1311-1322)
Two primary weapons with 3 upgrade levels:
- **Vulcan**: Spread shot, 3 levels unlock wider spread
- **Laser**: Single/multi-lane piercing beam, 3 levels add lanes and pierce depth
- Weapon upgrades cycle: level 1 → 2 → 3 → switch weapon → level 1

### Audio System (lines 542-676)
Web Audio API synthesizer with no external files:
- Uses oscillators (sine, triangle, sawtooth, square) + noise bursts
- Dynamic filtering and panning for spatial audio
- Individual SFX: shoot, laser, explosions (by enemy type), hit, bomb, power-up, boss effects, UI click, game over
- Master compressor and gain control
- Muted state persisted to `localStorage`

### Input Handling (lines 767-862)
Multiple control schemes:
- **Keyboard**: Arrow keys + WASD for movement, Space to fire, X/Shift for bomb, P/Esc for pause
- **Mouse**: Pointer movement to steer, click to fire
- **Touch**: Left side drag to move (virtual joystick), right-side buttons for FIRE/BOMB
- Keyboard and joystick take priority over mouse to avoid conflicts

### Scoring & Combo System (lines 1087-1101)
- Base score per enemy type (10–35 points)
- Kill chain builds a multiplier (1x every 5 kills, up to 10x)
- Chain resets if player is hit or after ~2 seconds of no kills
- Boss kill: flat 500 points (doesn't add to chain)
- Score displayed in real-time HUD, best score persisted to `localStorage`

### Visual Effects
- **Screen shake** (lines 726-730): Magnitude and duration, reduced in low-motion mode
- **Flash effects** (lines 698-709): Hit flash and bomb flash animations
- **Particle explosions** (lines 1073-1085): Color-coded by event type, quality-adjusted for performance
- **Combo pop** (lines 712-717): Large animated text for milestone kills
- **Vignette** (critical health pulsing red border)

### Performance Adaptation (lines 473-483)
Monitors frame time and auto-adjusts quality:
- **Quality 0** (full): All effects, glow, high particle count
- **Quality 1** (medium): Reduced particles, reduced glow
- **Quality 2** (low): Minimal particles, no glow shadows
- Targets ~40–56 FPS on average

### Accessibility Features
- **Reduced motion toggle** (lines 529-540): Disables animations and screen shake for users who prefer no motion
- **Screen reader support** (lines 390, 468-471): Live region announces game events
- **Keyboard controls**: Full keyboard-only play support
- **Mobile touch controls**: Full game playable on touch devices

## Data Persistence

Uses `localStorage` with these keys:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Mute state
- `skyfighter.difficulty`: Last selected difficulty
- `skyfighter.reducedMotion`: Motion preference

## Game Loop Flow

1. **Initialization** (line 1640+): Set up canvas, input handlers, game state
2. **Main loop** (lines 1640-1653): Called via `requestAnimationFrame`
   - Calculate delta time (capped at 50ms)
   - Track performance
   - If running and not paused: `update(dt)` then `draw()`
3. **Update** (lines 1148-1302): Logic updates
   - Player movement (keyboard/mouse/joystick priority)
   - Bullet updates and collision detection
   - Enemy spawning, movement, shooting
   - Boss AI and pattern execution
   - Power-up collection
   - Stage progression (spawn boss after 18 kills)
4. **Draw** (lines 1520-1638): Render all game objects
   - Background with sky gradient and theme-specific colors
   - Clouds (parallax scrolling)
   - Particles, bullets, enemies, player, boss
   - Power-ups (rotating)

## Stage & Boss Progression

- **Stages**: Continuous, no hard limit. Each stage spawns a new boss.
- **Boss spawn**: After player defeats 18 enemies (`KILLS_PER_STAGE`)
- **Theme rotation**: Visual theme cycles through 6 themes (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- **Boss rotation**: Boss types cycle through 6 designs with unique patterns and health scaling
- **Scaling**: Enemy speeds, health, spawn rates increase gradually with frame count (lines 1201-1202)

## Files & Structure

- **index.html** — The entire game (HTML, CSS, JavaScript)
- **README.md** — Project metadata (minimal)
- **.git/** — Version control

## Development Notes

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` (line 502)
2. Define shape in `drawPlane()` calls within enemy rendering
3. Add explosion SFX to `EXPLODE_SFX` (line 678) and `sfx` object if needed
4. Adjust `spawnEnemy()` (line 1022) probability thresholds

### Adding a New Boss
1. Add boss design to `BOSSES` array (line 508)
2. Create 3 attack functions in appropriate style
3. Add pattern array to `BOSS_PATTERNS` (line 1372)
4. Boss #6 will use TITAN patterns by default (cycle wraps)

### Tweaking Difficulty
Edit `DIFFICULTIES` object (line 485):
- `spawnBase/spawnMin`: Spawn rate (frames between enemies)
- `enemySpeed`: Multiplier on base enemy speeds
- `enemyShootChance`: Probability [0-1] of enemies firing
- `lives/bombs`: Starting player resources
- `bossHpMul`: Multiplier on base boss health

### Adding Visual Effects
- **Screen shake**: Call `screenShake(magnitude, duration)` (already used for hits, bombs, boss defeat)
- **Flash**: Call `flashHit()` or `flashBomb()`
- **Particles**: Call `spawnExplosion(x, y, color, count)`
- **Popup text**: Call `showComboPop(text)` or `showStageBanner(text)`

### Audio Synthesis
All SFX are procedural in the `sfx` object (line 624). To add a new sound:
1. Create a new function in `sfx` object
2. Use `beep(freq, duration, type, volume, opts)` for tones
3. Use `noiseBurst(duration, volume, opts)` for noise
4. Options: `sweepTo` (frequency ramp), `filterFreq`, `filterType`, `pan`, `delay`
5. Call `ensureAudio()` first if triggering from user input

### Testing Controls
- **Keyboard**: Arrow keys, Space, X, P
- **Mouse**: Move to steer, click to fire
- **Touch** (on mobile): Drag left side to move, tap FIRE/BOMB buttons
- Test pause, mute, motion toggle, difficulty change before starting

## Performance Tips
- Game auto-scales quality if frame time is high
- Keep particle count reasonable (explosion count adjusts with quality)
- Shadow blur is disabled at quality > 0
- Glow effects scale with quality level
- Reduced motion mode still plays game but skips animations

## Browser Support
- Requires `requestAnimationFrame`, Canvas 2D, Web Audio API
- Tested on modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile: Touch events, viewport meta, safe-area support
- Accessibility: Respects `prefers-reduced-motion` media query
