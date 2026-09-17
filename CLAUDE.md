# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. The entire game is implemented as a single self-contained HTML file (`index.html`) with embedded Canvas rendering and WebAudio synthesized sound effects. No external dependencies or build tools are required.

## Quick Start

- **Open the game**: Open `index.html` directly in any modern web browser
- **Test locally**: Use `python -m http.server` or similar to serve the file for local testing
- **No build step required**: Changes to the code take effect immediately upon browser refresh

## Code Architecture

The game logic is organized within a single IIFE (Immediately Invoked Function Expression) starting at line 423. The structure follows these key sections:

### 1. **Game State & Configuration** (Lines 463-530)
- `DIFFICULTIES`: Three difficulty presets (easy, normal, hard) with spawn rates, enemy speed, lives, bombs, boss HP scaling
- `THEMES`: Six visual themes with sky gradients, cloud colors, and sun colors
- `ENEMY_TYPES`: Three enemy archetypes (grunt, interceptor, tank) with stats and scoring
- `BOSSES`: Six boss designs with unique visual colors and HP multipliers
- Global state: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`

### 2. **Audio Subsystem** (Lines 542-676)
- **WebAudio synthesis**: All sound effects are generated dynamically using oscillators, noise, and filters
- `ensureAudio()`: Initializes the AudioContext on first user interaction
- `beep()`: Creates melodic tones with optional frequency sweep and filtering
- `noiseBurst()`: Generates noise-based explosion and impact sounds
- `sfx` object: Pre-defined sound effects for game events (shoot, explode, hit, bomb, powerup, etc.)

### 3. **Rendering & Layout** (Lines 752-1638)
- **Canvas sizing**: Fixed 480×720 internal resolution, scaled to fit viewport
- **Theme system**: Each stage cycles through visual themes via `THEMES`
- `draw()`: Main render function using 2D Canvas API
- `drawPlane()`: Renders player and enemy aircraft
- `drawBossShip()`: Renders boss with animated features
- `drawClouds()`: Parallax cloud layers with stage-dependent coloring
- Screen shake effect applied via canvas context translation

### 4. **Input Handling** (Lines 767-861)
- **Keyboard**: Arrow keys and WASD for movement, Space to fire, X/Shift for bomb, P/Esc for pause
- **Touch controls**: Virtual joystick on left half, fire/bomb buttons on right
- **Mouse**: Player glides toward cursor; left-click fires
- Input priority: Keyboard/joystick > mouse steering (avoids fighting between input methods)

### 5. **Game Loop & Updates** (Lines 1148-1302, 1640-1665)
- `update(dt)`: Logic tick (movement, collision, spawning, state changes)
- `draw()`: Render tick
- `requestAnimationFrame(loop)`: Main loop with delta-time calculation and frame-rate tracking
- **Adaptive performance**: `trackPerf()` adjusts quality level (0=full, 2=low) based on average frame times

### 6. **Boss AI & Attack Patterns** (Lines 1324-1439)
- `BOSS_PATTERNS`: Array of pattern functions for each of 6 boss designs
- Pattern functions: `fireRadial()`, `fireAimedSpread()`, `fireColumns()`, etc.
- Boss telegraph mechanic: 22-frame warning before attack fires
- Boss movement: Bounces between canvas edges

### 7. **Game States & Flow**
- **Menu/Overlay**: Difficulty selection, stats display, start/resume button
- **Running**: Player controls active, enemies spawn, physics/collision run
- **Paused**: Input ignored except pause toggle
- **Game Over**: Final stats and high score check
- **Stage transitions**: Boss defeat triggers next stage banner and new theme

## Key Game Mechanics

### Scoring & Multiplier
- Kill chain increments `chain` counter when enemies are defeated
- `chainTimer` resets every 2.2 seconds of inactivity
- `multiplier = 1 + floor(chain / 5)`, capped at 10x
- Boss kills award flat 500 points

### Weapons System
- **Vulcan**: Spread increases at level 2-3 (1→2→3 shots); spreads up to ±0.24 radians
- **Laser**: Pierces enemies; level 2-3 adds parallel lanes (±6px offset); capped at 3 shots
- Power-ups cycle: weapon level up to 3, then toggle between Vulcan ↔ Laser and reset to level 1

### Enemy Types & Spawning
- `spawnEnemy()`: Selects type randomly, constrained by stage; weaving interceptors use sine wave paths
- Spawn rate decreases with time via `difficultyRamp` (every 300 frames of gameplay)
- Tank enemies show visible HP bar above them

### Boss Lifecycle
- Boss appears after 18 kills (`KILLS_PER_STAGE`)
- "Entering" phase: Boss descends for ~1 second
- Attack cycles: Telegraph (22 frames) → Attack fires → Cooldown (85 frames)
- 6 bosses rotate with stage number: `(stage - 1) % BOSSES.length`

### Power-Ups & Drops
- Spawned on enemy death with probabilities: 6% life, 8% bomb, 20% weapon, 66% nothing
- Move downward at 90px/s; picked up on collision with player

## Storage & Preferences
- `localStorage` keys:
  - `skyfighter.bestScore`: High score
  - `skyfighter.muted`: 1/0 for audio mute state
  - `skyfighter.difficulty`: Current difficulty selection
  - `skyfighter.reducedMotion`: 1/0 for reduced motion (respects OS prefers-reduced-motion)

## Accessibility Features
- Reduced motion mode: Disables animations (screen shake, flash, vignette pulse)
- Screen reader announcements via `announce()` using ARIA live region
- Touch controls for mobile devices with explicit FIRE/BOMB buttons
- Keyboard shortcuts clearly documented in UI hint text

## CSS & Styling
- Dark color scheme (`#05122b`, `#0b1d3a` backgrounds)
- All colors defined inline in style section (lines 10–358)
- Responsive units: `clamp()` for font sizing and spacing
- Safe area insets respected for notched devices

## Common Development Tasks

### Adding a New Enemy Type
1. Add entry to `ENEMY_TYPES` with `{ w, h, hp, speedMul, color, dark, score, minStage, weave? }`
2. Update `spawnEnemy()` probability logic if needed
3. Add explosion SFX to `EXPLODE_SFX` mapping if unique sound desired
4. If weave=true, interceptors already weave automatically; tanks don't

### Adding a New Boss
1. Add design to `BOSSES` array with colors and `hpMul`
2. Create attack pattern function in `BOSS_PATTERNS` array (must match `BOSSES` index order)
3. Pattern functions receive boss object and call `fireRadial()`, `fireAimedSpread()`, etc. to populate `enemyBullets`

### Adding a New Visual Theme
1. Add theme object to `THEMES` with sky gradient stops, cloud/sun colors, and name
2. Themes cycle via `(stage - 1) % THEMES.length`, so order determines which stages use which themes

### Adjusting Game Balance
1. Edit `DIFFICULTIES` object: spawn rates, enemy speed, shoot chance, lives, bombs, boss HP scaling
2. Edit `KILLS_PER_STAGE` to change boss trigger point
3. Edit `BOSS_PATTERNS` arrays to tweak attack difficulty

### Tweaking Performance
- Quality levels: 0=full rendering, 1=medium (60% particles), 2=low (35% particles)
- Toggled automatically based on frame time averages
- Glow effects (`ctx.shadowBlur`) skipped when quality > 0

## Debugging Tips

- Open browser DevTools console to see any errors
- Check `localStorage` via DevTools → Application tab
- Pause and resume via P or Esc keys (UI button always available)
- High frame times in DevTools Performance panel indicate GPU/CPU bottleneck
- Audio context issues often occur if game is not triggered by user interaction first
