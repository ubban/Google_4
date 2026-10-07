# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 canvas game (index.html, 1669 lines). The game runs entirely in the browser with no build process required.

## Quick Start

### Running the Game Locally

```bash
# Serve the file locally using any HTTP server
python -m http.server 8000
# or
npx http-server
# or
npx serve
```

Then open `http://localhost:8000` in your browser. The game will load immediately.

### Testing in Production

Simply commit and push to the designated branch—the HTML file can be served directly from any HTTP server or deployed as a static asset.

## Code Architecture

The entire game is contained in a single IIFE (Immediately Invoked Function Expression) that starts at line 423. The code is organized into logical sections:

### Core Game Loop (lines ~1640–1660)
- `gameLoop()`: Main animation loop using `requestAnimationFrame`, calls `update(dt)` and `draw()` each frame
- `update(dt)`: Updates all game state (player, enemies, bullets, bosses, camera shake)
- `draw()`: Renders all game elements to the canvas

### Game State & Systems

**Player System:**
- `player` object: position, velocity, health, bombs, current weapon, animation frame
- `updatePlayer(dt)`: Handles movement, weapon fire, bomb drops
- `drawPlane()`: Renders player and enemy ships using canvas primitives (no sprites)

**Weapon System (lines ~485–500):**
- Defined in `DIFFICULTIES` object with `weapons` array
- Each weapon has: `name`, `cooldown`, `bulletCount`, `spread`, `damage`, `color`
- Weapon upgrades increase level, improving cooldown and damage
- Cycle through weapons with keyboard/touch

**Enemy System:**
- `ENEMY_TYPES` object defines 3 enemy types: `grunt`, `interceptor`, `tank`
- Each has: `health`, `speed`, `fireRate`, `bulletDamage`, `color`, `dark` (for shading), `points` (score value)
- Enemy spawning driven by `STAGE_TEMPLATES` (line ~1090+) which define enemy waves per stage
- `updateBoss(dt)`: Boss spawns after final enemy wave in each stage

**Difficulty System (lines ~485–500):**
- Three difficulties: `easy`, `normal`, `hard`
- Affects enemy stats, spawn rates, player starting health
- Stored in localStorage, persists across sessions

**Audio System (lines ~545–680):**
- No audio files—all sound effects generated with Web Audio API
- `beep()`: Sine/square wave tones (used for weapon fire, UI)
- `noiseBurst()`: Brownian noise (used for explosions)
- `sfx` object: Pre-configured sound effects for all events (firing, explosions, UI, etc.)
- Audio disabled when `muted` flag is true (controllable via mute button)

**UI & HUD:**
- Score, best score, weapon name/level, health hearts, bomb count displayed via DOM elements
- Boss health bar appears only during boss battles
- Stage banners and combo notifications use CSS animations
- Screen shake, flash effects, and reduced-motion support for accessibility

**Input System:**
- Desktop: Arrow keys + Z for weapon cycle + X for bomb
- Touch: Dual-stick layout (left for movement, right for aiming/fire)
- Touch aim affects bullet spread and direction

### Key Functions by Responsibility

**Drawing & Rendering:**
- `drawPlane(x, y, w, h, color, darkColor, flame)`: Draws procedural ship
- `drawBossShip(boss)`: Draws boss with animated components
- `drawClouds(speed, fillColor, rx, ry)`: Parallax background clouds
- `draw()`: Main render pass

**Game Logic:**
- `update(dt)`: Main update loop
- `updatePlayer(dt)`: Player movement and weapon logic
- `updateBoss(dt)`: Boss AI and attacks
- `updateEnemy(en, dt)`: Individual enemy updates

**Audio:**
- `beep(freq, dur, type, vol, opts)`: Generates tones
- `noiseBurst(dur, vol, opts)`: Generates noise bursts
- `ensureAudio()`: Initializes Web Audio context (required for user interaction)

