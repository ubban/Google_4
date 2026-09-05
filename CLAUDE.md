# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely as a single HTML5 canvas application. The game features:
- Multiple difficulty levels (Easy, Normal, Hard)
- Progressive stages with escalating enemy types
- Boss battles with distinct attack patterns
- Weapon upgrade system (Vulcan vs Laser with 3 levels each)
- Power-ups (lives, bombs, weapons)
- Combo scoring multiplier system
- Procedurally themed stages (6 visual themes)
- Full keyboard, mouse, and touch support
- Synthesized audio (no external audio files)
- Accessibility features (screen reader support, reduced motion mode)

## Codebase Structure

The entire game is contained in a single `index.html` file (~1670 lines) with three sections:

### 1. HTML Structure (lines 361-420)
- Game canvas container
- HUD elements (score, best, weapon, lives, bombs)
- Boss health bar
- UI buttons (mute, motion toggle, pause)
- Overlay system for title screen and game-over screen
- Touch controls (virtual joystick, fire/bomb buttons)
- Screen-reader announcements container

### 2. CSS Styling (lines 10-358)
- Canvas and container layout (fullscreen, centered)
- HUD stats styling with glassmorphism effect
- Animations: heartBeat, vignettePulse, flashHit, flashBomb, comboPulse, stagePulse, cardIn, iconBob
- Responsive sizing (clamp units for mobile-to-desktop scaling)
- Theme-aware colors (light/dark modes via prefers-color-scheme)
- Reduced motion media queries for accessibility

### 3. JavaScript Game Logic (lines 422-1666)

**Key State Objects:**
- `player`: Position, speed, weapon, level, invulnerability timer
- `enemies`, `bullets`, `enemyBullets`: Collision entities
- `boss`: Boss object with health, attack patterns, telegraph timing
- `particles`: Visual effects from explosions
- `powerUps`: Weapon/life/bomb pickups

**Core Game Loop** (line 1640-1652):
- `requestAnimationFrame(loop)` drives the game at 60fps
- Delta-time capped at 50ms to prevent huge jumps on lag
- Calls `update(dt)` for game logic
- Calls `draw()` for canvas rendering

**Major Subsystems:**

1. **Performance Adaptation** (lines 473-483)
   - Tracks frame times over 45 frames
   - Adjusts `quality` level (0=full/glow effects, 1=medium, 2=low)
   - Disables shadow blur and particle counts at lower quality

2. **Audio** (lines 542-676)
   - WebAudio synthesizer (no external asset dependencies)
   - `beep()` for tonal sounds with frequency sweep, filter, panning
   - `noiseBurst()` for percussive/explosion sounds
   - `sfx` object maps 13+ sound effects to synth functions
   - Master gain node + dynamics compressor for audio balance

3. **Input Handling** (lines 767-861)
   - Keyboard: Arrow keys / WASD for movement, Space for fire, X/Shift for bomb, P/Esc for pause
   - Mouse: Hover to steer, click to fire, movement indicator cursor
   - Touch: Virtual joystick in left zone, fire/bomb buttons in right zone
   - Priority: keyboard/joystick > mouse steering (prevents conflicts)

4. **Entity System**
   - **Player** (lines 1148-1172): Clamp to canvas, accumulate velocity from all input sources
   - **Enemies** (lines 1022-1041, 1210-1228): Types are grunt/interceptor/tank with different speeds, HP, scores, fire patterns
   - **Bosses** (lines 1043-1056, 1411-1439): 6 named bosses with distinct attack pattern sets
   - **Bullets** (lines 1124-1146): Player bullets pierce/laser variants; enemy bullets are radial/aimed/spread patterns
   - **Collision** (line 1069-1071): Rectangle overlap test

5. **Difficulty Tuning** (lines 485-489)
   - 3 presets control spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
   - Spawn timer decreases as frame count increases (dynamic difficulty ramp at line 1201)

