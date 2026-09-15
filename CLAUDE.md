# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game implemented as a single HTML5 file with embedded CSS and JavaScript. The game features wave-based enemy encounters, boss battles, multiple weapons, power-ups, and full touch/mobile support.

## Quick Start

**Running the game:**
- Open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
- No build process, dependencies, or server required
- The game launches immediately and is fully playable

**Supported input methods:**
- Mouse: Click and move to control the player ship
- Touch: Tap and drag on mobile/tablet devices
- Touch controls also include an on-screen joystick for absolute positioning (not relative)

## Code Architecture

The entire game logic lives in a single `<script>` block in `index.html`. The code is organized into the following key systems:

### Game State & Global Variables
- Core variables define game state: `gameRunning`, `score`, `health`, `bombs`, `wave`, `difficulty`, `stageClear`
- Entity arrays track: `bullets`, `enemies`, `bosses`, `powerups`, `explosions`, `shakes`, `flashes`
- Canvas and rendering variables: `canvas`, `ctx`, dimensions stored globally

### Input & Controls
- `canvasPointFromEvent()` - Converts DOM coordinates to game canvas coordinates
- `joystickStart/Move/End()` - Virtual joystick for touch controls (absolute positioning, not delta-based)
- Input is processed continuously during `requestAnimationFrame` loop based on state variables

### Rendering & UI
- `drawPlane()` - Renders the player ship with rotation and flame animation
- `drawBossShip()` - Renders boss enemies with health bar
- Main canvas rendering happens in the animation loop
- HUD updates via DOM elements: `#hud`, `#bossBarWrap`, `#overlay`
- Screen effects: vignette flash, hit flash, screen shake, combo pop-ups, stage banners

### Game Loop Structure
- Single `requestAnimationFrame` loop drives the entire game
- Per-frame: Input processing → Physics update → Collision detection → Rendering
- Difficulty modifiers are baked into spawn rates, enemy behavior, and damage values
- Wave progression triggers boss spawns after defeating all regular enemies

### Enemy & Boss Systems
- `spawnEnemy()` - Creates enemies with wave-dependent stats
- `spawnBoss()` - Creates bosses (different boss per stage, health based on wave)
- Enemies follow simple movement patterns and shoot in predetermined directions
- Boss behavior: Moves in loops, shoots projectiles, has attack phases

### Weapon System
- Four weapons cycle: Laser (piercing), Missiles (multiple), Spread Shot, Plasma
- Weapon stats (damage, fire rate, projectile speed) scale with difficulty
- `weaponLabel()` - Returns the current weapon name
- Weapon switching happens via spacebar or UI button

### Power-up System
- `maybeSpawnPowerUp()` - Randomly spawns at enemy destruction locations
- Power-up types: Health restore (❤️), Bomb recharge (💣), Weapon upgrade (⭐), Score multiplier (x)
- Pickup radius is generous to encourage engagement

### Audio System
- `beep()` - Generates basic tone audio via Web Audio API with frequency/duration/waveform options
- `noiseBurst()` - Generates white noise for explosion/hit effects
- `ensureAudio()` - Initializes audio context (required for web audio on first user interaction)
- Audio uses `audioContext` global, panned based on game events
- Muting via `#muteBtn` toggle saves preference to localStorage

### Collision Detection
- `rectsOverlap()` - Axis-aligned bounding box collision
- Collision loops check: player-bullets vs enemies, player vs projectiles, player vs power-ups

### Animation & Effects
- Screen shake via `screenShake(magnitude, duration)` - adds jitter to canvas transform
- Flash effects via `#flash` element with CSS animations
- Combo pop-ups fade and scale during display
- Stage banners show level progression
- Vignette effect pulses during critical health

### Storage & Persistence
- High score stored in `localStorage` under key `skyFighterHighScore`
- Mute state stored in `localStorage` under key `skyFighterMute`

## Difficulty Levels

