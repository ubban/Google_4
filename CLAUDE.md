# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. The game features:
- Player aircraft that can switch between two weapon types (VULCAN and LASER)
- Enemy spawning with increasing difficulty and variety across stages
- Boss encounters at the end of each stage (6 unique boss patterns)
- Procedural terrain themes (6 different sky/cloud/sun color schemes)
- Score tracking with difficulty-based high scores
- Mobile-friendly touch controls alongside keyboard/mouse support
- Completely procedural audio synthesis (no external audio files)

## Architecture

The entire game is implemented in a single `index.html` file (~1,670 lines) using vanilla JavaScript. There are no external dependencies—all rendering uses the Canvas 2D API and audio is synthesized using the WebAudio API.

### Core Game Systems

**Entity Management:**
- `player` — the player's aircraft with position, weapon state, cooldown timers, and invulnerability
- `enemies` — dynamically spawned array of enemies with different types (grunt, interceptor, tank)
- `bullets` — player projectiles
- `enemyBullets` — incoming fire from enemies
- `boss` — single boss entity active during stage boss phases, follows hard-coded attack patterns
- `powerUps` — weapon upgrades and bomb pickups

**Game Loop & State:**
- Main rendering loop uses `requestAnimationFrame` with delta-time calculation
- Global state: `score`, `lives`, `stage`, `stageKills`, `multiplier` (combo chain)
- Game states: `running`, `paused`, `gameOver`
- Difficulty levels (easy/normal/hard) stored in localStorage with configurable spawn rates, enemy speeds, and boss HP multipliers

**Input Handling:**
- Keyboard: WASD/Arrows to move, Space to fire, X/Shift for bomb
- Mouse: hover to steer (crosshair cursor), click to fire
- Touch: left half of screen is joystick (drag to move), right half has FIRE/BOMB buttons
- Mouse and keyboard inputs take priority over touch when both are active

**Audio:**
- Pure WebAudio synthesis with no audio files—all sounds are procedural beeps and noise bursts
- Audio context initialized on first user input to comply with browser autoplay policies
- Master gain + compressor chain for dynamic range control
- Spatial audio: panning calculations (`panFor()`) place sounds across the stereo field based on screen position

**Performance Adaptation:**
- Tracks average frame time over 45 frames
- Automatically scales quality setting (0=full, 1=medium, 2=low) to maintain ~40 FPS
- Currently used for particle effects rendering optimization

### Data Structures

**DIFFICULTIES** — defines spawn rates, enemy speed multipliers, starting lives/bombs, and boss HP scaling for easy/normal/hard
**ENEMIES_TYPES** — enemy stats: width, height, HP, speed multiplier, color, score value, and minimum stage unlock
**BOSSES** — 6 boss variants with hull colors, core color, and HP multipliers
**BOSS_PATTERNS** — array of boss behavior patterns defining attack sequences (found around line 1372)
**THEMES** — 6 visual themes with sky gradient colors, cloud/sun tones, and theme names

### Key Functions

- `resetGame()` — reinitialize all entities and state to start a new game
- `update(dt)` — advance game state: move entities, check collisions, handle spawning
- `render()` — draw all entities and UI elements to canvas
- `fireWeapon()` — fire the current weapon (VULCAN or LASER)
- `useBomb()` — trigger smart bomb (damages all on-screen enemies)
- `togglePause()` — pause/unpause the game
- Audio functions: `beep()` and `noiseBurst()` synthesize all sound effects

## Development Workflow

### Running the Game

The game is a single HTML file—simply open `index.html` in a web browser. No build step, no server needed.

```bash
# Option 1: Open directly in browser
open index.html

# Option 2: Run a simple server (if you need it for testing)
python -m http.server 8000
# Then visit http://localhost:8000/index.html
```

### Testing & Development

- **Live editing:** Edit `index.html` directly and reload the browser to see changes
- **Console debugging:** The game exposes key variables in the IIFE scope—open DevTools to check `score`, `frame`, etc. (though some are not globally accessible due to closure)
- **Difficulty testing:** Use the in-game UI buttons to select Easy/Normal/Hard before starting
- **Mobile testing:** Use browser DevTools device emulation or test on an actual device with touch screen

