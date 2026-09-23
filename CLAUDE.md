# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely in a single HTML file. It runs directly in any modern browser without build tools, package managers, or external dependencies. The game includes full gameplay mechanics, audio synthesis, touch controls, and accessibility features.

## Architecture

### Single-File Architecture
The entire game lives in `index.html` (~1670 lines):
- **Lines 1-359**: HTML structure and CSS styling
- **Lines 362-419**: DOM elements (canvas, HUD, overlay, touch controls)
- **Lines 422-1669**: Game logic in a single IIFE (Immediately Invoked Function Expression)

### Key Systems

**Canvas Rendering** (lines 754-765)
- Base resolution: 480×720px
- Scales responsively to fit window while maintaining aspect ratio
- Max scale: 1.6x
- Uses 2D canvas context for all graphics

**Game Loop & State** (lines 1600+)
- Runs at 60fps via `requestAnimationFrame`
- Three main states: title overlay, gameplay, paused
- Tracks: player position/health, enemies, projectiles, boss, score, combo

**Entity System**
- Player: ship position, health, weapon level, bombs
- Enemies: position, type, health, AI behavior
- Boss: phase-based attacks, visual patterns, health bar
- Projectiles: bullets from player and enemies, smart bombs
- Pickups: weapon upgrades and bomb refills

### Difficulty System (Lines 485-489)
Three presets modify game balance:
- **Easy**: Slower spawns, weaker enemies, more lives/bombs
- **Normal**: Default balance (starting difficulty)
- **Hard**: Fast enemy waves, higher shoot chance, tougher boss

Each difficulty scales: spawn rates, enemy speed, enemy shoot chance, player lives/bombs, boss HP multiplier

### Stage & Boss System
- **KILLS_PER_STAGE** (line 491): 18 kills advance to next stage
- **BOSSES** (lines 508-515): 6 boss types with unique visuals and HP multipliers
- **THEMES** (lines 493-500): 6 visual themes with different sky colors and sun/cloud styling
- Boss appears after clearing stage, then defeats reset to next stage with new boss type

### Weapon & Power-Up System
Weapons tracked by name and level:
- **VULCAN**: Starting weapon, rapid fire
- **LASER**: Higher damage, narrower spread
- Player upgrades weapon by collecting pickups
- Weapon level shown in HUD; resets when hit

### Audio System (Lines 542-676)
**WebAudio API synthesis** — no external audio files:
- Creates sine/square/sawtooth oscillators
- Applies gain envelopes, frequency sweeps, filters
- `beep(freq, duration, type, volume, opts)`: melodic tones
- `noiseBurst(duration, volume, opts)`: noise-based effects (explosions, bombs)
- Stereo panning based on entity position for spatial audio
- Master gain with dynamics compression

SFX defined in `sfx` object (lines 624-676):
- `shoot`, `laser`, `explode*`, `hit`, `bomb`, `powerup`, `boss*`, `stageClear`, `uiClick`, `gameover`

### Performance Tracking (Lines 473-483)
Adaptive quality system monitors frame times:
- Tracks last 45 frame deltas
- If avg > 1/40ms (< 40fps): quality increases (more rendering skipped)
- If avg < 1/56ms (> 56fps): quality decreases
- Quality levels: 0 (full), 1 (medium), 2 (low) — affects particle/visual counts

### Input Handling

**Keyboard** (lines 768-776):
- Arrow keys + WASD: move ship
- Space: fire
- X or Shift: use bomb
- P or Esc: pause

**Mouse** (via game loop):
- Move to steer, click to fire

**Touch** (lines 779-850):
- Drag in left zone (55% width, 55% height from bottom) to move
- Tap FIRE button to shoot
- Tap BOMB button to use bomb
- Joystick visual appears during drag

### Accessibility Features
- **Reduced Motion** (lines 529-540): Button to disable shake, flash, pulse animations
- **Screen Reader** (line 390, 468-471): `srLive` element with `aria-live="polite"` for announcements
- **Semantic HTML**: Proper button roles, title attributes, controls hint text

### User Preferences (Local Storage)
Persisted via localStorage:
- `skyfighter.bestScore`: High score
- `skyfighter.muted`: Audio on/off
- `skyfighter.difficulty`: Selected difficulty
- `skyfighter.reducedMotion`: Motion reduction preference (defaults to system `prefers-reduced-motion`)

