# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single HTML file with embedded CSS and JavaScript. It runs directly in the browser with no build process or external dependencies. The game features multiple difficulty levels, weapon upgrades, boss encounters, and accessibility support.

## Running the Game

Simply open `index.html` in a web browser. No build step, server, or dependencies required. The game runs immediately and stores local state (best score, volume preference, difficulty) in browser localStorage.

## Architecture

The entire game is contained in a single `index.html` file (1669 lines) with three main sections:

### 1. DOM Structure & Styling (Lines 1-360)
- **Game canvas**: HTML5 Canvas element where all game rendering occurs
- **HUD elements**: Score, best score, multiplier, weapon status, lives, bombs (absolute positioned overlays)
- **UI overlays**: Start screen, pause menu, game over screen with results
- **Touch controls**: Virtual joystick and buttons for mobile (dynamically shown/hidden)
- **Visual effects**: Vignette, flash effects, combo popups, stage banners, boss health bar
- **Accessibility**: Screen reader region (`#srLive`), reduced motion support via CSS animations

### 2. Game Logic (Lines 422+)
All game logic is in a single IIFE. Key systems:

#### Core State & Loop
- Canvas setup with responsive sizing
- 60 FPS game loop via `requestAnimationFrame`
- Adaptive performance tracking (quality levels 0-2) based on frame time averaging
- Paused state management

#### Game Configuration
- **DIFFICULTIES**: Three presets (easy/normal/hard) controlling spawn rates, enemy speed, shoot chance, lives, bombs, boss HP
- **THEMES**: Six color themes for different stages (Daylight, Sunset, Night, Storm, Arctic, Volcanic)
- **KILLS_PER_STAGE**: 18 enemies per stage before boss spawn

#### Entity Systems
Game entities are stored in simple arrays: `player`, `enemies`, `enemyShots`, `playerShots`, `pickups`, `particles`

**Player object**:
- Position (x, y), velocity
- Lives, bombs (collected from pickups)
- Weapon state: type (VULCAN, LASER, GATLING, MISSILE), level (1-5)
- Invincibility timer after taking damage
- Size collision box (12px radius)

**Enemy object**:
- Position, velocity
- Health and score value
- Shoot interval counter
- Collision box
- Boss flag for stage-end encounters

**Pickups**:
- Weapon upgrades (spawn when enemies die)
- Ammo type determines weapon granted
- Auto-collect within proximity range

**Particles**: Visual effects (explosions, projectile trails)

#### Core Game Loop Order
1. **Input polling**: Keyboard, mouse, touch
2. **Entity updates**: Movement, timers, state changes
3. **Spawning**: Enemies based on difficulty curve; pickups on enemy death
4. **Collision detection**: 
   - Player vs enemies (lose life if not invincible)
   - Player shots vs enemies (deal damage)
   - Enemy shots vs player (lose life)
   - Pickups vs player (upgrade weapon/collect bomb)
5. **Cleanup**: Remove dead entities, off-screen shots
6. **Rendering**: Draw all entities, HUD, effects
7. **Performance tracking**: Adjust quality if needed

#### Weapon System
Five weapons with distinct behaviors, each upgradeable to level 5:
- **VULCAN**: Rapid-fire spread shot (3 projectiles per shot)
- **LASER**: Single powerful beam (higher damage, pierces enemies)
- **GATLING**: Automatic sustained fire pattern
- **MISSILE**: Homing projectiles
- **SABER** (unlocked late): Melee weapon (closest range, highest damage)

Weapons level up when player collects matching pickups. Each level increases fire rate and damage.

#### Difficulty Progression
- Spawn rate decreases over time (enemy frequency increases as difficulty ramps)
- Kill counter tracks progress: every 18 kills triggers a stage change
- Boss spawns at stage end with health scaling by difficulty multiplier
- Multiplier system: combo multiplier (up to 8x) resets on player damage

#### Scoring System
- Base points from enemy kills (varies by enemy type)
- Multiplier applied per kill: 1x → 8x based on consecutive kills without damage
- Combo pop animation displays multiplier during active streak
- Best score persisted to localStorage

### 3. Input Handling
Multiple input methods coexist and don't interfere:

**Keyboard**:
- Arrow keys or WASD: Move player
- Space: Fire weapon
- X or Shift: Drop bomb
- P or Esc: Pause
- Mute/Motion toggle buttons (visual only on desktop)

**Mouse**:
- Movement: Player position follows mouse (desktop only)
- Click: Fire weapon
- Crosshair cursor feedback

**Touch**:
- Left 55% of screen (move zone): Drag to position player / show virtual joystick
- Right 45% (action zone): FIRE and BOMB buttons
- Joystick is optional visual aid for move zone

**Dynamic UI**: Touch controls shown only on devices with touch support; mouse hint shown on desktop

### 4. Visual & Audio
- **Canvas rendering**: Sprites drawn as geometric shapes (circles, rectangles, triangles)
- **Background**: Gradient sky theme rotating by stage, clouds, sun
- **Effects**: 
  - Flash on hit/bomb
  - Vignette (red pulse) when player takes damage
  - Particle trails on projectiles
  - Combo pop animation with scale and fade
  - Stage banner animation
- **Sounds**: 7 sound effects (shoot, hit, bomb, pickup, stage, boss, game over) - all beep/plink synthesis via Web Audio API (no external files)
- **Music**: Layered background track (looped Web Audio synthesis)
- **Accessibility**: Reduced motion CSS removes all animations if prefers-reduced-motion media query is active

### 5. Storage & Persistence
Uses browser localStorage keys:
- `skyfighter.bestScore`: High score (number)
- `skyfighter.muted`: Audio mute state (boolean)
- `skyfighter.difficulty`: Last selected difficulty (string: easy/normal/hard)
- `skyfighter.reducedMotion`: User prefers reduced motion (boolean)

## Key Constants & Configuration

Located near top of JavaScript section:

```javascript
DIFFICULTIES = { easy, normal, hard }           // Spawn rate, speed, shoot chance, lives, bombs, boss HP
KILLS_PER_STAGE = 18                            // Enemies to spawn before boss
THEMES = [6 color themes]                       // Stage backgrounds rotate through these
```

Within game loop, tunable parameters:
- Enemy spawn curve: `spawnBase`, `spawnMin` per difficulty
- Boss health multiplier: `bossHpMul` per difficulty
- Player collision radius, weapon projectile speed, bomb blast radius, etc.

## Development Notes

### Adding Features
- **New weapon**: Add WEAPON_NAME and sprite to weapon array, define firing pattern and damage
- **New enemy type**: Create enemy object with sprite pattern, health, behavior
- **New stage theme**: Add theme object to THEMES array with sky gradient and cloud/sun colors
- **Difficulty tuning**: Modify DIFFICULTIES parameters
- **Balance changes**: Adjust spawn rates, projectile speeds, collision sizes, and damage values

### Performance
- Adaptive quality system reduces particle count and draw complexity if frame time exceeds ~1/40 (25ms)
- Large arrays of entities could impact performance; consider pooling or spatial partitioning if entity count grows
- Canvas rendering is 2D primitive-based (no heavy layers or filters)

### Accessibility
- Screen reader announcements via `announce()` function (updates `#srLive` ARIA live region)
- Reduced motion support: Animations disabled if `prefers-reduced-motion` is set
- Game is keyboard-fully playable without mouse
- Touch support for mobile with visual button feedback

### Mobile Optimizations
- Viewport set to `user-scalable=no, viewport-fit=cover` for full-screen mobile experience
- Touch actions disabled to prevent browser gestures (pinch, long-press)
- Responsive font sizes using CSS `clamp()`
- Safe area insets respected for notch/island devices
- Portrait orientation assumed

## Testing
No formal test framework. Verify changes by:
1. Opening `index.html` in browser
2. Testing on target difficulty level
3. Checking enemy/boss spawning progression
4. Verifying weapon switching and upgrades work
5. Testing touch controls on mobile device or touch simulator
6. Verifying audio/mute toggle and reduced motion accessibility
7. Checking localStorage persistence (clear and replay to verify best score saves)

## Commit & Push
Push changes to the designated branch: `claude/inspiring-planck-tvelu5`