### Code Locations

- **HTML Structure & Styling:** Lines 1–359
  - Canvas setup, HUD elements, overlay screens, mobile touch control zones
  - CSS includes animations (heartbeat, combo pop, stage banner, screen flash)

- **Audio Setup & SFX Definitions:** Lines 542–676
  - WebAudio context initialization and master gain setup
  - `beep()` and `noiseBurst()` synthesizers
  - Individual SFX definitions for all game sounds

- **Game State & Constants:** Lines 863–899
  - Initial player stats and entity arrays
  - Difficulty configuration
  - Enemy and boss definitions

- **Main Game Loop:** Lines ~1400+ (approx, varies with boss patterns)
  - `update()` function: collision detection, enemy spawning, score multipliers
  - `render()` function: canvas drawing, HUD updates
  - `requestAnimationFrame` setup

### Common Modifications

**Adding a New Enemy Type:**
1. Add entry to `ENEMY_TYPES` object with width, height, HP, speed multiplier, color, score, and minimum stage
2. In the spawning logic (inside `update()`), add the new type to the random selection pool
3. Define any special attack patterns if needed

**Adjusting Difficulty Balance:**
- Edit `DIFFICULTIES[difficulty]` values: `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- These are applied when `resetGame()` runs

**Adding a New Visual Theme:**
- Add a new entry to `THEMES` array with sky gradient colors (`[top, middle, bottom]`), cloud RGB, sun RGB, and name
- The theme selection happens randomly on stage start

**Creating a New Boss:**
- Add boss definition to `BOSSES` array with hull colors and HP multiplier
- Define a corresponding pattern in `BOSS_PATTERNS` array

**Audio Adjustments:**
- Edit the `sfx` object to modify existing sound parameters (frequency, duration, envelope)
- All sounds use `beep()` (tonal) or `noiseBurst()` (percussion/explosion) primitives
- Use `opts.delay` for layered sounds (e.g., `sfx.bomb` chains multiple beeps)

### Testing Checklist

- Game starts and loads the title screen without errors
- Difficulty buttons work and persist (check localStorage via DevTools)
- Player moves and fires with keyboard, mouse, and touch input
- Enemies spawn, move, and fire at increasing difficulty
- Enemies die on hit, award score multiplier increases apply
- Boss appears after KILLS_PER_STAGE (18) kills and fights with expected pattern
- Screen flash/vignette on hit (unless reduced motion enabled)
- Audio plays correctly and responds to mute toggle
- Score and high score update and persist in localStorage
- Game over screen appears when lives reach zero

## Accessibility & Compatibility

- **Reduced Motion:** Respects `prefers-reduced-motion` media query and provides a toggle to disable animations
- **Keyboard Navigation:** All controls accessible via keyboard; no mouse required for core gameplay
- **ARIA Live Region:** Screen reader updates via `#srLive` element for game events
- **Mobile:** Full responsive design with touch joystick and button controls
- **Browser Support:** Requires ES6, Canvas 2D API, and WebAudio API (all modern browsers)

## Responsive Design

- Base canvas size: 480×720 pixels (portrait orientation, arcade cabinet aspect)
- Scales up to fit viewport with max scale of 1.6× (uses `Math.min()` to prevent excessive enlargement)
- Safe area insets respected for notched/foldable devices (via CSS `env(safe-area-inset-*)`)
- Touch control zones adapt: movement zone (left 55%, bottom 55%), action zone (right 45%, bottom 55%)

## Performance Notes

- Adaptive quality system tracks frame times and reduces rendering complexity if average FPS drops
- Collision detection is O(n) per bullet per enemy—scales well for typical enemy counts (< 50)
- All audio synthesis is real-time; muting disables audio context to save CPU
- Canvas redraws every frame; no dirty-rect optimization (unnecessary for this scale)
