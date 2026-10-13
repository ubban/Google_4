# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 canvas application. The entire game (HTML, CSS, JavaScript, and canvas rendering) is contained in `index.html` (1669 lines).

**Key Features:**
- Three difficulty levels (Easy, Normal, Hard)
- Multiple weapon types with upgrade progression
- Boss encounters with varied attack patterns
- Combo multiplier system for consecutive hits
- Smart bombs for area damage
- Touch/mobile and keyboard/mouse controls
- Procedurally varied enemy and boss spawning
- Screen shake and visual feedback effects
- Sound effects (procedural generation via Web Audio API)
- Persistent high score tracking (localStorage)

## Running the Game

### Local Development
Simply open `index.html` in a web browser:
```bash
# Option 1: Direct file open
open index.html

# Option 2: Serve locally (Python 3)
python3 -m http.server 8000
# Then navigate to http://localhost:8000
```

No build process, dependencies, or server required. The game runs entirely client-side.

### Game Controls
- **Desktop (Mouse + Keyboard):**
  - Mouse movement: Steer the player
  - Click or Space: Fire weapon
  - X or Shift: Detonate bomb
  - Arrow keys or WASD: Move (alternative to mouse)
  - P or Esc: Pause

- **Mobile (Touch):**
  - Drag left side: Move player
  - Tap FIRE button: Shoot
  - Tap BOMB button: Smart bomb

## Architecture Overview

The game follows a single-file architecture with clear functional sections:

### Code Organization (by line ranges)
1. **CSS Styling** (lines 10–359): UI layout, animations, responsive design
2. **HTML DOM** (lines 361–420): Game canvas, HUD, overlay menus, touch controls
3. **Game Script** (lines 422–end):
   - Constants and configuration (difficulty presets, themes, weapon stats)
   - DOM element references
   - Game state variables (player, enemies, score, etc.)
   - Audio system (procedural SFX generation)
   - Game logic functions (update, draw, collision, spawning)
   - Event listeners (input handling, resize, window management)
   - Main game loop

### Key Game Systems

#### 1. **Player State & Mechanics**
- Location: `player` object (position, weapon, health, invulnerability)
- Weapon system: 5 types (Vulcan, Spread, Laser, Multi, Charge) with 3 levels each
- Health: 3 lives by default (configurable per difficulty)
- Invulnerability: Brief window after taking damage or using bomb
- Smart bombs: Limited-use screen-clear attacks
- See functions: `fireWeapon()`, `applyPowerUp()`, `useBomb()`

#### 2. **Enemy & Boss System**
- Regular enemies: Spawned in waves, each has position, velocity, and attack pattern
- Bosses: Appear after stage milestone, have 3 attack patterns, health bar UI
- Enemy bullets: Tracked separately, use varied patterns (radial spray, aimed spread, homing)
- Difficulty affects: Enemy spawn rate, bullet patterns, boss health
- See functions: `spawnEnemy()`, `spawnBoss()`, `updateBoss()`, `fireRadial()`, `fireAimedSpread()`, etc.

#### 3. **Scoring & Progression**
- Base score: Enemies (10–50 pts) and bosses (5000+ pts)
- Combo multiplier: Increases with consecutive kills (decay on miss)
- Stage progression: New stage every N enemies, increases difficulty
- Power-ups: Weapon upgrades, health, bomb ammo, score bonuses
- See functions: `addScore()`, `breakChain()`

#### 4. **Rendering Pipeline**
- Canvas-based 2D rendering (no WebGL)
- Theme system: 5 predefined color palettes cycling by stage
- Layers: Sky gradient → clouds → particles → objects → UI overlay
- Screen shake and red vignette effects for visual feedback
- See functions: `draw()`, `drawPlane()`, `drawBossShip()`, `drawClouds()`

#### 5. **Audio System**
- Web Audio API for procedural sound generation (no audio files)
- Sound categories: UI clicks, weapon fire, explosions, game over
- Configurable mute button
- Low-frequency noise synthesis for explosions
- See functions: `beep()`, `noiseBurst()`, and `sfx` object methods

#### 6. **Input Handling**
- Mouse: Position tracking for aiming
- Keyboard: Movement and action keys
- Touch: Virtual joystick on left, action buttons on right
- Accessibility: Screen reader announcements for game events
- See functions: `joystickStart()`, `joystickMove()`, canvas event listeners

#### 7. **Game State Management**
- Global state: `running`, `paused`, `gameOver`
- Per-frame timing: `frame` counter, `dt` delta time
- Persistence: localStorage for high score (`STORAGE_BEST`)
- Difficulty selection: Easy/Normal/Hard with different enemy/bullet configs
- See `resetGame()` for initialization, `update(dt)` for main tick

## Modification Guide

### Adjusting Difficulty
Edit the `DIFFICULTIES` object (around line 461):
```javascript
const DIFFICULTIES = {
  easy: { spawnRate: 0.015, bossSpawnRate: 0.012, ... },
  normal: { spawnRate: 0.025, bossSpawnRate: 0.020, ... },
  hard: { spawnRate: 0.04, bossSpawnRate: 0.035, ... }
};
```

### Adding New Weapon Types
1. Add weapon stats to `WEAPON_STATS` (line ~470)
2. Create a new fire pattern function (e.g., `fireMyWeapon()`)
3. Call it from `fireWeapon()` switch statement
4. Add weapon label to `weaponLabel()`

### Modifying Boss Attack Patterns
Edit `updateBoss()` function (line 1411):
- Boss phase changes based on health thresholds
- Each phase calls a different fire pattern function
- Add new patterns like `fireDoubleRing()` and call from the phase logic

### Adjusting Visual Effects
- **Color themes:** Edit `THEMES` array (line ~436)
- **Screen shake:** Modify `screenShake(magnitude, duration)` parameters
- **Animations:** CSS in `<style>` block (lines 10–359) uses standard keyframes

### Tuning Game Balance
Key parameters to adjust:
- `spawnRate`: Enemy spawn frequency (higher = more enemies)
- `bulletCount`, `bulletSpeed`: Boss bullet patterns
- Point values in `addScore()`
- Weapon fire rates in `fireWeapon()`
- Player movement speed and hitbox size

## Testing Changes

Since this is client-side only:
1. Save changes to `index.html`
2. Refresh browser (clear cache if needed: Cmd+Shift+R / Ctrl+Shift+R)
3. Test desired mechanic in-game across difficulties
4. Check mobile responsiveness if modifying layout

## Performance Notes

- Canvas rendering is the primary bottleneck; avoid creating too many particles or enemies
- Audio synthesis is fast but avoid excessive simultaneous sounds
- Touch joystick calculations run every frame; optimize if adding complex input
- No memory leaks expected (arrays pre-allocated, no accumulating event listeners)
