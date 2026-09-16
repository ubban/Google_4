# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built entirely with HTML5 Canvas and the WebAudio API. The complete game is contained in a single `index.html` file (~1670 lines) with no external dependencies.

## Running & Testing

- **Play the game**: Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
- **No build step required**: This is a static HTML file—simply open it locally or serve it with any HTTP server
- **Testing**: Play through different difficulty levels (Easy/Normal/Hard) to verify game mechanics work correctly
- **Mobile testing**: The game is fully responsive and touch-enabled; test on various viewport sizes

## Code Architecture

The game is organized into distinct functional sections within a single IIFE (Immediately Invoked Function Expression) to avoid global scope pollution:

### HTML Structure & Styling (lines 1-359)
- **Canvas element** (`#gameCanvas`): 480×720px game render surface
- **UI overlays**: Score HUD, boss health bar, combo popups, stage banners
- **Control zones**: Touch-optimized fire/bomb buttons and joystick for mobile
- **Accessibility**: ARIA live region (`#srLive`) for screen reader announcements
- **Responsive CSS**: Uses `clamp()` for fluid sizing across all devices

### Game State (lines 863-870)
Global variables track:
- Player position, weapons, health, invulnerability
- Enemies, bullets (player & enemy), particles, power-ups, boss
- Score, lives, stage, kills, combo chain, difficulty
- Frame counter and timing deltas

### Core Game Loop (lines 1640-1653)
- `loop()` runs at 60 FPS via `requestAnimationFrame`
- Calls `update(dt)` for logic, then `draw()` for rendering
- Adaptive performance tracking adjusts visual quality on low-end devices

### Update Logic (lines 1148-1302)
`update(dt)` handles:
1. **Player movement**: Keyboard (arrows/WASD), mouse (pointer tracking), joystick (touch)
2. **Weapon firing**: Two weapon types (Vulcan, Laser) with different spread patterns
3. **Enemy spawning**: Difficulty-based spawn rates; three enemy types (Grunt, Interceptor, Tank)
4. **Collision detection**: Rect-overlap checks for bullets/enemies/boss, player damage
5. **Score & combo**: Chain kills to build multiplier (1×–10×)
6. **Boss lifecycle**: Enter → attack phase → death → next stage

### Rendering (lines 1520-1638)
`draw()` renders:
- Dynamic sky gradient (6 themed color schemes per stage)
- Parallax clouds
- Canvas 2D objects: player/enemies (custom `drawPlane()`), bullets, particles, power-ups
- Boss ship with animated core (`drawBossShip()`)
- Screen shake effect (when taking damage or using bombs)
- Particle explosions with gravity and fade

### Audio Subsystem (lines 542-676)
- **No audio files**: All SFX generated in real-time with WebAudio
- `beep()`: Frequency sweeps with oscillators and filters
- `noiseBurst()`: Procedural noise for impact sounds
- **SFX catalog**: shoot, laser, explode variants, hit, bomb, powerup, boss attacks, game over
- Master gain & compressor for consistent volume across all sounds
- Mute toggle stored in localStorage

### Input Handling (lines 767-861)
- **Keyboard**: Arrow keys + WASD for movement, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move cursor over canvas to steer, click to fire
- **Touch**: Left zone for movement (joystick), right zone for fire/bomb buttons
- Input states managed in `keys{}` object; mouse/joystick override keyboard when active

### Boss System (lines 1372-1454)
- **6 bosses** with unique designs and attack patterns
- Each boss has 3 attack patterns that rotate
- Patterns include: radial bursts, aimed spreads, spiral patterns, homing shots, teleport dashes
- Boss telegraph warning (visual glow + sound) before firing
- Adaptive health scaling based on difficulty and boss type
- Boss defeated → advance to next stage with new theme

### Difficulty System (lines 485-489)
Three difficulty presets affect:
- Enemy spawn rate & minimum spawn delay
- Enemy movement speed & shoot chance
- Player starting lives (3 or 4) & bombs (2 or 3)
- Boss health multiplier

### Persistence (localStorage)
- Best score
- Mute state
- Difficulty selection
- Reduced motion preference

## Key Game Mechanics

1. **Weapon System**: Vulcan (spread shots) upgrades to Level 3, then switches to Laser (piercing shots) with separate upgrades
2. **Power-ups**: Life, Bombs, Weapon upgrades spawn from defeated enemies (~6% drop rate)
3. **Combo Multiplier**: Builds from continuous kills; resets after ~2.2 seconds without a kill
4. **Boss Patterns**: Each boss cycles through its pattern set; damage from bombs is 24 HP vs. 3 HP per bullet
5. **Invulnerability**: 1.5 seconds after taking damage; visual blink while active
6. **Stages**: Infinite progression; visual themes cycle every 6 stages

## Performance Optimization

- **Adaptive quality**: Monitors frame time; downgrades particle count / glow effects on low-end devices
- **Object pooling**: Bullets and particles are destroyed when off-screen; no heap churn
- **Reduced motion**: Respects `prefers-reduced-motion` media query; disables shake/flash/animations on user preference
- **Canvas scaling**: Maintains 480×720 internal resolution; CSS scales for display (up to 1.6× on large screens)

## Common Development Tasks

- **Add a new enemy type**: Define in `ENEMY_TYPES` object (line 502), add spawn logic in `spawnEnemy()` (line 1022)
- **Add a boss attack pattern**: Create function in `BOSS_PATTERNS` array (line 1372); use helper functions like `fireRadial()`, `fireAimedSpread()`
- **Adjust difficulty**: Modify `DIFFICULTIES` constants (line 485) or tune spawn/speed/health multipliers
- **Add a new theme**: Add object to `THEMES` array (line 493) with sky colors, cloud/sun colors, and name
- **New sound effect**: Create function in `sfx` object (line 624) using `beep()` or `noiseBurst()` primitives
- **UI changes**: Modify HTML structure and inline CSS (lines 1-359); update element references in JS

## Notable Implementation Details

- **Collision**: Axis-aligned rectangle overlap via `rectsOverlap()` (line 1069)
- **Screen shake**: Random offset to canvas context on each frame proportional to magnitude & duration
- **Enemy weaving**: Interceptors oscillate side-to-side using `Math.sin(frame * 0.05)`
- **Boss telegraph**: Visual whitening + pulsing glow; audio cue played 22 frames before pattern fires
- **Particle system**: Position, velocity, life tracking; render as circles with alpha fade
- **Chain multiplier formula**: `1 + Math.min(9, Math.floor(chain / 5))` → max 10× from 50+ consecutive kills

## Accessibility

- **Touch controls**: Optimized for mobile with large target areas
- **Keyboard support**: Full game playable without mouse
- **Screen reader support**: Status announcements via ARIA live region (score changes, boss names, game state)
- **Reduced motion**: Respects prefers-reduced-motion; disables screen shake, flash, and animation
- **High contrast**: Dark theme with bright UI elements for visibility
