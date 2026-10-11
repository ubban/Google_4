# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game delivered as a single-file HTML application (`index.html`). The entire game—HTML structure, CSS styling, and JavaScript logic—is contained in one 1669-line file. No build process, npm dependencies, or external libraries are required; the game runs in any modern browser.

## How to Run

1. **Open in browser**: Simply open `index.html` in any modern web browser (Chrome, Firefox, Safari, Edge)
2. **Local development**: Use any simple HTTP server:
   - Python 3: `python -m http.server 8000`
   - Node.js: `npx http-server`
   - Or open the file directly: `file:///path/to/index.html`

## Architecture

The game is organized as a single IIFE (Immediately Invoked Function Expression) enclosing all game logic. The structure is:

### 1. **Game Configuration** (lines 463-500)
   - **Storage keys**: Best score, mute state, difficulty, reduced motion preference
   - **Difficulty presets**: `DIFFICULTIES` object defines Easy/Normal/Hard with spawn rates, enemy behavior, lives, bombs
   - **Themes**: `THEMES` array (6 different sky/weather themes) with color palettes
   - **Game constants**: `KILLS_PER_STAGE = 18` (enemies to defeat per stage)

### 2. **Game State** (established early in IIFE)
   - Player position, health, weapon type/level, bomb count, score, multiplier
   - Current stage, difficulty, whether game is paused
   - Arrays for enemies, projectiles, bombs, explosions, pickups
   - Boss state (current boss, health, attack patterns)

### 3. **Game Loop** (main update/render cycle)
   - Uses `requestAnimationFrame` for smooth 60 FPS
   - **Update phase**: Player movement, weapon firing, enemy AI, collision detection, scoring
   - **Render phase**: Canvas drawing, HUD updates, effect animations
   - **Adaptive performance**: Frame time tracking auto-degrades quality on low-end devices

### 4. **Core Systems**

   **Input Handling**
   - Keyboard: Arrow keys / WASD for movement, Space / X / Shift for weapons/bombs
   - Mouse: Crosshair following (on desktop), click to fire
   - Touch: Left zone for movement (drag joystick), right zone for FIRE/BOMB buttons
   - Pause: P or Esc keys, or pause button (⏸)

   **Player**
   - Centered at bottom of canvas, moves horizontally within bounds
   - Weapon system: VULCAN (default rapid fire) → upgradeable weapon types (different damage/behavior)
   - Weapon levels increase with pickups; each level improves fire rate or spread
   - Health system: 3 lives (configurable by difficulty)
   - Smart bombs: Limited (2-3 per game), detonate all on-screen enemies
   - Invincibility frames after taking damage

   **Enemy System**
   - Spawned in waves based on spawn timer (adaptive to difficulty)
   - Simple patrol + shoot patterns (vertical movement, occasional bullets)
   - Defeated when health reaches 0; drop score pickups (scaling with combo multiplier)
   - Boss spawns after `KILLS_PER_STAGE` enemies defeated in a stage

   **Boss Fights**
   - Boss name and health bar display at top
   - Boss has distinctive movement + attack patterns (varies per boss)
   - 3-5 bosses per game, each progressively harder
   - Health scales with difficulty (`bossHpMul`)

   **Scoring & Progression**
   - Score increases per enemy defeated (10 base × multiplier)
   - Combo multiplier increases with consecutive hits, resets on damage
   - Weapon pickups advance weapon level
   - Stages advance every 18 enemies defeated
   - Each new theme/stage provides visual variety

   **Visual Effects**
   - Canvas-based particle explosions (when enemies die)
   - Screen vignette flash on player damage
   - Combo popup floating text (centered, animated scale)
   - Stage banner on stage transitions
   - Responsive theme backgrounds

### 5. **UI & HUD** (HTML elements controlled by JavaScript)
   - **Top display**: Score, Best Score, Combo multiplier, Weapon type & level, Health hearts, Bomb count
   - **Boss bar**: Shows during boss fights
   - **Overlay**: Start screen with difficulty selector, game-over screen with final stats
   - **Buttons**: Mute (🔊), Reduced Motion (🎆), Pause (⏸)
   - **Touch controls**: Visible only on mobile, positioned left/right at bottom

### 6. **Accessibility**
   - Screen reader announcements via `aria-live="polite"` for important events
   - Reduced motion option (respects `prefers-reduced-motion`, disables animations)
   - Color contrast maintained in all UI elements
   - Focus management on overlay buttons

## Key Code Patterns

**Configuration vs. State**
- Constants like `DIFFICULTIES` and `THEMES` are defined early and never change
- Dynamic state (score, lives, enemies array) is managed in the game loop
- Use `localStorage` for persistence (best score, difficulty preference)

**Canvas Rendering**
- Clear canvas and redraw every frame (no dirty-rect optimization needed at this scale)
- Draw order: background → game objects → effects → HUD
- All drawing through `ctx` (2D canvas context)

**Collision Detection**
- Simple bounding box (AABB) checks between player/enemies/projectiles
- Hitboxes are typically square; irregular shapes use approximate boxes

**Audio (placeholder)**
- Mute button toggles game-wide sound flag; actual sound effects not implemented

## Development Workflow

### Adding a New Feature

1. **New difficulty setting**: Add entry to `DIFFICULTIES` object, test with different spawn/speed/lives values
2. **New weapon type**: Define weapon behavior (spread, damage, fire rate), add to weapon unlock system
3. **New boss**: Create boss class/object with unique movement/attack pattern, add to boss rotation
4. **New visual theme**: Add theme to `THEMES` array with sky colors and cloud tints

### Testing Locally

- Open `index.html` in browser (no server needed, but server recommended to avoid CORS issues)
- Test on multiple screen sizes: desktop, tablet, mobile (Chrome DevTools → responsive mode)
- Test touch controls on actual device or use Chrome DevTools touch emulation
- Test accessibility: keyboard navigation, screen reader output (NVDA, JAWS, VoiceOver)

### Performance Considerations

- The adaptive quality system (`quality` variable) automatically reduces detail on low-end devices
- Particle count, enemy count, and draw complexity decrease as frame time increases
- No external libraries → minimal overhead; most perf bottleneck is canvas rendering

### Browser Compatibility

- Works in all modern browsers (Chrome, Firefox, Safari 11+, Edge)
- Uses standard Canvas 2D API (widely supported)
- Graceful degradation if touch events not supported (falls back to keyboard/mouse)

## Summary of Major Variables/Functions

| Item | Type | Purpose |
|------|------|---------|
| `canvas`, `ctx` | DOM/Context | Canvas element and 2D drawing context |
| `player` | Object | Player state: x, y, health, weapon, bombs, score |
| `enemies` | Array | All active enemies (position, health, type) |
| `projectiles` | Array | All bullets in flight |
| `DIFFICULTIES` | Object | Preset difficulty configurations |
| `THEMES` | Array | Visual theme definitions |
| `gameLoop` | Function | Main update/render cycle |
| `updatePlayer` | Function | Handle input and move player |
| `updateEnemies` | Function | Advance enemy positions, firing, and AI |
| `spawnEnemy` | Function | Create new enemy instance |
| `spawnBoss` | Function | Create boss for current stage |
| `checkCollisions` | Function | Hit detection across all entities |
| `render` | Function | Draw all game objects and HUD |

