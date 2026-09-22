# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-em-up game implemented as a single-file HTML5 application. The game features responsive canvas-based rendering, WebAudio synthesis (no external audio files), touch and keyboard controls, progressive difficulty, and procedural boss encounters.

## Quick Start

- **Open**: Simply open `index.html` in a browser
- **No build step required**: This is a standalone HTML file with embedded CSS and JavaScript
- **Test in browser**: Works on desktop (keyboard/mouse) and mobile (touch controls)

## Game Architecture

The entire game is contained in `index.html` (~1,669 lines) with three sections:

### 1. **HTML Structure (Lines 1-421)**
- `#gameCanvas`: Main rendering surface (2D canvas context)
- `#overlay`: Modal UI for title screen, game-over, pause states
- `#hud`: Real-time HUD (score, lives, bombs, weapon info)
- `#touchControls`: Mobile joystick and fire/bomb buttons
- Various UI elements: boss health bar, combo popups, stage banners, vignette/flash effects

### 2. **CSS Styling (Lines 10-420)**
- Responsive layout using `clamp()` for fluid scaling across screen sizes
- Dark theme with gradient backgrounds and glassmorphism UI (backdrop-filter, blur)
- CSS animations: heartbeat (critical health), combo pulse, stage banner, card-in, screen flash
- Touch-action and cursor styling optimized for game interaction
- Safe-area insets for notched devices

### 3. **JavaScript Game Logic (Lines 422-1,669)**
The game runs in a single IIFE (Immediately Invoked Function Expression) with these key subsystems:

#### **Core Game State**
- `bestScore`, `muted`, `difficulty`, `reducedMotion` — stored in localStorage
- Game loop uses `requestAnimationFrame` for 60 FPS target with adaptive performance tracking
- Difficulty modes (easy/normal/hard) scale: spawn rates, enemy speed, boss HP, player lives

#### **Player & Controls**
- `player` object: position, velocity, health, bombs, weapon level, score multiplier
- Keyboard input: arrow keys/WASD for movement, Space/Z for fire, B for bombs
- Touch input: Joystick (virtual D-pad) in left half of screen, Fire/Bomb buttons right half
- Weapon system: 4 weapon levels (spread increases, fire rate improves)

#### **Enemy & Boss Systems**
- Enemy types defined in `ENEMY_TYPES` object: grunt, interceptor (fast, weaves), tank (heavy)
- Each enemy tracks: position, velocity, health, weapon type, AI shooting chance
- Boss encounters (6 unique bosses) have distinct visual designs and health multipliers
- Stage progression: 18 kills triggers next stage, bosses spawn at stage transitions

#### **Visual & Audio Systems**
- **WebAudio Synthesis**: All sound generated via oscillators, no external audio files
  - `beep()`: Tonal sounds (player fire, hits) with optional pitch sweep and filter
  - `noiseBurst()`: Noise-based effects (explosions, bomb detonation)
  - Pan stereo based on screen position (left/right channels)
  - Master gain with dynamics compressor for balanced output
- **Canvas Rendering**: Draws plane sprites, enemies, projectiles, explosions via `drawPlane()` and `drawBossShip()`
- **Screen Effects**: Vignette pulse on critical health, white flash on hit/bomb, combo text popup

#### **Game Loop Flow**
1. Input processing (keyboard, touch, gamepad)
2. Update: player movement/fire, enemy AI, collision detection, projectile physics
3. Spawn logic: enemies every N frames (scale-dependent), bosses at stage transitions
4. Rendering: clear canvas, draw sky gradient (theme-based), render all entities, HUD
5. Performance tracking: monitor frame times, scale down quality if avg FPS drops below 40

## Key Configurations

### **Difficulties** (Line 485-489)
```javascript
DIFFICULTIES = {
  easy:   { spawnBase: 75, spawnMin: 40, enemySpeed: 1.0, ... lives: 4, bombs: 3, ... },
  normal: { spawnBase: 58, spawnMin: 26, enemySpeed: 1.4, ... lives: 3, bombs: 2, ... },
  hard:   { spawnBase: 40, spawnMin: 18, enemySpeed: 1.9, ... lives: 3, bombs: 2, ... }
}
```
Adjusts spawn timing, enemy behavior, and boss HP.

