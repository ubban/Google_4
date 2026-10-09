# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single HTML5 file with embedded Canvas rendering and JavaScript game logic. The game features:
- Multiple difficulty levels (Easy, Normal, Hard)
- Progressive stages with enemy variety and boss encounters
- Weapon upgrade system (Vulcan, Spread, Laser)
- Smart bomb mechanics
- Full keyboard and touch/mobile controls
- Accessibility features (reduced motion, screen reader support)

## Running the Game

1. **Development/Testing**: Open `index.html` directly in a web browser (any modern browser with HTML5 Canvas support)
2. **Local server** (optional, for deployment testing): `python3 -m http.server 8000` then visit `http://localhost:8000/index.html`
3. **Live gameplay**: The game starts paused on the overlay screen. Click "Start Game" or press a difficulty button then click Start.

## Code Architecture

All code is in a single `index.html` file (~1669 lines):

### Structure
- **HTML (lines 1-420)**: DOM elements, canvas, overlay UI, touch controls
- **CSS (lines 10-359)**: Styling for HUD, overlays, animations, responsive design
- **JavaScript (lines 422+)**: Complete game engine in an IIFE

### Key Game Systems

#### 1. **State Management** (core game state)
- `gameState`: 'menu' | 'playing' | 'paused' | 'gameOver'
- `difficulty`: current difficulty setting affecting spawn rates, enemy behavior
- `player`: Position, health, weapon, bomb count
- `score`, `bestScore`: Game progression tracking
- `stage`: Current stage number (increments after killing KILLS_PER_STAGE enemies)

#### 2. **Entity Systems**
- **Player**: Single ship with position, velocity, weapon system, health
- **Enemies**: Array with types (basic, tank, fast) spawned with difficulty-dependent rates
- **Projectiles**: Player bullets and enemy bullets with collision detection
- **Boss**: Special entity spawned at stage 3 and every subsequent stage, with health bar UI
- **Power-ups**: Weapon upgrades (Vulcan, Spread, Laser) and bomb pickups

#### 3. **Rendering & Graphics** (`draw` function)
- Canvas 2D context rendering at adaptive frame rate
- `quality` variable (0-2) dynamically lowers rendering detail if FPS drops below target
- Plane sprites drawn via `drawPlane()` helper
- Boss drawn via `drawBossShip()` helper
- All positions in game coordinates, rendered to canvas

#### 4. **Input Handling**
- **Keyboard**: Arrow keys, WASD for movement; Space/X/Shift for fire/bomb; P/Esc for pause
- **Mouse**: Move to steer, click to fire (desktop); crosshair cursor
- **Touch**: Left zone for movement joystick, right zone for fire and bomb buttons
- Input state tracked in `keys` and `mousePos` objects

#### 5. **Collision & Physics**
- `checkCollisions()`: Bullet-enemy, enemy-player, player-powerup collisions
- `checkBossCollisions()`: Specialized collision checking for boss
- Simple AABB (axis-aligned bounding box) collision rectangles
- Position and velocity updates in `update()` function

#### 6. **Difficulty Settings** (DIFFICULTIES constant)
- Controls enemy spawn rate, speed, shooting probability, lives, bombs, boss HP multiplier
- Change multipliers in the `DIFFICULTIES` object to adjust game balance

#### 7. **Audio** (Web Audio API)
- Muted/unmuted state persisted in localStorage
- `playSound()` creates oscillator-based beeps (no audio files needed)
- Mute button UI in top-right

#### 8. **Persistence**
- Best score, mute state, selected difficulty, reduced motion preference saved to localStorage
- Keys: `skyfighter.bestScore`, `skyfighter.muted`, `skyfighter.difficulty`, `skyfighter.reducedMotion`

#### 9. **Accessibility**
- Screen reader announcements via `announce()` function and `srLive` element
- Reduced motion mode disables animations on preference or button toggle
- Semantic HTML roles and high contrast colors

### Important Constants & Configs

| Constant | Purpose | Location |
|----------|---------|----------|
| `DIFFICULTIES` | Spawn/speed/health by difficulty | ~line 485 |
| `KILLS_PER_STAGE` | Enemies to defeat before next stage | ~line 491 |
| `THEMES` | Sky gradient and cloud/sun colors per stage | ~line 493 |
| `WEAPON_TYPES` | Vulcan, Spread, Laser properties (spread, fireRate, etc.) | ~line 500+ |
| `ENEMY_TYPES` | Basic, Tank, Fast enemy specs | ~line 600+ |

## Development Notes

### Adding New Features
- **New Weapon**: Add to `WEAPON_TYPES` object with properties (spread, fireRate, damage), update `weaponLvl` calculation
- **New Enemy Type**: Add to `ENEMY_TYPES`, update spawn logic in `spawnEnemies()`
- **New Stage Theme**: Add to `THEMES` array with sky gradient colors and cloud/sun RGB values
- **Boss Variation**: Modify `updateBoss()` and `drawBossShip()` for behavior/appearance

### Performance Considerations
- The `quality` variable (0-2) auto-adjusts based on frame time averaging
  - At quality 0: Full glow effects, shadows
  - At quality 1-2: Reduced visual effects to maintain 40+ FPS
- Monitor frame time via the `frameTimes` array if optimizing

### Touch/Mobile
- Touch controls overlay (#touchControls) shown via CSS when needed
- Joystick base (#joystickBase) can be displayed for visual feedback
- Bottom zones: 55% height for movement, 45% for fire/bomb buttons

### Testing Gameplay
- Easy: Good for testing without pressure (4 lives, lower spawn/speed)
- Normal: Default balance
- Hard: Intense (3 lives, high spawn/speed, strong boss)
- Advance to boss by defeating 18 enemies per stage
- Test phase transitions, boss health bar, combo popups

## Git & Commits

When making changes:
- Keep commits focused on one feature or fix
- Describe "why" in commit messages, not just "what"
- Test in-browser after each change to verify visual/gameplay impact