**UI/UX:**
- `showComboPop(text)`: Displays combo multiplier popup
- `showStageBanner(text)`: Displays stage transition banner
- `screenShake(mag, dur)`: Triggers camera shake animation
- `flashHit()`, `flashBomb()`: Screen flash effects

## Data Structures

### Player Object
```javascript
{
  x, y,           // Position
  w, h,           // Width/height
  vx, vy,         // Velocity
  health, maxHealth,
  bombs, maxBombs,
  weapon: {idx, lvl, fireCount},  // Current weapon index, level, and fire count
  anim            // Animation frame for thrust effect
}
```

### Enemy Object
```javascript
{
  x, y,           // Position
  vx, vy,         // Velocity
  health,
  w, h,           // Dimensions
  type,           // 'grunt', 'interceptor', 'tank'
  fireTimer,      // Countdown to next shot
  color, dark,    // Rendering colors
  anim            // Animation frame
}
```

### Boss Object (when active)
```javascript
{
  x, y,           // Position
  w, h,           // Dimensions
  health, maxHealth,
  pattern,        // Current attack pattern index
  patternTimer,   // Countdown to pattern change
  fireTimer,      // Countdown to next shot
  moveTimer,      // Countdown to next position change
  attackX, attackY  // For homing attack patterns
}
```

## Common Development Tasks

### Adding a New Weapon Type
1. Add entry to the `weapons` array in `DIFFICULTIES` (line ~490)
2. Increase weapon index if adding a new slot
3. Define bullet spread, count, damage, and visual color
4. Weapon will automatically be cycle-able via UI

### Tweaking Difficulty
- Edit `DIFFICULTIES` object (lines ~485–500)
- Adjust `weapons` array (cooldown, damage, bullet count)
- Modify enemy stats in `ENEMY_TYPES` (lines ~502–527)
- Adjust spawn rate by editing `STAGE_TEMPLATES` (lines ~1090+)

### Adding Enemy Waves
- Edit `STAGE_TEMPLATES` object to define wave sequences
- Each wave specifies: enemy type, count, delay before spawning
- Boss spawns automatically after the final wave (hardcoded in update logic)

### Modifying Damage/Health
- Player: `player.maxHealth` in game initialization
- Enemies: `health` property in `ENEMY_TYPES`
- Boss: `boss.maxHealth = 300` (or custom per stage)

### Audio/Sound Effects
- Pre-configured SFX in `sfx` object (lines ~624–677)
- To add new sound: define a new entry with `beep()` or `noiseBurst()` call
- Mute state is checked before playing any sound

### Visual Tweaks
- Colors defined in `theme` object (line ~1085+)
- Ship rendering logic in `drawPlane()` and `drawBossShip()`
- Canvas operations use 2D context drawing primitives (no asset files)

## Performance Notes

- Single canvas element with 2D context
- No sprite sheets or image assets—all procedural rendering
- Touch controls use event listeners on canvas; desktop uses keyboard listeners
- Screen shake implemented via canvas transform (applied each frame, not permanent DOM changes)
- Reduced-motion CSS class applied to body; affects animation durations and effect magnitudes

## Browser Compatibility

- Requires HTML5 Canvas API
- Web Audio API for sound effects
- LocalStorage for save data (difficulty, mute state, high score)
- Touch events for mobile support
- requestAnimationFrame for animation loop

## File Organization

- Single file: `index.html` (index.html)
  - Head: CSS styles (viewport, layout, animations, themes)
  - Body: Canvas, HUD elements, overlay menus, touch control zones
  - Script: Game logic (IIFE starting at line 423)

## Testing & Iteration

When testing changes:
1. Reload the browser (hard refresh if needed to clear cache)
2. Check browser console for any errors
3. Test on both desktop and mobile (or emulate with device tools)
4. Test difficulty selection in menu
5. Test mute toggle and reduced-motion accessibility option
6. Verify high score persistence across page reloads (check localStorage)