6. **Boss Attack Patterns** (lines 1372-1409)
   - 6 boss types, each with 3 unique attack patterns
   - Functions like `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, `fireSpiralBurst()`, `fireCross()`, `fireHomingSeeded()`, `fireDoubleRing()`
   - Pattern sequence cycles through as boss attacks

7. **Visual Theming** (lines 493-500)
   - 6 themes with distinct sky gradients, cloud, and sun colors
   - Theme selection based on stage number (wraps around)
   - Sky rendering: gradient background + radial glow + layered cloud sprites

8. **Score & Multiplier** (lines 1087-1101, 1087-1093)
   - Chain counter increments on kill, multiplier = 1 + floor(chain/5) (up to 10x)
   - Chain timer resets every 2.2 seconds without kills
   - Breaking chain resets multiplier to 1x

9. **Screen Effects** (lines 726-730, 698-709)
   - Screen shake: magnitude + duration, applied as translation in draw context
   - Flash overlays: white flash on hit (0.25s), yellow flash on bomb (0.5s)
   - Vignette pulse: red vignette when health = 1 life

10. **UI/Overlay System** (lines 863-1505)
    - Overlay div covers canvas for title screen, pause, game-over
    - Difficulty buttons select presets (persisted in localStorage)
    - Stats display with animations and sound effects
    - Pause state: game logic halts, overlay shows "Paused"

## Common Development Tasks

### Running & Testing
- **No build step required:** Simply open `index.html` in a modern browser (Chrome, Firefox, Safari, Edge)
- **Testing mobile:** Use browser DevTools device emulation or test on actual device via local network
- **Testing touch:** DevTools has a touch emulation mode; physical device recommended for accurate feel

### Modifying Game Balance
- Adjust `DIFFICULTIES` object (line 485-489) for spawn rates, enemy behavior, player lives
- Modify `KILLS_PER_STAGE` (line 491) to change how many enemy kills trigger boss spawn
- Enemy type definitions in `ENEMY_TYPES` (line 502-506): w/h, hp, speed multiplier, colors, score value
- Boss parameters in `BOSSES` (line 508-515): health multiplier, color scheme
- Weapon fire rates: `fireDelay` variable at line 1177

### Adding New Enemy Types
1. Add entry to `ENEMY_TYPES` with shape, HP, speed, color, minStage
2. Add corresponding spawn chance in `spawnEnemy()` (line 1022-1041)
3. Add explosion SFX mapping in `EXPLODE_SFX` (line 678) if desired
4. Weapon collision logic already handles any ENEMY_TYPES entry

### Adding New Boss Attack Patterns
1. Define pattern functions using existing helpers (`fireRadial`, `fireAimedSpread`, etc.) or write custom
2. Add them to `BOSS_PATTERNS` array in the appropriate sub-array index (one per boss type)
3. Pattern function receives boss object `b` and should push entries to `enemyBullets`

### Changing Visual Themes
- Edit `THEMES` array (line 493-500): sky gradient colors, cloud color, sun color, display name
- Colors are RGB strings (e.g., '255,255,220') used in rgba() constructors for flexibility
- Sky gradients use 3 stops; modify `drawClouds()` calls (line 1543-1544) to add/change layers

### Adjusting Canvas Size
- Base dimensions hardcoded in `BASE_W`, `BASE_H` (line 753): currently 480x720 (mobile portrait)
- `resize()` function (line 754-764) scales to fit window while maintaining aspect ratio
- All positioning is relative to canvas dimensions, so changes should scale naturally

### Adding Sound Effects
- Use `beep()` for tonal sounds: freq, duration, wave type, volume, options object
  - Options: `sweepTo` (frequency sweep), `filterType`/`filterFreq`, `pan`, `delay`
- Use `noiseBurst()` for noise-based sounds: duration, volume, options object
- Add to `sfx` object (line 624-676) and call via `sfx.effectName()`

### Accessibility Considerations
- `prefers-reduced-motion` is detected on init (line 520); reduce animation intensity when active
- Screen reader announcements via `announce()` function (line 468-471) update `#srLive` element
- Color contrast meets WCAG standards for HUD text
- Mobile touch controls available alongside keyboard

### Persisted Player Data
- Uses `localStorage` with keys:
  - `skyfighter.bestScore`: high score
  - `skyfighter.muted`: mute state (0/1)
  - `skyfighter.difficulty`: last selected difficulty
  - `skyfighter.reducedMotion`: motion preference (0/1)

## Architecture Notes

### Single-File Constraint
Everything runs in one HTML file to ensure portability (no build, no dependencies). This means:
- Global state is confined to an IIFE (`(function () { 'use strict'; ... })()`) to avoid namespace pollution
- No module system; code is organized top-to-bottom by logical section
- Canvas rendering is immediate-mode (redraw every frame)—no scene graph

### Collision & Physics
- Collision detection: simple rectangle overlap (AABB)
- Movement: position += velocity * deltaTime (no gravity or forces)
- Player movement priority: keyboard/joystick > mouse steering
- Enemy spawning: random X, Y off-screen top, descend at varying speeds

### Frame Timing & DT
- Game updates with delta time (`dt`); all speeds are in units/second
- Frame count (`frame` variable) used for animation and pattern cycling (not tied to real time)
- Performance adaptation adjusts quality based on average frame time

### Rendering Order
1. Background gradient + sun glow + clouds (parallax via frame-based offset)
2. Player ship + weapon glow
3. Player bullets (with glow if quality=0)
4. Enemy bullets (circles with glow if quality=0)
5. Enemies with health bars (tanks only)
6. Boss ship (if present)
7. Power-ups (rotating squares)
8. Particles (explosions)
9. Screen shake applied to entire canvas transform

## Debugging Tips
- Open DevTools Console: check for JavaScript errors or `console.log()` statements added to code
- Performance: DevTools Performance tab to check frame rate; look for `quality` variable changes
- Canvas inspect: draw bounding boxes around entities for collision debugging (add to `draw()`)
- Audio: check `audioCtx.state` in Console if audio isn't working; may need user gesture to enable
- LocalStorage: inspect with DevTools Application tab; clear via `localStorage.clear()`
