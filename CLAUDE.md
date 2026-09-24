# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML application. The entire game (markup, styles, and logic) is contained in `index.html` (~1670 lines).

## Quick Start

- **Run locally**: Open `index.html` directly in any modern browser (Chrome, Firefox, Safari, Edge)
- **No build step required**: This is a client-side game with no server or build dependencies
- **Mobile-friendly**: Works on desktop, tablet, and mobile with touch controls

## Architecture

### Single-File Design

The entire game is in `index.html` with three sections:

1. **HTML** (lines 1-420): Canvas container + HUD UI + overlay dialogs
2. **CSS** (lines 10-358): Complete styling with animations and responsive layout
3. **JavaScript** (lines 422-1666): Core game logic in an IIFE (Immediately Invoked Function Expression)

### Core Game Loop

The game uses `requestAnimationFrame` to drive the main loop (`loop()` function, line 1640):

- **update(dt)**: Updates game state each frame (physics, collisions, spawning, etc.)
- **draw()**: Renders everything to canvas
- **trackPerf(dt)**: Monitors frame times and adjusts quality dynamically (3 levels: full, medium, low)

### Key Game Systems

#### 1. **Game State**
- Player object: position, dimensions, weapon, level, invulnerability timer
- Arrays: `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- Scoring: `score`, `multiplier`, `chain` (kill combo system)
- Progression: `stage`, `kills`, `stageKills` (18 kills → boss spawn)

#### 2. **Input Handling** (lines 767-862)
- **Keyboard**: Arrow keys or WASD for movement, Space to fire, X/Shift for bombs, P/Esc to pause
- **Mouse**: Move to steer, click to fire (mouse target glides smoothly toward pointer)
- **Touch/Mobile**: Left side joystick for movement, right side buttons for fire/bomb
- Priority: keyboard/joystick > mouse (prevent fighting)

#### 3. **Combat System**
- **Player weapons**:
  - Vulcan: spread pattern increases with level (1→2→3)
  - Laser: pierce through enemies, multi-lane fire at level 3
  - Switch via weapon power-ups
- **Enemy spawning**: Difficulty ramps over time; 3 types (grunt, interceptor, tank)
- **Boss system**: 6 boss designs with unique attack patterns (e.g., radial bursts, homing missiles, spirals)
- **Collisions**: `rectsOverlap()` checks (lines 1069-1070)

#### 4. **Difficulty & Progression**
- Three difficulties: Easy, Normal, Hard (stored in localStorage)
- Parameters: enemy spawn rate, speed, shoot chance, boss HP multiplier, starting lives/bombs
- Auto-scaling quality to maintain 60 FPS even on low-end devices

#### 5. **Audio System** (lines 542-676)
- WebAudio API synthesis (no external audio files)
- `beep()`: Frequency-based square/sawtooth/sine tones with envelope
- `noiseBurst()`: Procedural noise bursts with filter
- 13 SFX types with spatial audio (panning based on x position)
- Muted state persisted to localStorage

#### 6. **Visual Effects**
- **Themes**: 6 color themes with sky gradients + sun + clouds (line 493)
- **Screen shake**: `screenShake(magnitude, duration)` for impact feedback
- **Flash effects**: Hit flash and bomb flash animations
- **Particles**: Explosion sparks with physics
- **Animations**: CSS keyframes for combo pop, stage banner, UI card entrance

#### 7. **Accessibility**
- Screen reader announcements via `srLive` element
- Reduced motion support (respects `prefers-reduced-motion` media query)
- Color contrast and text sizing
- Keyboard-only playable

### Data Constants

- **DIFFICULTIES** (line 485): Easy/Normal/Hard parameter tables
- **ENEMY_TYPES** (line 502): Grunt, Interceptor, Tank definitions (size, HP, speed, score)
- **BOSSES** (line 508): 6 boss designs with visual colors and HP multipliers
- **BOSS_PATTERNS** (line 1372): Attack pattern functions for each boss type
- **THEMES** (line 493): 6 theme color palettes

## Code Navigation Guide

| Concern | Location |
|---------|----------|
| Player rendering | `drawPlane()` (line 913) |
| Boss rendering | `drawBossShip()` (line 970) |
| Enemy spawning | `spawnEnemy()` (line 1022) |
| Boss spawning | `spawnBoss()` (line 1043) |
| Player-bullet collisions | `update()` lines 1232-1253, 1255-1266 |
| Enemy-bullet collisions | `update()` lines 1268-1287 |
| Power-up collection | `update()` lines 1289-1294 |
| Boss attack patterns | `updateBoss()` (line 1411), `BOSS_PATTERNS` (line 1372) |
| Canvas sizing | `resize()` (line 754) |
| Main render loop | `draw()` (line 1520) |
| UI updates | `renderHearts()` (line 680), `renderBombs()` (line 690) |

## Performance Optimization

- **Quality scaling** (lines 474-483): Monitors frame time over 45 frames; adjusts particle count, glow effects, and other features
- **Object pooling**: Reuses enemy bullets, particles rather than creating/destroying
- **Canvas optimization**: 480×720 internal resolution scaled to fit window (capped at 1.6×)
- **RequestAnimationFrame**: Syncs with display refresh rate

## Common Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Add explosion SFX mapping in `EXPLODE_SFX` (line 678) if needed
3. Update `spawnEnemy()` probability checks (line 1028)

### Adding a New Boss Pattern
1. Create a pattern function (e.g., `fireRadial()`, `fireAimedSpread()`) that calls `enemyBullets.push()`
2. Add to `BOSS_PATTERNS` array at the correct index (aligned with `BOSSES` order)
3. Boss patterns execute during attack phase (`updateBoss()` line 1427)

### Changing Game Balance
- **Difficulty**: Edit `DIFFICULTIES` object (line 485)
- **Stage progression**: `KILLS_PER_STAGE` (line 491)
- **Spawning**: `spawnTimer` ramp in `update()` (line 1201)
- **Scoring**: Adjust base scores in `ENEMY_TYPES` and `addScore()` multiplier formula (line 1090)

### Adding a New Theme
1. Add color palette to `THEMES` array (line 493): `{ sky: [top, middle, bottom], cloud: rgb, sun: rgb, name }`
2. Automatically cycles every 6 stages via `(stage - 1) % THEMES.length`

### Debugging in Browser
- Open DevTools (F12), go to Console
- Game state variables are in closure; use breakpoints on game logic functions
- Check localStorage keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

## Browser Support

- **Requires**: Modern browser with Canvas 2D, WebAudio API, localStorage
- **Tested**: Chrome, Firefox, Safari, Edge (latest versions)
- **Mobile**: iOS Safari, Chrome Android
- **Audio context**: May require user interaction to initialize (click/touch)

## Storage & Persistence

Game state persists via `localStorage`:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio mute state (0/1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion reduction toggle (0/1)

Clear storage in DevTools if needed: `localStorage.clear()`

## Notes for Future Development

- The IIFE closure keeps all game logic scoped; avoid polluting global scope
- Canvas context (`ctx`) is obtained once at startup; reused for all draws
- All physics use delta-time (`dt`) for frame-rate independence
- Boss patterns are data-driven; new patterns can be added without code changes
- Audio is entirely procedural (no asset files); easier to modify/add SFX
- Reduced motion mode gracefully degrades animations via CSS without duplicating logic
