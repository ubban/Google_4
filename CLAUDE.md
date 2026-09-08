# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. It features:
- Canvas-based 2D rendering
- Procedurally generated WebAudio sound effects (no external assets)
- Multiple difficulty levels (Easy, Normal, Hard)
- Progressive stages with different visual themes
- Boss encounters
- Touch and keyboard/mouse input support
- Responsive design with adaptive performance

The entire game is contained in `index.html` (~1700 lines, 58KB).

## Development

### Running the Game

Simply open `index.html` in a modern web browser (Chrome, Firefox, Safari, Edge). The game:
- Runs entirely in the browser with no build step or server needed
- Saves state to localStorage (best score, muted status, difficulty, motion preferences)
- Detects touch devices and enables touch controls automatically
- Adapts performance (quality variable, lines 474-482) when frame times degrade

### Testing Locally

1. **Desktop**: Use Chrome DevTools (F12) to open the browser console and test game states
2. **Mobile**: Use Chrome DevTools device emulation (Ctrl+Shift+M) or test on an actual device
3. **Performance**: Open the console and check frame times; the game auto-adjusts quality (full → medium → low) if needed

### Key Browser Features Used

- Canvas 2D context for rendering
- Web Audio API for sound synthesis (including dynamics compression)
- localStorage for persistence
- Touch Events API for mobile input
- matchMedia for reduced-motion preference detection
- requestAnimationFrame for game loop

## Code Architecture

The single HTML file is organized in this order:
1. **HTML head** (lines 1-359): Meta tags, styles, favicon
2. **HTML body** (lines 360-420): Game container, canvas, HUD, overlay UI, touch controls
3. **JavaScript** (lines 422-end): All game logic in an IIFE

### JavaScript Structure

**Setup & Constants** (lines 423-530)
- DOM element references
- Storage keys for localStorage
- Difficulty presets (spawn rates, enemy speed, lives, bombs, HP multipliers)
- Themes (6 different sky/cloud/sun color combinations)
- Enemy types (grunt, interceptor, tank with different HP/speed/scoring)
- Boss definitions (6 bosses with unique colors and HP multipliers)

**Audio System** (lines 542-676)
- WebAudio synth with optional muting
- `beep(freq, duration, type, volume, opts)`: Play tones with optional frequency sweep and filters
- `noiseBurst(duration, volume, opts)`: Play noise (used for explosions)
- `sfx` object: Named sound effects (shoot, laser, explode*, hit, bomb, powerup, boss*, stage*, gameover, uiClick)
- All sounds generated on-the-fly; no external audio files

**UI & Rendering** (lines 680-730)
- Heart display for lives (includes "critical" pulse when at 1 HP)
- Bomb count display
- Screen shake effect (magnitude & duration)
- Combo pop animation (triggered on kill chains)
- Stage banner animation
- Vignette effect (red pulse when critical health)
- Flash effects (white flash on hit, larger on bomb)

**Input Handling** (lines 752-861)
- **Keyboard**: Arrow keys / WASD to move, Space to fire, X / Shift to bomb, P / Esc to pause
- **Mouse**: Move to steer, click to fire (firing stops on mouseup anywhere)
- **Touch**: Left 55% of screen = movement (drag-based joystick), right 45% = FIRE/BOMB buttons
- Input detection automatically enables/disables touch controls based on device type
- Joystick visual feedback (base + stick) only shown on touch devices

**Game State** (lines 863-898)
- Player object: position, speed, weapon type/level, cooldown, invulnerability timer
- Bullets, enemies, enemy bullets, particles, power-ups (all arrays)
- Score, lives, maxLives, running, paused, gameOver flags
- Bomb count, spawn timer, frame counter
- Stage tracking, kill counter, chain multiplier system

**Game Loop & Core Logic** (lines 900-end)
- `resetGame()`: Initialize all game state when starting a new game
- `update(dt)`: Called each frame to update all entities
  - Player movement (keyboard, joystick, or mouse-driven)
  - Enemy spawning (rate decreases as stage progresses)
  - Weapon cooldown and firing
  - Particle updates and removal
  - Collision detection (player/enemy, bullets/enemies, player/power-ups)
  - Chain multiplier decay
  - Boss HP updates
