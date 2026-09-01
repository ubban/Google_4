# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game. It's a single-file implementation (index.html) with no build tools, dependencies, or external assets. The game features:
- Player-controlled aircraft with weapon upgrades
- Multiple enemy types with varying behavior
- Boss encounters with procedural attack patterns
- Difficulty settings (Easy/Normal/Hard)
- Web Audio API-based dynamic sound synthesis
- Touch and keyboard controls
- Persistent score tracking via localStorage

## Development

Since this is a single-file HTML/CSS/JavaScript application, there are **no build steps, package managers, or linting tools**. Development is straightforward:

1. **Open in browser**: Open `index.html` directly in any modern web browser
2. **Edit and reload**: Make changes to `index.html` and reload the page to test
3. **No dependencies**: All code is vanilla JavaScript—no frameworks or libraries

## Code Architecture

The entire game is structured in a single `<script>` IIFE (Immediately Invoked Function Expression) that encapsulates all state and logic. Key sections:

### Input Handling (lines 767–861)
- Keyboard input via `keydown`/`keyup` events
- Mouse steering and click-to-fire
- Touch controls with on-screen joystick and action buttons
- Auto-initializes Web Audio context on user interaction (browser requirement)

### Game State (lines 863–912)
- **player**: Position, weapon, level, invulnerability frames
- **bullets**, **enemies**, **enemyBullets**, **particles**, **powerUps**, **boss**: Pooled game objects
- **score**, **lives**, **stage**, **multiplier**: Session state
- **DIFFICULTIES**: Three preset configurations (Easy/Normal/Hard) controlling spawn rate, enemy speed, and HP scaling
- Persistent storage: Best score, muted state, difficulty, reduced motion preference

### Update Loop (lines 1148–1302)
- **update(dt)**: Core game logic runs at 60fps
  - Player movement (keyboard + joystick + mouse steering)
  - Weapon firing and cooldowns
  - Collision detection (player ↔ enemies, bullets ↔ enemies, etc.)
  - Enemy spawning with progressive difficulty ramp
  - Scoring and combo/multiplier system
  - Stage progression (18 kills triggers boss warning; boss spawning)
  - Chain timer: killing enemies resets when not attacked for 2.2s

### Rendering (lines 1520–1638)
- Canvas-based 2D rendering at 480×720 native resolution (scales to fit viewport)
- Dynamic sky gradient and clouds per stage (6 themes)
- Screen shake and flash vignette effects (respects `prefers-reduced-motion`)
- Glow effects (shadow/blur) for bullets and power-ups on high-quality settings

### Audio (lines 542–676)
- **No external audio files**: All sound is synthesized with Web Audio API
- Each effect (shoot, explode, boss hit, etc.) uses oscillators, noise buffers, and filters
- Stereo panning based on where events occur on screen
- Dynamic compressor limits volume

### Difficulty & Content

**Enemy Types** (lines 502–506):
- **Grunt**: Basic red enemy, 1 HP
- **Interceptor**: Fast orange enemy with weaving pattern, 1 HP
- **Tank**: Durable purple enemy, 4 HP, appears at stage 2+

**Boss Patterns** (lines 1372–1409):
- Six unique boss designs, each with 3 distinct attack patterns
- Patterns include: radial bursts, aimed spreads, columns, spirals, teleportation, homing projectiles
- Boss health scales with stage and difficulty multiplier

**Power-ups**:
- **Life**: Extra health
- **Bomb**: Detonates all enemies on screen
- **Weapon**: Upgrades or switches between VULCAN (spread shot) and LASER (piercing shots)
- 3 levels per weapon; cycling at max level switches weapons

### Performance Optimization (lines 473–483)
- Adaptive quality system: tracks frame time, adjusts shadow/glow rendering based on FPS
- Spawned particles, bullets, and enemies are culled when off-screen

## Key Implementation Details

**Collision Detection** (line 1069): Simple rectangle overlap—no pixel-perfect detection
- All entities are axis-aligned rectangles

**Chain & Multiplier System**:
- Killing an enemy increments the chain counter
- Chain resets if player isn't hit but takes 2.2s without a kill
- Multiplier ranges from 1 to 10 (every 5 chain kills)

**Boss Telegraph**: Before each boss attack, there's a 22-frame (366ms) telegraph phase with a visual cue (glowing core) and audio warning

**Canvas Scaling**: Game renders at 480×720 but scales to fit the viewport with `Math.min(maxW / BASE_W, maxH / BASE_H, 1.6)` to maintain aspect ratio

**Reduced Motion**: Respects `prefers-reduced-motion` media query; disables all CSS animations and screen shake/vignette effects

## Common Development Tasks

### Add a new enemy type
1. Add an entry to `ENEMY_TYPES` (line 502–506) with width, height, HP, color, and spawn conditions
2. Define behavior in `spawnEnemy()` (around line 1022) if needed (e.g., weaving pattern)
3. Add explosion sound effect to `EXPLODE_SFX` (line 678) if using custom sfx
4. Adjust spawn probabilities in `spawnEnemy()` (lines 1028–1030)

### Add a new boss
1. Add entry to `BOSSES` array (line 508–515) with hull/wing/core colors and HP multiplier
2. Add corresponding attack pattern array to `BOSS_PATTERNS` (line 1372+)
3. Define pattern functions using `fireRadial()`, `fireAimedSpread()`, etc. (lines 1324–1370)
4. Boss index is determined by `(stage - 1) % BOSSES.length`, so placement is automatic

### Add a new sound effect
1. Add function to `sfx` object (line 624+) using `beep()` and `noiseBurst()`
2. `beep(freq, duration, type, volume, opts)`: sine/square/sawtooth/triangle waves with optional sweep, filter, pan, delay
3. `noiseBurst(duration, volume, opts)`: white noise with envelope and filter
4. Call the function where needed in game logic

### Adjust difficulty balance
- Edit `DIFFICULTIES` object (line 485–489)
- Tweak `spawnBase`, `spawnMin`, `enemySpeed`, `enemyShootChance`, `lives`, `bombs`, `bossHpMul`
- Test across all stages to ensure reasonable progression

### Debug game state
- Open browser DevTools Console
- Inspect `player`, `enemies`, `score`, `stage` directly in the IIFE scope (wrapped in try-catch if necessary)
- Temporarily reduce game speed by calling `update()` with `dt = 0.001` for frame-by-frame stepping

## File Structure
- **index.html**: Complete game in one file (~1670 lines)
  - Lines 1–361: HTML structure and CSS
  - Lines 362–420: HTML elements (canvas, HUD, overlay, touch controls)
  - Lines 421–1670: JavaScript game engine

## Testing
There are no automated tests. Verification is manual:
1. **Gameplay**: Play through all difficulty levels and stages
2. **Boss patterns**: Verify each boss cycles through its patterns correctly
3. **Controls**: Test keyboard, mouse, and touch input on target devices
4. **Performance**: Check frame rate on low-end devices (use DevTools throttle)
5. **Edge cases**: Test game over, pause/resume, mute toggle, reduced motion toggle

## Performance Notes
- Canvas resolution is fixed at 480×720 to prevent lag on mobile
- Particle count is capped dynamically based on frame time (quality system)
- Enemies and bullets are removed when off-screen
- No memory leaks expected; all objects are recycled or garbage-collected normally

## Browser Compatibility
- Requires: Canvas 2D API, Web Audio API, localStorage, requestAnimationFrame
- Touch: Detected via `ontouchstart` and `navigator.maxTouchPoints`
- Mobile: Scales to fit viewport; safe area insets respected for notched devices
