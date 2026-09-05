# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sky Fighter is a Raiden-style vertical shoot-'em-up game built entirely in vanilla HTML5, CSS, and JavaScript as a single-file application. The game features procedurally-generated sound using the WebAudio API (no external audio assets), responsive canvas rendering, and comprehensive touch controls for mobile devices.

## Project Structure

The entire game is contained in a single `index.html` file (~1670 lines):
- **Lines 1-421**: HTML document structure and CSS styling
  - HUD elements (score, health, bombs, weapon display)
  - Overlay screens (main menu, game over, pause)
  - Canvas container and touch control zones
  - Responsive design using CSS clamp() and viewport-relative units
- **Lines 422-end**: JavaScript game engine (IIFE pattern for scope isolation)
  - Initialization and DOM references
  - Audio synthesis system (beep(), noiseBurst(), sfx object)
  - Game state management
  - Input handling (keyboard, mouse, touch with virtual joystick)
  - Entity management and rendering (player, enemies, bullets, particles, boss)
  - Game loop using requestAnimationFrame

## Key Architecture Concepts

### Game Loop & Timing
- Uses `requestAnimationFrame` for 60 FPS target
- Delta-time (dt) based updates for frame-rate independent movement
- Frame counter tracks total elapsed frames for animations
- Adaptive quality system reduces visual effects if frame times exceed thresholds (quality 0=full, 1=medium, 2=low)

### Entity System
Game entities are plain JavaScript objects:
- **Player**: Position (x, y), dimensions, weapon state, cooldowns, invulnerability timer
- **Enemies**: Spawned with type (grunt/interceptor/tank), pattern (aimed/spread), HP, weaving behavior
- **Boss**: Stages-specific design, HP tracking, telegraph pattern, movement
- **Bullets**: Player and enemy projectiles with velocity
- **Particles**: Explosion effects with fade-out animation
- **PowerUps**: Weapon upgrades and health restores

### Audio System
Procedural sound synthesis using WebAudio API (no external files):
- `beep(freq, duration, type, volume, opts)`: Oscillator-based tones with optional sweep, filter, panning
- `noiseBurst(duration, volume, opts)`: Filtered noise for explosions and impacts
- Stereo panning based on entity X position (`panFor()`)
- Master gain with dynamics compressor for consistency
- Lazy initialization (`ensureAudio()`) triggered by first user interaction

### Input Handling
Three input methods coexist with priority:
1. **Keyboard/Joystick** (highest priority when active)
2. **Mouse** (glides toward pointer, lower priority)
3. **Touch** (virtual joystick for mobile, fire button)

Key bindings:
- `Space` or left-click: Fire
- `X`, `Shift`: Use bomb
- `P` or `Escape`: Pause
- Arrow keys or WASD: Movement (keyboard only)

### Rendering
Canvas 2D context immediate-mode rendering:
- Base canvas: 480×720 (BASE_W, BASE_H)
- Dynamic scaling up to 1.6× for high-res displays
- Draws in order: background gradient → entities → HUD → overlays
- Utilizes `ctx.save()`/`ctx.restore()` for transformation isolation
- Quality-dependent shadow and glow effects

## Game Progression

### Difficulty Levels
Stored in `DIFFICULTIES` object with parameters for:
- Enemy spawn rates (spawnBase, spawnMin)
- Enemy speed multiplier and shoot chance
- Initial lives and bombs
- Boss HP multiplier

Current difficulties: easy, normal, hard

### Stage System
- 18 kills per stage (KILLS_PER_STAGE = 18)
- Stages 1–6 cycle through 6 different boss designs
- Boss HP increases with stage and difficulty
- Visual themes (sky gradients, cloud colors) change per stage

### Scoring & Multipliers
- Base score: 10–35 points per enemy (type dependent)
- Combo chain: Multiplier increases for consecutive kills within 1.5 seconds
- Multiplier persists within a chain, resets when chain expires
- New high score saved to `localStorage`

## Data Persistence

Uses `localStorage` with four keys:
- `skyfighter.bestScore`: High score (number)
- `skyfighter.difficulty`: Selected difficulty (string: 'easy'/'normal'/'hard')
- `skyfighter.muted`: Audio mute state (0 or 1)
- `skyfighter.reducedMotion`: Reduced motion preference (0 or 1)

## Development & Testing

### Local Testing
1. Open `index.html` directly in a modern browser (Chrome, Firefox, Safari, Edge)
2. Game starts with main menu overlay
3. Select difficulty → press Start Game

### Testing Controls
- **Keyboard**: Arrow keys to move, Space to fire, X/Shift for bomb, P/Esc to pause
- **Mouse**: Move pointer to steer, click to fire
- **Touch**: Virtual joystick in lower-left, fire button lower-right, bomb button upper-right

### Testing Difficulty Progression
- Easy: 4 lives, slower enemies, lower spawn rate
- Normal: 3 lives, moderate challenge
- Hard: 3 lives, dense enemy waves, fast tanks

### Visual Testing
- Resize window to test responsive scaling
- Test with reduced motion preference enabled (Settings → Accessibility)
- Verify theme colors change across stages 1–6
- Check that shadows/glows adapt based on adaptive quality system

### Audio Testing
- Verify procedural sounds play (no console errors from AudioContext)
- Test mute button toggles audio without errors
- Verify spatial panning (explosions left/right match screen position)

## Common Modifications

### Add New Enemy Type
1. Add entry to `ENEMY_TYPES` with properties: w, h, hp, speedMul, color, dark, score, minStage, weave
2. Add explosion sound to `sfx` object if unique behavior needed
3. Add to `EXPLODE_SFX` mapping if custom explosion required
4. Update spawn logic in `spawnEnemy()` if special behavior desired

### Add New Boss Design
1. Add entry to `BOSSES` array with name, colors (hullTop, hullBottom, wing, core), hpMul
2. Boss appearance is drawn by `drawBossShip()` using these colors
3. Boss attack patterns defined in main game loop where boss state is checked

### Adjust Game Balance
- Enemy spawn rates: Modify `DIFFICULTIES[difficulty].spawnBase/spawnMin`
- Enemy difficulty: Change `enemySpeed`, `enemyShootChance`, `hp` multipliers
- Boss difficulty: Modify `bossHpMul` or individual boss `hpMul` values
- Player weapon balance: Change cooldown in `resetGame()` or bullet velocity in firing code

### Add Accessibility Features
- Reduced motion already handled: check `reducedMotion` flag before animations/effects
- Screen reader support: Use `announce()` function to update ARIA live region (`srLive`)
- Ensure theme contrast meets WCAG standards if modifying color palette

## Notes on Code Style

- Uses strict mode and an IIFE for scope isolation
- No external dependencies (no libraries, frameworks, or asset files)
- Event handling uses both legacy listeners and modern ones (e.g., touchstart with passive false)
- Respects `prefers-reduced-motion` media query for motion sensitivity
- Safe-area insets used for notch-aware mobile layouts
- Audio context requires user interaction before playback (modern browser security)