- `render()`: Called each frame to draw everything
  - Sky background (gradient)
  - Clouds
  - Enemies and bosses
  - Bullets and particles
  - Player
  - Boss health bar (when boss active)
  - Screen shake offset applied to canvas

**Key Gameplay Systems**

1. **Weapon System** (player.weapon): Two types (vulcan, laser) with levels (1-5+)
   - Different fire patterns and damage
   - Power-ups increase weapon level or switch weapon
   - Rapid-fire timer extends when weapon upgraded

2. **Enemy Spawning** (spawnTimer): Difficulty affects spawn rate
   - `spawnBase`: Base spawn interval
   - `spawnMin`: Minimum interval (harder difficulties spawn faster)
   - Enemy types scale by stage (tank only appears stage 2+)
   - Boss appears after `KILLS_PER_STAGE` kills (18)

3. **Scoring & Multiplier**
   - `chain`: Tracks consecutive kills
   - `multiplier`: Increases with chain length, decays over time
   - Score = (enemy base points × multiplier)
   - Chain display triggers at certain thresholds

4. **Boss System**
   - 6 different boss types with unique visuals
   - Selectable coloring (hullTop, hullBottom, wing, core)
   - HP multiplier per difficulty
   - Boss health bar shown at top
   - Boss defeated → stage advances → difficulty ramps

5. **Power-Ups**
   - Dropped when enemies die
   - Types: weapon upgrade (switch or level up), bomb recovery
   - Randomly positioned within screen bounds

6. **Adaptive Performance** (lines 474-482)
   - Tracks average frame time over 45 frames
   - Quality levels: 0 (full) → 1 (medium) → 2 (low)
   - Adjusts rendering detail to maintain 60 FPS

## Modifying the Game

### Adding a New Enemy Type

1. Add entry to `ENEMY_TYPES` object (line ~502), specifying: width, height, HP, color, score value, minimum stage, and optional `weave` movement
2. Update the spawning logic if the enemy has special behavior
3. Add explosion sound to `EXPLODE_SFX` mapping (line ~678) if needed

### Adjusting Difficulty

Edit `DIFFICULTIES` object (lines 485-489):
- `spawnBase`: Higher = slower spawn, lower = faster spawn
- `enemySpeed`: Multiplier on enemy movement speed
- `enemyShootChance`: Probability enemy shoots each frame
- `bossHpMul`: Multiplier on boss HP

### Adding a New Theme

Add to `THEMES` array (lines 493-500):
- `sky`: Array of 3 gradient colors [top, middle, bottom]
- `cloud`: RGB components for cloud color
- `sun`: RGB components for sun circle color
- `name`: Display name

### Changing Visual Polish

- Stage banner text: Line ~719, modify `showStageBanner()`
- Combo pop styling: CSS lines ~105-126, or modify `showComboPop()`
- Boss bar styling: CSS lines ~149-179
- Screen shake intensity: Line ~728, adjust `shakeMag` calculations

## Performance Considerations

- **Rendering**: Canvas 2D is used (not WebGL), suitable for the 480×720 resolution
- **Collision Detection**: Simple AABB (axis-aligned bounding box) checks
- **Audio**: WebAudio buffers are created on-demand and garbage collected naturally
- **Touch Events**: Use `preventDefault()` to avoid scroll/zoom interference (lines 814-833)
- **Frame Rate**: Game targets 60 FPS; adaptive quality kicks in if average frame time > 1/40s

## Browser Compatibility

Works on modern browsers supporting:
- Canvas 2D context
- Web Audio API (or webkit prefix fallback)
- Touch Events
- ES6+ (template literals, arrow functions, destructuring)
- localStorage

Tested on: Chrome/Edge 90+, Firefox 88+, Safari 14+, mobile browsers (iOS/Android).