Three difficulty levels scale various game parameters:

| Level | Enemy Damage | Spawn Rate | Boss Health | Weapon Upgrades |
|-------|-------------|-----------|-------------|-----------------|
| Easy | 0.7x | Slower | Reduced | Faster | 
| Normal | 1.0x | Standard | Normal | Standard |
| Hard | 1.4x | Faster | Increased | Slower |

Difficulty can be changed mid-game via difficulty buttons and immediately affects incoming enemies.

## Extending the Game

### Adding New Weapons
1. Add weapon type constant near the weapon cycle logic
2. Update `weaponLabel()` to return the new weapon name
3. Add firing logic in the main loop's weapon branch
4. Adjust fire rate and damage for game balance

### Adding New Bosses
1. Create a new boss object in `spawnBoss()` with unique properties:
   - Position, dimensions, health
   - Movement pattern (use parametric curves or waypoints)
   - Projectile patterns (angle spread, fire rate)
2. Add rendering logic in `drawBossShip()` if needed for visual differentiation
3. Update boss names in the `bossNames` array (if one exists, or create it)

### Modifying Game Balance
- Enemy spawn rate: Adjust the interval in `spawnEnemy()`
- Damage values: Scale in weapon fire logic and enemy projectile damage
- Health/bombs: Modify starting values in `resetGame()`
- Wave progression: Adjust `enemyCount` or spawn thresholds

## Important Implementation Details

### Canvas Scaling & Responsive Design
- Canvas resizes on window resize via `resize()` function
- Game maintains aspect ratio and centers in viewport
- Safe area insets considered for mobile notches (CSS variables)

### Touch Control Behavior
- Virtual joystick uses **absolute positioning**: tap sets player position directly
- Not delta-based movement (not relative to ship's current position)
- Joystick feedback is visual only; coordinates drive ship position each frame

### Performance Considerations
- All rendering is 2D canvas (no WebGL or libraries)
- Particle effects are minimal; simple shapes and opacity transitions
- No asset loading (no images or audio files) — all generated programmatically
- Frame timing tracked with `trackPerf(dt)` for debugging

### Wave/Stage Progression
- Each wave spawns a set number of enemies (`enemyCount` variable, increases over time)
- After defeating all enemies in a wave, boss appears
- After defeating boss, new stage begins with higher stats
- No hard level cap; game continues indefinitely with increasing difficulty

## Common Development Tasks

### Adjusting game difficulty
Modify the `difficulty` variable and its effect on spawn rates, damage, and health in the main loop.

### Adding visual effects
Use the existing effect framework: screen shake, flashes, vignette, combo pop-ups, or add new DOM element animations.

### Debugging
- Open browser DevTools (F12)
- Check console for any errors
- Use `trackPerf()` calls to monitor frame timing
- Inspect `gameState` variables in console (all are global)

### Testing specific scenarios
- Set `gameRunning = true` in console to start without menu
- Directly set `wave`, `health`, `bombs` to test specific game states
- Modify `difficulty` on the fly to test balance changes

## Audio & Accessibility

- All game audio is optional; game fully playable muted
- Haptic feedback not implemented (potential mobile enhancement)
- Color contrast ratios meet accessibility standards for UI text
- Game state communicated through visual feedback (not audio-only)

## Browser Compatibility

Tested and working on:
- Chrome/Edge (latest)
- Firefox (latest)
- Safari 12+
- Mobile browsers (iOS Safari, Chrome Mobile, Samsung Internet)

Requires:
- ES6 JavaScript support
- Canvas 2D API
- Web Audio API (for sound)
- LocalStorage (for persistence)
- Touch Events API (for mobile)

## Files

- `index.html` - Complete game (HTML, CSS, JavaScript)
- `README.md` - Minimal project description
- `CLAUDE.md` - This file

No separate CSS or JS files; everything is embedded in the HTML for a single deployable asset.
