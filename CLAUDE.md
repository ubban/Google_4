# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file browser game (index.html) with no external dependencies. The entire game is built with vanilla HTML, CSS, and JavaScript using the Canvas API and Web Audio API.

### Key Features
- 3 difficulty levels (Easy/Normal/Hard) affecting enemy spawn rates, health, and bullet patterns
- 6 unique boss encounters with distinct attack patterns
- Weapon progression system (Vulcan spreader and Laser with pierce capability)
- Touch controls, keyboard input, and mouse steering
- Screen shake, flash effects, particle system
- WebAudio synth for all sound effects (procedurally generated, no audio files)
- Adaptive quality system that reduces particle count/glow effects on low-end devices
- Accessibility features: reduced motion support, screen reader announcements
- LocalStorage persistence for best scores and player settings

## Running the Game

Simply open `index.html` in a web browser. No build step, no local server required—the game runs immediately.

**For development/testing:**
- Use a local web server if you need to test cross-origin features: `python -m http.server 8000` or `npx serve`
- Test on different screen sizes and orientations (game scales to fit viewport)
- Test touch controls on mobile devices or use browser dev tools device emulation

## Game Architecture

The codebase is organized as a single-file application (~1670 lines) split into logical sections:

### Game State & Setup (lines 864–911)
- Player, bullets, enemies, particles, boss objects
- Score, lives, stage, chain/multiplier system
- Difficulty data (DIFFICULTIES object, line 485)
- Theme data (THEMES array, line 493—6 sky backgrounds)
- Enemy type definitions (ENEMY_TYPES, line 502)
- Boss definitions (BOSSES, line 508)

### Input Handling (lines 767–861)
- Keyboard input (arrows, WASD, Space, X, Shift, P, Esc)
- Mouse movement for player steering + click to fire
- Touch controls with virtual joystick on mobile
- Pause toggle (P or Esc)

### Audio System (lines 543–676)
- Web Audio API context initialization
- Synth sound generation: `beep()` and `noiseBurst()` functions
- Sound effects library (sfx object): shoot, laser, explosions, powerups, boss attacks, game over
- Mute/unmute toggle persisted to LocalStorage

### Rendering (lines 680–638)
- `drawPlane()`: Draws player, enemies, boss ships
- `drawBossShip()`: Boss-specific renderer with design colors
- `drawClouds()`: Parallax cloud scrolling based on theme
- Canvas sizing with responsive scaling (`resize()` function)
- Screen shake applied via canvas translation
- Flash effects for hits/bombs (CSS animations)

### Game Logic (lines 1148–1302)
- `update(dt)`: Main frame update
  - Player movement (keyboard/joystick/mouse steering)
  - Fire weapon (cooldown-based)
  - Update all moving objects (bullets, enemies, particles)
  - Enemy spawning and behavior (weaving interceptors, aimed fire)
  - Collision detection between player/enemies/bullets
  - Combo/chain system (multiplier resets after 2.2 seconds without kills)
- Boss mechanics separated into `updateBoss(dt)` (line 1411)
  - Boss position tracking and wall bounces
  - Attack pattern telegraph system (visual/audio warning before attack)
  - Pattern execution via BOSS_PATTERNS array (line 1372)

### Boss Attack Patterns (lines 1324–1409)
Six bosses with unique attack patterns:
- **VANGUARD**: Radial bursts, aimed fans, V-shaped spreads
- **BEHEMOTH**: Columns, homing spreads, dense radials
- **SPECTER**: Teleport + spiral + double aimed shots
- **DREADNOUGHT**: 8-way cross, homing missiles, dense radials
- **WRAITH**: Teleport barrage, expanding rings, aimed crosses
- **TITAN**: Heavy columns, full rings, relentless homing

Each pattern fires enemy bullets using radial-style vectors (dx/dy) that are updated during boss update phase.

### Performance Optimization (lines 473–483)
- `quality` variable (0=full, 1=medium, 2=low)
- Adapts based on average frame time over 45 frames
- Reduces: particle count, glow effects, shadow blur on lower quality

### User Interface (lines 680–724)
- HUD stats: Score, Best, Multiplier, Weapon, Hearts, Bombs (dynamically rendered HTML)
- Overlay system for start screen, pause, game over
- Difficulty selection buttons
- Boss health bar (shown when boss present)
- Toast-like notifications: Combo popups, stage banners
- Screen vignette pulses when on last life

## Key Game Systems

### Collision Detection
- `rectsOverlap(a, b)` (line 1069): Simple axis-aligned bounding box collision
- Used for: bullets vs enemies, bullets vs boss, player vs enemies, player vs bullets, player vs powerups

### Scoring & Multiplier
- `addScore(base)` (line 1087): Increases chain counter, calculates multiplier (1 + floor(chain/5), capped at 10x)
- Multiplier persists for 2.2 seconds; breaks on enemy/boss collision
- Boss kill: +500 base score

### Weapon System
- **Vulcan**: Spreads to 3 lanes at level 3; pierces nothing
- **Laser**: Upgradeable pierce (1-3 hits); fires in multiple lanes at higher levels
- Weapon pickup spawns randomly on enemy kills (6% chance)
- Each pickup either upgrades level (max 3) or switches weapon type

### Bomb System
- Smart bomb clears all on-screen enemies and boss damage (24 HP)
- Limited ammo (2-3 based on difficulty); can be restocked with pickups
- Triggers full-screen flash and heavy screen shake

### Difficulty System
Affects: spawn frequency, enemy speed, shoot chance, lives, bomb count, boss HP multiplier

### Reduced Motion Preference
- Respects `prefers-reduced-motion` OS setting
- Toggle via settings button (🎆/🚫 icon)
- Disables: screen shake, flashing, pulsing animations, particle reduction
- Stored in LocalStorage

## Customization Points

### Add a New Boss
1. Add entry to BOSSES array (line 508) with colors/HP multiplier
2. Add attack patterns to BOSS_PATTERNS (line 1372) at matching index
3. Boss color/design is selected by `(stage - 1) % BOSSES.length`

### Add a New Enemy Type
1. Add to ENEMY_TYPES object (line 502) with stats and rendering color
2. Set `minStage` to control when it first appears
3. Enemy rendering in `draw()` (line 1603) handles tank HP bars automatically

### Add a New Theme/Skybox
1. Add to THEMES array (line 493) with sky gradient stops, cloud/sun colors, name
2. Theme is selected by `(stage - 1) % THEMES.length`
3. Themes cycle: Daylight → Sunset → Night → Storm → Arctic → Volcanic

### Tweak Game Balance
- Enemy spawn rates: Edit DIFFICULTIES (line 485)
- Weapon damage: Adjust damage in collision code (line 1259 for boss = 3 HP)
- Boss health: Formula at line 1046: `70 + stage * 35` multiplied by difficulty/design factors
- Powerup drop rates: Line 1060 (6% life, 8% bomb, 20% weapon)

## Notes for Development

- **No build process**: Edit index.html, refresh browser to see changes
- **LocalStorage keys** (lines 463–466): Best score, mute state, difficulty, reduced motion preference
- **Frame counter** (`frame` variable): Used for animations, enemy patterns, cloud scrolling; incremented every update
- **High-performance canvas drawing**: Glyph glyphs drawn with canvas instead of text (player plane, enemies, boss)
- **Touch handling**: Separate joystick system from keyboard; both feed into same movement vector (keyboard takes priority)
- **Pause state**: Freezes updates but keeps render loop running (allows pause overlay rendering)
- **Stage progression**: Incremented after boss kill; affects enemy types spawned, boss HP, theme selection, stage banner display
