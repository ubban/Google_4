# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game, entirely self-contained in a single `index.html` file (~1670 lines). It features:
- Responsive canvas-based rendering with adaptive quality scaling
- Multiple difficulty levels and visual themes
- Synthesized sound effects (Web Audio API, no external assets)
- Full touch, keyboard, and mouse control support
- 6 unique boss encounters with distinct attack patterns
- Weapon upgrade system and power-ups
- Persistent score tracking and settings via localStorage

## Architecture & Code Organization

The game uses a single-file architecture with clear logical sections:

### Layout & Structure (lines 1-420)
- **HTML structure**: Canvas element, HUD elements (score, lives, bombs, boss bar), overlay screen, touch controls
- **CSS styling**: Responsive flexbox layout with `clamp()` for scaling, animations, reduced-motion support, glassmorphism effects
- **Key DOM elements**: Canvas (#gameCanvas), HUD display (#hud, #score, #hearts, etc.), overlay (#overlay), controls (#topBar buttons)

### Audio System (lines 542-676)
- **Web Audio synth-based**: No external sound files, all SFX are synthesized in real-time using oscillators, filters, noise buffers
- **Functions**:
  - `ensureAudio()`: Lazy-initialize AudioContext (resolves browser autoplay restrictions)
  - `beep()`: Synthesized tone with optional frequency sweep, filter, and panning
  - `noiseBurst()`: White noise with frequency filtering
  - `sfx` object: Pre-configured sound effects (shoot, laser, explosions by type, power-ups, boss sounds, etc.)
- **Master gain & compression**: Applied to all audio for consistent volume
- **Mute state**: Persisted via localStorage, respects user preference

### Game State & Settings (lines 467-527)
- **Difficulty levels**: DIFFICULTIES object defines spawn rates, enemy speed, shoot chance, lives, bombs, boss HP multiplier
- **Constants**: ENEMY_TYPES (grunt, interceptor, tank), BOSSES (6 designs with visual/pattern variations), THEMES (6 background themes)
- **Persistent storage**: Best score, mute state, difficulty, reduced motion preference all saved in localStorage
- **Input state**: `keys` object tracks keyboard, `joyVec` for touch joystick, `mouseTarget` for mouse steering
- **Game variables**: player, bullets, enemies, enemyBullets, particles, powerUps, boss (all managed during gameplay)

### Rendering System (lines 913-1020, 1507-1638)
- **Canvas setup**: 480x720 base (scaled to fit window with max 1.6x scale)
- **Drawing functions**:
  - `drawPlane(x, y, w, h, color, darkColor, flame)`: Generic fighter plane sprite (used for player and enemies)
  - `drawBossShip(b)`: Boss-specific rendering with design colors, core glow, wing rotation
  - `drawClouds()`: Parallax cloud layers for visual depth
  - `draw()`: Main render loop; applies screen shake, draws background gradient + sun/clouds, all game objects
- **Visual effects**:
  - Screen shake with magnitude/duration
  - Flash animations (hit/bomb)
  - Vignette pulse when critical (1 life)
  - Particle explosions (color-coded by source)
  - Combo pop-ups and stage banners
  - Glow effects (shadow/blur) on high quality, disabled on lower quality
- **Adaptive quality**: `quality` variable (0=full, 1=medium, 2=low) disables glow, reduces particles based on frame time tracking

### Input Handling (lines 768-861)
- **Keyboard**: Arrow keys / WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- **Touch**: Left zone for movement (joystick), right zone for FIRE/BOMB buttons
- **Mouse**: Movement steers player toward pointer, click to fire (if canvas is hovered)
- **Priority**: Keyboard/joystick input takes priority over mouse for movement (prevents conflicts)
- **Audio resume**: Input events trigger `ensureAudio()` to work around browser autoplay restrictions

### Game Loop & Physics (lines 1148-1302)
- **`update(dt)`**: Called at ~60fps (clamped to max 50ms delta)
  - Player movement (keyboard/joystick/mouse priority)
  - Weapon cooldown and firing (different rates for vulcan vs laser)
  - Enemy spawning with difficulty-based spawn timer that ramps over time
  - Enemy behavior (movement with weaving pattern for some types, shooting aimed/spread patterns)
  - Bullet/enemy/power-up movement
  - Collision detection (enemy-bullet, enemy-player, boss-bullet, power-up pickup)
  - Chain/multiplier decay
  - Boss state updates
- **Collision**: Simple AABB (`rectsOverlap()`) checks
- **Particles**: Position/velocity update with fade-out over lifetime

### Boss System (lines 1043-1454)
- **Boss lifecycle**: Entering (slide in from top) → active (patrol + attack) → defeated
- **6 boss designs** (lines 508-515): VANGUARD, BEHEMOTH, SPECTER, DREADNOUGHT, WRAITH, TITAN
  - Each has unique hull/core colors and HP multiplier
  - Each mapped to 6 unique attack patterns (lines 1372-1409)
- **Attack patterns**: Radial bursts, aimed spreads, teleport + attack, homing missiles, double rings, etc.
- **Telegraph phase**: Boss glows/pulses before firing; allows player to react
- **Bullet types**: Radial bullets (move via dx/dy), aimed bullets (aimed at player), homing (seeded with spread)
- **Pattern cycling**: Boss cycles through 3 attack patterns, each triggered after telegraph + attack timer

### Weapon & Power-Up System (lines 1124-1322)
- **Weapons**: VULCAN (spread bullets, fast fire) vs LASER (fewer bullets, piercing, slower fire)
- **Levels**: Weapon level 1-3 increases spread/lanes
- **Power-ups**: Life, Bomb, Weapon (upgrade level or swap weapon)
- **Power-up spawn**: 6% life, 8% bomb, 20% weapon on enemy kill

### UI & Overlay (lines 401-419, 1469-1504)
- **Main menu**: Shows start button, difficulty selection, controls hint
- **Game over overlay**: Displays final score, stage reached, new best score message
- **Pause overlay**: Shows current game state with resume button
- **HUD**: Real-time display of score, best score, multiplier, current weapon, lives, bombs, boss health bar
- **Accessibility**: Screen reader announcements via `announce()` function, `prefers-reduced-motion` support

### Difficulty & Stage Progression
- **Difficulty settings**: Easy (fewer enemies, slow, more lives), Normal (balanced), Hard (many enemies, fast, fewer bombs)
- **Stages**: Kill 18 enemies to trigger boss → defeat boss to advance stage
- **Difficulty scaling**: Spawn timer decreases over time (frame-based), enemy types unlock at certain stages
- **Themes**: Visual theme rotates per stage (6 themes cycle)

## Development Notes

### Monolithic Design
The entire game is in one file by design for simplicity and self-containment. To modify:
1. Locate the relevant section (use the line numbers above)
2. Make changes carefully to preserve the IIFE structure
3. Test locally by opening index.html in a browser

### Common Modifications

**Adding a new enemy type:**
1. Add entry to `ENEMY_TYPES` object (line ~502)
2. Add to enemy selection logic in `spawnEnemy()` (line ~1026)
3. Create explosion sound effect in `sfx` object if needed, map in `EXPLODE_SFX`

**Adding a new boss:**
1. Add entry to `BOSSES` array (line ~508)
2. Create a 3-pattern array in `BOSS_PATTERNS` (line ~1372)
3. Set unique colors (hullTop, hullBottom, wing, core) and HP multiplier

**Tweaking difficulty:**
- Modify `DIFFICULTIES` object values (spawn timings, speeds, shoot chances, lives/bombs)
- Adjust `KILLS_PER_STAGE` constant (line 491) to change progression speed
- Modify base HP calculation in `spawnBoss()` (line 1046)

**Adding visual themes:**
1. Add entry to `THEMES` array (line ~493) with sky gradient stops, cloud/sun colors, and name
2. Themes auto-cycle per stage

**Changing game balance:**
- Enemy bullet speeds, spread angles in `fireRadial()`, `fireAimedSpread()`, etc.
- Weapon cooldown rates in `fireWeapon()` (line ~1124)
- Bullet speeds and pierce values
- Player movement speed (line ~877)

### Rendering & Performance
- **Canvas sizing**: 480x720 base, scales to fit with max 1.6x (responsive)
- **Adaptive quality**: Disabled glow/shadows and particle count on lower-end devices (tracks frame times)
- **Screen shake**: Reduced or disabled if `reducedMotion` is true
- **Animations**: CSS animations for UI; canvas-based for game objects

### Audio
- All sound is synthesized; no external audio files needed
- Frequency, waveform type, filters, and panning can be tweaked in `sfx` object
- `ensureAudio()` handles lazy initialization and browser autoplay restrictions
- Mute button persists state; respects audio context suspension

### Testing & Running
- Open `index.html` in any modern browser (desktop or mobile)
- No build process, no external dependencies
- Web Audio and touch APIs are feature-detected (game still runs on older devices, without sound)
- localStorage persists across sessions; test with browser dev tools storage tab

### Accessibility
- Keyboard controls fully supported (no mouse-only paths)
- Touch controls for mobile
- Reduced motion support (animations disabled if user prefers)
- Screen reader announcements for game events
- Color contrasts meet WCAG standards
