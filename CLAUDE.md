# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up browser game. It's a single-file HTML5 application with embedded CSS and JavaScript. The game runs entirely in the browser without build tools or external assets.

## Quick Start

1. **Run the game**: Open `index.html` in a modern web browser
2. **No build step required** — the game runs directly from the HTML file

## Game Architecture

The game is implemented as a monolithic state machine with these core systems:

### Game State Management
- **Player state**: position, weapon level, cooldown, invulnerability timer
- **Score & progression**: lives, bombs, kills, stage number, combo multiplier
- **Entities**: bullets (player), enemies, enemy bullets, particles, power-ups, boss
- **Audio context**: initialized on first user interaction
- **UI state**: paused, game over, overlay state

### Core Game Loop
1. **Update phase** (`update(dt)`): processes input, moves entities, detects collisions, spawns enemies/bosses, applies physics
2. **Render phase** (`draw()`): clears canvas, draws background, entities, and UI
3. **Performance tracking**: adaptive quality system reduces particle/glow effects under 40fps

### Entity Systems

**Bullets (Player)**
- Three bullet types: Vulcan (straight spread), Laser (piercing with multi-lane fire)
- Weapon levels control spread pattern and fire rate
- Laser level 3 enables multi-lane and piercing behavior

**Enemies**
- Three types: Grunt (weak, common), Interceptor (fast, weaving), Tank (tanky, high HP)
- Spawn rate increases with difficulty; tanks spawn at stage 2+
- AI: shoot pattern (spread or aimed), weaving movement on interceptors
- Difficulty levels control spawn rates, enemy speed, shoot chance, HP multipliers

**Boss System**
- Spawn at 18 kills per stage
- Six boss designs with unique attack patterns (radial, aimed spreads, homing, teleport)
- Telegraph phase (22 frames) before attack pattern fires
- Boss health scales with stage and difficulty

**Power-Ups**
- Life restore (6% spawn chance)
- Bomb (8% spawn chance)
- Weapon upgrade (20% spawn chance) — cycles through weapon levels, then switches weapon

### Input Handling
- **Keyboard**: Arrow keys or WASD for movement, Space for fire, X/Shift for bombs, P/Esc for pause
- **Mouse**: movement steers the plane when hovering canvas; click to fire
- **Touch**: left 55% of screen for joystick movement, right side buttons for fire/bomb
- Mouse and keyboard/joystick input prioritize keyboard; mouse only activates if no directional input

### Sound System
- **WebAudio synthesis only** — no external audio files
- Synth creates sound effects using oscillators, filters, and noise bursts
- Master gain (0.9) feeds through compressor for dynamic range
- Spatial panning for left/right effects based on x-position
- Mute state persists in localStorage

### Difficulty Settings
Three presets control spawn timing, enemy speed, shoot chance, lives, bombs, boss HP:
- **Easy**: slower spawns, slower enemies, fewer bomb drops, lower boss HP
- **Normal**: balanced (default)
- **Hard**: rapid spawns, fast enemies, same limited bombs, higher boss HP

### Progression & Scoring
- **Chain multiplier**: +1x per 5 consecutive kills (capped at 10x); resets if 2.2s passes without a kill
- **Combo announcements**: "10 KILLS!", "20 KILLS!" every 10th kill
- **Boss rewards**: 500 points per defeated boss
- **Stage themes**: 6 visual themes cycle based on stage number

### Adaptive Performance
- Frame time tracking averages 45 frames of data
- Quality levels: 0 (full glow/particles), 1 (60% particles), 2 (35% particles)
- Dynamically scales down glows and shadow effects at low quality

## Key Code Sections

**Initialization & State** (lines 423–527)
- DOM element references, storage keys, difficulty definitions
- Best score/mute/difficulty/reduced-motion loaded from localStorage

**Update Loop** (lines 1148–1302)
- Input processing, entity movement, collision detection, spawning logic
- Chain multiplier decay, bomb/weapon usage

**Rendering** (lines 1520–1638)
- Canvas context setup, screen shake offset
- Background gradient and cloud layers (cycle with parallax)
- Entity drawing (player plane, bullets, enemies, boss, particles)

**Boss Patterns** (lines 1372–1409)
- 6 unique pattern sets (one per boss design)
- Pattern functions: fireRadial, fireAimedSpread, fireColumns, fireSpiralBurst, etc.

**Audio Functions** (lines 566–676)
- `beep()`: tone synthesis with optional sweep and filters
- `noiseBurst()`: filtered noise for explosive effects
- `sfx.*`: sound effect definitions for all gameplay events

**Drawing Helpers** (lines 913–1020)
- `drawPlane()`: renders player/enemy aircraft with flame when firing
- `drawBossShip()`: renders boss with animated core and wings

## Development Notes

### Canvas Sizing
- Base resolution: 480×720 (portrait-oriented)
- Scales up to 1.6× on desktop (max 768×1152)
- Responsive to window resize and orientation change

### Mobile Support
- Touch controls show on touch devices (detected via `ontouchstart` or `maxTouchPoints`)
- Joystick UI for movement (left 55%), buttons for fire/bomb (right 45%)
- Safe area insets for notch/home indicator safety

### Accessibility
- Reduced motion mode respects `prefers-reduced-motion` and can be toggled
- Screen reader announcements for kills, lives, stage progression via `srLive`
- Color-based visual feedback supplemented with text descriptions

### Storage Keys
- `skyfighter.bestScore`: high score persistence
- `skyfighter.muted`: sound mute state
- `skyfighter.difficulty`: selected difficulty (easy/normal/hard)
- `skyfighter.reducedMotion`: accessibility override

### Common Modifications

**Change difficulty balance**: Edit `DIFFICULTIES` object (line 485)
**Add/modify enemy types**: Edit `ENEMY_TYPES` (line 502)
**Adjust boss patterns**: Add/modify functions in `BOSS_PATTERNS` array (line 1372)
**Tweak colors/theme**: Edit `THEMES` array (line 493) or individual colors in CSS
**Adjust spawn rates**: Modify `spawnTimer` calculation in `update()` (line 1202)

### Testing Locally
1. Open `index.html` directly in a browser (no server needed)
2. All features work: keyboard, mouse, touch, sound, localStorage
3. Pause (P/Esc) to inspect game state in DevTools console
4. Clear localStorage to reset best score: `localStorage.clear()`

## Known Constraints

- Single-file design means no code splitting; entire game loads at once
- Canvas rendering (not GPU-accelerated WebGL) — performance may be limited on weak devices
- Audio context requires user interaction to initialize (click/keypress first)
- No server-side state — progress only persists locally via localStorage
