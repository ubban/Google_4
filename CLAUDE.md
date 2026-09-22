# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML5 application. It runs entirely in the browser with no build process or external dependencies.

## Running the Game

**Local Testing:**
- Simply open `index.html` in a web browser
- The game is fully self-contained and runs immediately

**Live Server (Optional):**
- For production or testing with CORS-sensitive features, use:
  ```bash
  python3 -m http.server 8000
  # or
  npx http-server
  ```
- Then navigate to `http://localhost:8000`

## Architecture & Game Systems

The entire game logic is contained in a single `index.html` file (~1670 lines) with three sections:

### 1. **HTML Structure & UI**
- Canvas element for game rendering
- HUD overlay with stats (score, best, weapon level, health, bombs)
- Difficulty selector buttons (Easy, Normal, Hard)
- Control buttons for mute, motion settings, and pause
- Accessible elements (sr-live for screen reader announcements)

### 2. **Inline CSS**
- Responsive design using clamp() for scaling across devices
- Dark theme with blue/dark backgrounds
- Animations for:
  - Combo popups and stage banners
  - Boss health bar
  - Vignette effects on critical health
  - Screen flash on hit/bomb
  - Heart and bomb UI animations
- Touch-friendly buttons and safe-area support

### 3. **Inline JavaScript**
The game logic is organized around several key systems:

#### **Game State**
- `player`: Position, velocity, health, invulnerability, weapon type, cooldowns
- `enemies`: Array of enemy ships with type-specific behavior (weave, shoot patterns)
- `boss`: Single boss entity with phases and attack patterns
- `bullets`: Player projectiles with optional pierce/laser properties
- `enemyBullets`: Enemy projectiles
- `powerUps`: Pickups (life restore, bomb, weapon upgrades)
- `particles`: Explosion and visual effect particles
- Game flags: `running`, `paused`, `gameOver`, `bossWarned`

#### **Key Functions**

**Initialization & Game Flow:**
- `resetGame()` - Reset all state for a new game
- `loop(now)` - Main game loop using requestAnimationFrame, handles update and draw

**Player Actions:**
- `fireWeapon()` - Spawn bullets based on current weapon type
- `useBomb()` - Clear enemies and deal heavy damage to boss
- `pickupPowerUp(p)` - Handle power-up collection (health, bomb, weapon upgrades)

**Enemy & Boss Management:**
- `spawnEnemy()` - Randomly select enemy type and spawn at top
- `spawnBoss()` - Trigger boss battle after KILLS_PER_STAGE kills
- `updateBoss(dt)` - Update boss position, attacks, and phases
- `killBoss()` - End boss fight and advance stage

**Collision & Damage:**
- `rectsOverlap(a, b)` - AABB collision detection
- Hit detection runs every frame for player/enemies, enemies/bullets, boss/bullets

**Rendering:**
- `draw()` - Clear canvas and render all entities
- `drawPlane(x, y, w, h, color, dark, drawCross)` - Draw ship sprite
- `drawBossShip(b)` - Draw boss with specific visual style
- `spawnExplosion(x, y, color, count)` - Create particle explosion

**Audio & Feedback:**
- `beep(freq, dur, type, vol, opts)` - Synthesize tones via Web Audio API
- `noiseBurst(dur, vol, opts)` - Synthesize noise (for explosions)
- `panFor(x)` - Calculate stereo pan based on screen position
- Separate sfx objects for different sound effects (vulcan, laser, explode, etc.)

**UI Updates:**
- `renderHearts(count, max)` - Update health display
- `renderBombs(count, max)` - Update bomb count display
- `showComboPop(text)` - Show combo milestone text
- `showStageBanner(num)` - Show stage transition text

#### **Game Balance Constants**
Located at top of script section:
- `KILLS_PER_STAGE` - Enemy kills needed to trigger boss (50)
- `CHAIN_WINDOW` - Time window for combo multiplier (3 seconds)
- `DIFFICULTY_PRESETS` - Per-difficulty settings:
  - `spawnBase`, `spawnMin` - Enemy spawn rates
  - Enemy health, speeds, and attack patterns
  - Boss health and damage values
  - Multiplier and scoring parameters

#### **Enemy Types** (`ENEMY_TYPES`)
- `normal`: Small fast enemies, basic spread shot pattern
- `tank`: Slower, high HP, single target shot, worth more points
- `weaver`: Fast with evasive weaving movement

#### **Weapon System**
- `vulcan`: Rapid-fire regular bullets (fireDelay: 0.15s)
- `laser`: Slower, piercing beam (fireDelay: 0.1s, continues through enemies)
- Weapon levels increase damage and visual size
- Progression via collectible power-ups

#### **Boss System**
- Boss has multiple attack phases
- Movement patterns include entrance animation, sine wave weaving
- Attacks alternate between spread patterns and target-seeking projectiles
- Health bar displayed at top during boss fight
- Defeated when HP <= 0

#### **Input Handling**
- Keyboard: WASD for movement, Space to fire
- Mouse: Aim at cursor position
- Touch: Tap to fire, swipe/drag to move
- Quality setting can be toggled to reduce effects on slower devices

## Development Guidelines

### Code Organization
- All game code is in one file, organized as one IIFE (Immediately Invoked Function Expression)
- State variables are declared at the top for easy access
- Core loop-based systems follow the pattern: update (logic) → draw (render)

### Making Changes
- **Balance Tweaks**: Modify `DIFFICULTY_PRESETS` constants for spawn rates, health, timing
- **Visual Changes**: Edit inline CSS or Canvas drawing code (drawPlane, drawBossShip, etc.)
- **Gameplay Mechanics**: Modify update() and collision detection sections
- **Audio/SFX**: Adjust beep() and noiseBurst() calls, or Web Audio parameters
- **New Features**: Add state to player/enemies/boss objects, then update collision and rendering logic

### Testing Considerations
- No automated tests - test game by playing it in browser
- Test on multiple browsers (Chrome, Firefox, Safari) and mobile
- Verify touch controls work on actual touch devices
- Check performance on lower-end devices (consider quality setting)
- Test all three difficulty levels
- Test boss and combo systems reach their intended states

### Performance
- Canvas quality can be reduced via `quality` variable (0 = no glow effects)
- Reduced motion setting disables screen shake, flash, and pulsing effects
- Particle count can impact performance - adjust explosion particle counts in spawnExplosion()
- Profile with DevTools if frame rate drops on slower devices

## Common Tasks

**Adjusting Difficulty:**
- Edit spawn rates, enemy HP, and damage values in `DIFFICULTY_PRESETS`
- Modify `KILLS_PER_STAGE` to change boss trigger timing

**Adding Sound Effects:**
- Add new beep() or noiseBurst() calls with appropriate frequencies/durations
- Ensure audio is created after user interaction (ensureAudio() handles this)

**Tweaking Weapon Balance:**
- Modify fireDelay and bullet properties (speed, pierce count)
- Adjust weapon level damage multipliers

**Changing Visual Style:**
- Modify CSS color variables and animations
- Adjust drawPlane() and particle colors for enemies/explosions

**Accessibility:**
- Respects reducedMotion flag (toggles screen shake, flash, vignette effects)
- Uses semantic HTML for screen reader support (sr-live element)
- Touch-friendly button sizing with safe-area support