### HUD & Overlay
- **HUD** (lines 367-374): Live score, best score, multiplier, weapon, health hearts, bomb count
- **Boss Bar** (lines 376-379): Shows when boss active, fills down as boss takes damage
- **Title Overlay** (lines 401-419): Menu with difficulty picker, start button, controls hint
- **End Screen**: Shows final score, stage, best score, and new-best notification

### Visual Effects
- **Screen shake**: Called on player hit or bomb; reduced under reduced-motion
- **Flash**: White flash on hit/bomb; removed under reduced-motion
- **Combo popup**: Animated text showing hit combo at screen center
- **Stage banner**: Announcement of new stage with fade animation
- **Vignette**: Red pulse effect when health critical (1 heart remaining)

## Common Development Tasks

### Testing/Running
The game runs in any modern browser:
1. Open `index.html` in a web browser
2. Click "Start Game" to begin
3. Use keyboard, mouse, or touch controls depending on device

### Making Balance Changes
Edit the DIFFICULTIES object (lines 485-489):
- `spawnBase`/`spawnMin`: Control how fast enemies appear
- `enemySpeed`: Multiplier for all enemy velocities
- `enemyShootChance`: Probability (0-1) enemy fires each frame
- `lives`/`bombs`: Starting player lives and bombs
- `bossHpMul`: Multiplier on boss HP

### Adding New Enemy Types
Add entry to ENEMY_TYPES (lines 502-506) with properties:
- `w`, `h`: Width/height in pixels
- `hp`: Hit points
- `speedMul`: Speed multiplier
- `color`, `dark`: Main and shadow colors
- `score`: Points for killing
- `minStage`: First stage this enemy appears
- `weave` (optional): Makes enemy zigzag pattern

Update spawn logic around line 1350 to check `minStage`.

### Adding New Bosses
Add to BOSSES array (lines 508-515) with name and visual colors:
- `hullTop`/`hullBottom`: Hull gradient colors
- `wing`: Wing color
- `core`: Core/weak-point color
- `hpMul`: Health multiplier on base boss HP
- `name`: Display name

Update boss drawing routine (around line 1420) and attack patterns (around line 1485) if you want unique behaviors.

### Adding Visual Themes
Add to THEMES array (lines 493-500):
- `sky`: Array of 3 colors for sky gradient
- `cloud`, `sun`: RGB values as strings (e.g., `'255,255,255'`)
- `name`: Display name

Selected by stage; cycles through list as player advances.

### Modifying Sounds
Edit specific `sfx` functions (lines 624-676):
- Change `beep()` calls: adjust freq (Hz), duration (sec), type ('sine'/'square'/'sawtooth'), volume (0-1)
- Change `noiseBurst()` calls: duration, volume, optional filter and pan
- Use `opts.delay` to time multiple sounds together
- Use `opts.sweepTo` for frequency glide over duration
- Use `opts.filterFreq` to set lowpass/highpass cutoff

### Tuning Visuals
**Canvas size**: Edit BASE_W, BASE_H (line 753)

**Colors**: Search for inline color hex strings:
- Player ship: search for player drawing code (~line 1280)
- Enemy colors: defined in ENEMY_TYPES, drawn per type
- Boss colors: defined in BOSSES, drawn per type
- UI colors: CSS in `<style>` block (lines 10-359)

**Particle systems**: Disabled at quality level 2; see line ~1550 for spawn logic

**Text styling**: CSS animations and fonts in style block

## Code Style & Patterns

- **Scope**: Single IIFE maintains closure over game state
- **Naming**: camelCase for variables/functions, UPPER_CASE for constants
- **No external dependencies**: Pure vanilla JavaScript, no libraries
- **Performance first**: Frame-time tracking, adaptive quality, efficient loops
- **Accessibility built-in**: Reduced motion respects user preference, screen reader announcements

## Browser Support

Requires:
- ES6 (arrow functions, const/let, template literals)
- Canvas 2D API
- Web Audio API (silent fallback if unavailable)
- localStorage (silent fallback if unavailable)
- Touch Events API (for mobile)
- Tested on modern Chrome, Firefox, Safari, Edge