### **Themes** (Line 493-500)
Six visual themes (Daylight Front, Sunset, Night, Storm, Arctic, Volcanic) define sky gradient, cloud color, and sun color for variety across playthroughs.

### **Enemy Types** (Line 502-506)
Each type has: dimensions (w/h), HP, speed multiplier, color scheme, score, minimum stage to appear, and special AI (weave for interceptor).

### **Bosses** (Line 508-515)
Six unique boss designs with color pairs (hull, wing, core) and HP multipliers for scaling difficulty.

## Common Development Tasks

### **Adjust Game Balance**
- **Enemy spawn rate**: Modify `spawnBase` and `spawnMin` in DIFFICULTIES
- **Player damage output**: Change weapon fire frequency or velocity in player fire-rate calc
- **Boss difficulty**: Adjust `hpMul` values in BOSSES array
- **Lives/bombs**: Update DIFFICULTIES values

### **Add a New Enemy Type**
1. Define in `ENEMY_TYPES` object with w, h, hp, speedMul, color, score, minStage
2. Update spawn probability in `spawnEnemy()` (line ~1022) to include chance for new type
3. Optionally add custom AI logic in enemy update loop

### **Add a New Boss**
1. Add entry to `BOSSES` array with name, hull colors, wing, core, hpMul
2. Boss rendering uses `drawBossShip()` (line ~970) — colors are interpolated from hull/wing/core
3. Bosses spawn automatically on stage transitions; health scales with `hpMul`

### **Modify Audio**
- Tone effects: Adjust `beep()` frequency, duration, oscillator type (sine/square/sawtooth)
- Filter sounds: Add `sweepTo` (pitch bend) or `filterType`/`filterFreq` in opts
- Noise effects: Change noise volume or duration in `noiseBurst()`
- Mute state: Checked in `beep()` and `noiseBurst()` before playback

### **Adjust Canvas Rendering**
- Sky gradient: Modify theme colors in `THEMES[].sky` array
- Plane/boss visuals: Edit `drawPlane()` and `drawBossShip()` drawing code (ctx.fillStyle, ctx.fillRect, etc.)
- HUD layout: CSS in the style block (clamp sizes, colors, positioning)
- Screen effects: Vignette color/intensity in CSS `#vignette` or `flashHit()`/`flashBomb()` opacity

## Data Persistence

Three localStorage keys (Line 463-466):
- `skyfighter.bestScore`: High score (numeric)
- `skyfighter.muted`: Audio mute state ('0' or '1')
- `skyfighter.difficulty`: Selected difficulty name (easy/normal/hard)
- `skyfighter.reducedMotion`: Reduced motion preference ('0' or '1')

All loaded on startup, updated on change (mute toggle, difficulty select, game-over).

## Browser & Performance Notes

- **Canvas size**: Responsive, maintains aspect ratio, uses `window.devicePixelRatio` for sharpness
- **Performance tracking**: Frame times averaged over 45 frames; quality level (0=full, 1=medium, 2=low) reduced if FPS drops below 40
- **Touch support**: Separate joystick and button zones on mobile; pointer events handled for both touch and mouse
- **Screen reader**: HUD changes announced via `#srLive` element for accessibility
- **Storage fallback**: If localStorage unavailable, game still works (no persistence)
- **Audio context**: Lazily initialized on first user input to comply with browser autoplay policies

## File Editing Tips

- **Search for function names**: `function playerFire()`, `spawnBoss()`, `rectsOverlap()` etc. are searchable hooks
- **Locate game loop**: Search for `requestAnimationFrame(tick)` to find main update-render loop
- **Track player/enemy state**: Global `player` object and `enemies[]` array hold mutable game state
- **Canvas context methods**: `ctx.fillRect()`, `ctx.fillStyle`, `ctx.drawImage()`, `ctx.clearRect()` drive all visuals
- **Collision detection**: `rectsOverlap()` (line ~1069) checks AABB overlap; extend for custom shapes

## Testing in Browser

1. Open `index.html` directly in a browser (file:// or http://)
2. Test on desktop: keyboard controls (arrows, space, B for bomb)
3. Test on mobile: tap/drag joystick, press Fire/Bomb buttons
4. Check all three difficulty modes and all six themes via UI
5. Verify boss encounters spawn correctly (18 kills per stage)
6. Check localStorage persistence: play, close tab, reopen — best score and settings should persist
