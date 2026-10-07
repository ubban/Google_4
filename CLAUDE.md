# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical bullet-hell shoot-'em-up game built entirely in a single HTML file. It features:
- Procedurally-drawn graphics using Canvas 2D (no image assets)
- Synthesized audio using Web Audio API (no audio files)
- Touch and keyboard controls
- Progressive difficulty system (Easy/Normal/Hard)
- Boss encounters with unique attack patterns
- Weapon upgrade system (Vulcan vs Laser)
- Score multipliers and combo system
- Responsive design supporting mobile and desktop

**No build tools, no dependencies, no bundler.** The entire game runs from `index.html` in any modern browser.

## Running the Game

Simply open `index.html` in a web browser:
- **Desktop**: Double-click `index.html` or open from browser
- **Local dev**: `python3 -m http.server 8000` then visit `http://localhost:8000`
- **Testing**: Open in browser DevTools console to check for errors

## Code Architecture

The single `index.html` file is organized as follows (all within a `<script>` IIFE):

### Game Configuration (lines 463–515)
- `STORAGE_*` constants for localStorage keys (best score, mute state, difficulty, reduced motion)
- `DIFFICULTIES` object defining Easy/Normal/Hard parameters (spawn rates, enemy speed, player lives)
- `THEMES` array defining 6 background gradients (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- `ENEMY_TYPES` object defining grunt/interceptor/tank properties (size, HP, speed, score)
- `BOSSES` array defining 6 boss designs with unique color schemes and attack pattern indices

### Core Systems

**Audio (lines 542–676):**
- `ensureAudio()` initializes Web Audio Context
- `beep()` creates pitched oscillator tones with frequency sweeps, filters, panning
- `noiseBurst()` creates noise-based SFX using AudioBuffer
- `sfx` object with all sound effects (shoot, laser, explode, hit, bomb, powerup, boss)
- Audio respects mute state and reduced-motion preferences

**Input Handling (lines 767–861):**
- Keyboard: Arrow keys, WASD for movement; Space/Click for fire; X/Shift/P for bomb/pause
- Mouse: Move pointer to steer; click to fire; leaves canvas to lose input focus
- Touch: Left zone for joystick movement; right buttons for FIRE/BOMB
- `mouseActive`, `joyActive`, `firing` flags track concurrent input modes
- Keyboard takes priority over mouse to prevent fighting inputs

**Canvas & Rendering (lines 752–1638):**
- Base resolution: 480×720 (portrait), scales to fit window with 1.6× max zoom
- `drawPlane()` renders player/enemy ships as geometric shapes with wings, hull, cockpit
- `drawBossShip()` draws boss with gradient hull, animated core glow, rotating engines
- `draw()` renders background gradient+sun+clouds, then all entities, then particles
- `drawClouds()` creates parallax background elements

**Game State & Loops (lines 863–1302):**
- `update(dt)` handles all physics, collision, spawning each frame
- Player movement clamped to canvas; collision detection with `rectsOverlap()`
- Enemy spawning with difficulty-based intervals; weaving patterns for interceptors
- Bullet tracking with pierce mechanic for lasers; enemy bullets use radial movement
- Power-ups spawn randomly from defeated enemies
- Boss updates handled separately in `updateBoss(dt)` with telegraph-then-attack pattern
- Score chain system: consecutive kills multiply score; timer resets on pause

**Boss AI (lines 1372–1434):**
- Each boss has unique attack pattern sequence (3 patterns per boss design)
- Patterns include: radial bursts, aimed spreads, homing missiles, spirals, crosses, columns
- Boss enters from top with 120px stop point; bounces left-right at edges
- Telegraph phase (22 frames) warns before attack; allows dodge planning

## Key Game Mechanics

### Difficulty Levels
Each difficulty scales these values:
- `spawnBase` / `spawnMin`: How often enemies spawn (frame-based)
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: Probability enemy can shoot at player
- `lives` / `bombs`: Starting resources
- `bossHpMul`: Boss health multiplier

### Weapons
- **Vulcan** (default): Wide spread at level 3; rapid fire but lower damage
- **Laser**: Piercing projectiles; fires faster; narrow at low levels, triple beam at level 3
- Toggle via weapon power-ups; caps at level 3 before switching type

### Combo/Chain System
- Kill counter increments on each enemy kill
- Each kill extends timer (2.2 seconds)
- Multiplier = 1 + floor(chain / 5), capping at 10×
- Timer resets on player hit or damage

### Adaptive Quality
`quality` flag (0=full, 1=medium, 2=low) adjusts based on frame time:
- Particle counts scale with quality
- Glow effects disabled on lower qualities
- Reduces burden on slower devices

## Performance Considerations

- **Particle culling**: Particles removed when lifespan ≤ 0; counts scale with quality
- **Enemy culling**: Removed when off-bottom of screen
- **Bullet culling**: Removed when off-canvas
- **RequestAnimationFrame**: 60 FPS cap; delta-time clamped to 50ms max per frame to prevent spiral
- **Canvas 2D rendering only**: No WebGL; trade-off is simplicity over raw performance

## Testing & Debugging

**In-browser DevTools:**
- Check console for errors (audio context fallback, canvas size)
- `localStorage.clear()` resets scores/settings to defaults
- Inspect `window.audioCtx.state` to verify audio initialization
- Watch `frame` counter in update() for frame-counting bugs

**Common Test Cases:**
- Easy mode: Slower spawns, fewer lives lost to careless play
- Boss patterns: Each boss has 3 unique patterns; test all 6 bosses across stages
- Weapon switching: Grab 6+ weapon pickups to toggle from Vulcan ↔ Laser multiple times
- Bomb usage: 2–4 bombs per stage; verify damage and explosion count
- Reduced motion: Toggle to disable shake/flash/pulse animations
- Mobile: Test joystick in left zone, fire/bomb buttons in right zone; verify touch doesn't scroll page

**Known Limitations:**
- Audio context requires user gesture (click/touch) to initialize
- No pixel-perfect collision (rect-based only)
- No frame-skip; laggy devices drop framerate rather than speed-run
- Boss patterns hardcoded (not data-driven); add new bosses by extending arrays

## Modifying the Game

### Adding a New Stage Theme
1. Push new object to `THEMES` array (line 493): `{ sky: [...], cloud: '...', sun: '...', name: '...' }`
2. Stage loops themes by index: `THEMES[(stage - 1) % THEMES.length]`

### Adding a New Boss
1. Push to `BOSSES` array (line 508) with `hullTop/hullBottom/wing/core` colors and `hpMul`
2. Push 3 attack pattern functions to `BOSS_PATTERNS` array (line 1372)
3. Cycle happens automatically; no other wiring needed

### Changing Enemy Types
- Edit `ENEMY_TYPES` object (line 502): `{ w, h, hp, speedMul, color, dark, score, minStage }`
- `minStage` controls when type first appears
- `weave` flag enables horizontal oscillation (for interceptors)

### Tweaking Difficulty
- Edit `DIFFICULTIES` object (line 485) for spawn rates, enemy stats, or lives
- Changes apply on next game start (live difficulty changes not supported)

### Audio Tuning
- Adjust `masterGain.gain.value` (line 556) for overall volume
- Edit compressor settings (line 549) for loudness/dynamics
- Modify individual `sfx.*` functions to change pitch/duration/filters

## State Persistence

Game data persists via localStorage:
- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Mute toggle (0 or 1)
- `skyfighter.difficulty`: Selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: Motion reduction (0 or 1, defaults to system prefers-reduced-motion)

Wipe data with DevTools console: `localStorage.clear()`

## Accessibility

- **Screen Reader Support**: All UI state announcements sent to `#srLive` aria-live region
- **Reduced Motion**: Detects `prefers-reduced-motion` media query; can override via button
- **Color Contrast**: All UI text meets WCAG AA standards against backgrounds
- **Touch Controls**: Dual-zone layout for mobile (55% left for movement, 45% right for fire/bomb)
- **Keyboard**: Full keyboard support; no mouse required to play
