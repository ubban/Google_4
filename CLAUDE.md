# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a browser-based Raiden-style vertical shoot-'em-up game. It's a single-file HTML application with embedded CSS and JavaScript—no build process, no dependencies, fully playable in any modern browser.

## Running the Game

The game runs directly in any modern browser:
- **Local development**: Open `index.html` directly in a browser (file:// protocol works fine)
- **Web server**: Serve `index.html` via HTTP (recommended for testing on mobile/touch devices)
- **Quick HTTP server**: `python3 -m http.server 8000` then visit `http://localhost:8000/index.html`

No build step, compilation, linting, or testing frameworks are required. The game is a self-contained artifact.

## Code Architecture

The entire game is implemented in a single `index.html` file (~1700 lines) structured as follows:

### 1. **HTML Structure** (lines 1–421)
- Canvas element (`#gameCanvas`) for rendering
- UI overlays for title screen, pause, game-over, and HUD
- Responsive layout with mobile-friendly design
- Accessibility features (screen reader announcements, `aria-live` region)
- Touch control zones for mobile (joystick, fire button, bomb button)

### 2. **CSS Styling** (lines ~10–380)
- Dark theme optimized for space shooter aesthetic
- Responsive design with `clamp()` for fluid scaling
- Animations for UI feedback (combo pop, stage banner, heart beat, screen flash)
- Glassmorphism effects with backdrop blur and semi-transparent overlays
- Safe area insets for notched devices

### 3. **JavaScript Game Engine** (lines 422–end, ~1250 lines)
Wrapped in an IIFE for encapsulation. Main sections:

#### **Configuration & Constants** (lines 463–525)
- `DIFFICULTIES`: Spawn rates, enemy behavior, lives, bombs per difficulty
- `THEMES`: 6 visual themes with sky gradients and color palettes
- `ENEMY_TYPES`: Grunt, Interceptor, Tank with HP, speed, score values
- `BOSSES`: 6 named boss types with unique color schemes and HP multipliers
- `KILLS_PER_STAGE`: Enemy count threshold to advance to next stage

#### **Audio System** (lines 543–677)
- Web Audio API synthesis (no external audio files)
- `ensureAudio()`: Lazy initialization, handles suspended AudioContext
- `beep()`: Oscillator-based tones with sweep, filter, pan, and delay options
- `noiseBurst()`: White noise for explosion effects
- `sfx` object: Pre-configured sound effects (shoot, powerup, explode, boss hit, game over, etc.)

#### **Input Handling** (lines 768–839)
- Keyboard input tracking via keydown/keyup
- **Joystick (touch)**: `joystickStart/Move/End()` calculate normalized stick vector (`joyVec`)
- **Fire/Bomb buttons (touch)**: Handled via touch events, map to same keys as keyboard

#### **Game State Management** (lines 841–1147)
- Player: position, HP, weapon type/level, bombs, score, combo chain
- Enemies: array of active enemies, each with position, HP, type, AI state
- Bullets: player bullets and enemy bullets in separate arrays
- Explosions: particle effects with color, lifetime
- Power-ups: dropped by destroyed tanks, auto-collect when near

Key functions:
- `resetGame()`: Initialize new run, clear all entities, reset UI
- `fireWeapon()`: Trigger active weapon pattern (depends on weapon type and level)
- `applyPowerUp(type)`: Handle weapon/health/bomb pickups
- `useBomb()`: Trigger smart bomb (clears enemies, creates effect)
- `addScore(base)`: Increment score with multiplier, update best score
- `breakChain()`: Reset combo multiplier when player is hit

#### **Game Physics & Logic** (lines 1148–1410)
- `update(dt)`: Main update loop
  - Player movement (keyboard or joystick)
  - Bullet updates (position, collision with enemies)
  - Enemy AI (random movement, shooting patterns, stage progression)
  - Power-up physics (drift downward, collect on overlap)
  - Explosions (particle lifetime)
  - Screen shake, visual effects
  - Weapon fire rate tracking

#### **Boss System** (lines 1043–1456)
- 6 named bosses, each with unique visual design and attack pattern
- `spawnBoss()`: Triggered when players reach a certain kill count
- `updateBoss(dt)`: Boss AI with sinusoidal movement, coordinated attack patterns
  - Boss patterns: radial spread, aimed spread, columns, spiral burst, cross, homing missiles, double rings
  - `fireRadial/fireAimedSpread/fireColumns/etc()`: Each implements a distinct attack
- `killBoss()`: Trigger next stage, play sound, update UI

#### **Rendering** (lines 1507–1639)
- `draw()`: Complete frame redraw
  - Background: Layered gradient sky with animated clouds
  - Player plane: `drawPlane()` with 3D-like shading, flame animation
  - Enemy ships: Rendered based on type with color/dark shading
  - Boss ships: `drawBossShip()` with layered hull, wings, glowing core
  - Bullets: Simple circles (player blue, enemies orange/red)
  - Explosions: Colorful particles with fading opacity
  - HUD: Score, best score, weapon name/level, hearts, bombs, boss health bar
  - Overlays: Title screen, game-over stats, pause screen

#### **Game Loop** (lines 1640–1670)
- `loop(now)`: RequestAnimationFrame callback
  - Calculate delta time, cap at 33ms (30 FPS buffer)
  - Adaptive performance system: downgrade from full → medium → low quality based on frame times
  - Pause handling
  - Call `update(dt)` and `draw()`

### 4. **Key Game Mechanics**

**Weapon System:**
- Starts at Vulcan (rapid-fire)
- 6 weapon types with escalating power and bullet patterns
- Collected via tank powerups; levels up by collecting same type again
- Weapon level affects fire rate and projectile behavior

**Difficulty Settings:**
- **Easy**: Slower spawns, slower enemies, lower accuracy, more lives/bombs, boss 20% weaker
- **Normal**: Baseline difficulty
- **Hard**: Fast spawns, fast enemies, higher accuracy, boss 30% stronger

**Combo System:**
- Multiplier (x1 → x99) increases score per kill
- Breaks when player takes damage
- Resets each new stage

**Adaptive Performance:**
- Tracks average frame time over 45 frames
- Downgrades quality (full → medium → low) if avg > 25ms
- Upgrades if avg < 17.8ms
- Quality setting affects enemy spawn rate and visual detail

**Accessibility:**
- Screen reader support via `srLive` announcements
- Reduced motion preference respected (screen shake, flash, pulse animations)
- Touch controls with large tap targets
- High contrast UI
- Pause/resume at any time

**Local Storage:**
- Best score persisted across sessions
- Difficulty preference
- Mute state
- Reduced motion preference

## Development Notes

### Adding New Content

**New Enemy Type:**
1. Add entry to `ENEMY_TYPES` object with `w`, `h`, `hp`, `speedMul`, `color`, `dark`, `score`, `minStage`
2. Update `spawnEnemy()` to weight the new type
3. Optionally update `drawPlane()` or add special rendering in `draw()` if shape differs

**New Boss:**
1. Add entry to `BOSSES` array (name, colors, hpMul)
2. Add new fire pattern function: `fireNewPattern(b, speed)` (follow existing patterns)
3. Update `updateBoss()` to call new pattern when boss is at certain HP threshold
4. Adjust attack timing in the sequential pattern calls

**New Visual Theme:**
1. Add entry to `THEMES` with `sky` (3-color gradient stops), `cloud`, `sun` colors (RGB strings), and `name`
2. Reference in `draw()` to apply gradient on first frame and clouds

**New Audio Effect:**
1. Add function using `beep()` or `noiseBurst()` for synthesis
2. Add entry to `sfx` object with frequency, duration, type, volume, and options
3. Call in gameplay code where sound should trigger

### Modifying Game Balance

- **Spawn rates & difficulty**: Edit `DIFFICULTIES` object values
- **Enemy HP/speed**: Edit `ENEMY_TYPES` values
- **Boss HP**: Adjust `BOSSES[i].hpMul` or overall `bossHpMul` in difficulty
- **Stage progression**: Change `KILLS_PER_STAGE` constant
- **Weapon balance**: Modify bullet counts, spread angles, or speeds in fire pattern functions

### Performance Optimization

- Adaptive quality system already handles frame drops
- `quality` variable: 0=full, 1=medium (skip every other enemy spawn), 2=low (quarter spawn rate)
- Canvas size adapts to viewport via `resize()` function, applies aspect-ratio scaling
- Explosion pool is cleared each frame; no memory leaks from stale particles

### Common Tasks

**Test a boss fight:** Set `KILLS_PER_STAGE = 1` to spawn boss on first enemy kill, then play

**Disable audio:** Set `muted = true` at startup (line 518) or comment out `ensureAudio()` calls

**Force reduced motion:** Set `reducedMotion = true` to disable screen shake, flash, and animations

**Change starting weapon:** Modify `weaponLvl` and `weapon` assignment in `resetGame()`

**Adjust canvas size:** Edit `BASE_W` and `BASE_H` constants (currently 480×720); aspect ratio enforced by CSS

## Browser Compatibility

- Requires ES6 (arrow functions, template literals, const/let)
- Uses Canvas 2D API (widely supported)
- Web Audio API (some restrictions on iOS; play audio only after user gesture)
- Touch Events API and Pointer Events
- LocalStorage for persistence
- CSS Grid and Flexbox for layout

Target: Modern browsers (Chrome 51+, Firefox 54+, Safari 10+, Edge 15+)

## Testing

No automated tests. Verify changes by:
1. Opening the game in a browser
2. Testing across difficulties
3. Checking mobile/touch controls on actual device or Chrome DevTools device emulation
4. Listening for audio effects (if modifying sound)
5. Verifying accessibility with a screen reader or keyboard-only navigation

## File Overview

- **index.html** (1,669 lines): Complete game engine
- **README.md**: Minimal timestamp placeholder
- **.git/**: Standard git history

All game code is in `index.html`. No external files or dependencies.
