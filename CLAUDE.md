# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game implemented as a single-file HTML/CSS/JavaScript application. The entire game (1600+ lines) is contained in `index.html` with no external dependencies or build process. Simply open the file in a browser to play.

## Running & Testing the Game

- **Play the game**: Open `index.html` in any modern web browser
- **Test on devices**: Use browser DevTools responsive design mode to test mobile (touch controls) and desktop (keyboard/mouse)
- **Keyboard controls**:
  - Arrow keys or WASD to move
  - Space or mouse click to fire
  - X, Shift, or click bomb button to use bombs
  - P or Escape to pause
- **Mouse controls**: Move cursor over canvas to steer; click to fire
- **Touch controls**: Left zone for movement joystick, right zone for fire button, bomb button in top-right
- **Accessibility**: Press mute button (🔊) or motion button (🎆) in top-right to toggle audio or reduce animations

## Code Architecture

The single `index.html` file is organized into these logical sections:

### 1. **HTML Structure & DOM Setup** (lines 1-400)
- Game canvas and overlay elements
- HUD elements: score, lives (hearts), bombs, weapon display, boss health bar
- Pause/menu overlays and difficulty buttons
- Touch control zones (left for movement, right for firing, bomb button)
- Icon buttons in top-right (mute, motion reduction, etc.)

### 2. **CSS Styling** (inline `<style>` tag)
- Responsive layout using CSS Grid and Flexbox with `clamp()` for scaling
- Dark theme with blue/gold accent colors
- Animations: heart beat (critical health), vignette pulse (damage), combo popups, stage banners
- Mobile-friendly safe-area insets for notches
- Reduced motion support via CSS class `reduced-motion`

### 3. **Game Configuration** (lines 450-520)
- `DIFFICULTIES` object: defines lives, bombs, spawn rates, enemy/boss scaling per difficulty level
- `ENEMY_TYPES` object: grunt, interceptor, tank with stats (HP, speed, score, color)
- `BOSSES` array: 6 boss definitions (Vanguard, Behemoth, Specter, Dreadnought, Wraith, Titan) with hull colors and HP multipliers
- `localStorage` keys for persisting best score, mute state, difficulty, reduced motion preference

### 4. **Audio System** (lines 540-680)
- WebAudio API (no external audio files—all sounds procedurally generated)
- `ensureAudio()`: initializes AudioContext and applies compression
- `beep()`: generates pitched tones with optional frequency sweep, filtering, panning
- `noiseBurst()`: generates white noise with filtering
- `sfx` object: named sound effects (shoot, laser, explode, hit, bomb, powerup, boss sounds, etc.)
- `EXPLODE_SFX` map: ties enemy types to their explosion sounds

### 5. **Canvas Sizing & Input Handling** (lines 750-900)
- **Canvas sizing**: Fixed 480×720 internal resolution, scaled responsively to fit window (max 1.6× upscale)
- **Keyboard input**: `keys` object tracks down state; Space/P/X/Shift trigger special actions
- **Mouse input**: Cursor position steers player; mouse button fires
- **Touch input**: Virtual joystick on left zone, fire button on right zone, bomb button in top-right
- **Input priority**: Keyboard/joystick > mouse (mouse doesn't interfere if keyboard is active)

### 6. **Game State & Core Loop** (lines 900–end)
- **State variables**: `player`, `bullets`, `enemies`, `enemyBullets`, `particles`, `powerUps`, `boss`
- **Scoring**: `score`, `chain` (hit streak), `multiplier` (combo multiplier), `kills` (per-stage), `lives`
- **Stage progression**: `stage`, `stageKills` (enemies defeated on current stage)
- **Game flow variables**: `running`, `paused`, `gameOver`, `frame` (frame counter)
- **Functions**:
  - `resetGame()`: Initialize all state for a new game
  - `update()`: Main game loop—updates all entities, handles spawning, collision, scoring
  - `draw()`: Renders player, enemies, bullets, particles, HUD, overlays
  - `drawPlane()`, `drawEnemy()`, `drawBoss()`: Entity rendering with custom shapes
  - `useBomb()`: Smart bomb that damages all enemies on screen
  - `togglePause()`: Pause/resume the game
  - `showGameOver()`, `showStageBanner()`, `showCombo()`: UI feedback functions

### 7. **Spawning & Difficulty Scaling** (within `update()`)
- Enemies spawn based on difficulty spawn rate and current stage
- Enemy count, speed, and HP increase per stage
- Boss spawns after defeating stage-specific kill target
- Boss health scales with difficulty multiplier

### 8. **Physics & Collision** (within `update()`)
- Enemies move downscreen at varying speeds and patterns (straight, weaving)
- Player bullets move upscreen; enemy bullets move downscreen
- Simple axis-aligned bounding-box collision (AABB)
- Collision points trigger sounds, particles, score, and knockback

### 9. **Weapon System**
- Two weapons: Vulcan (rapid fire, no sweep) and Laser (slower, sweeps across screen)
- Weapon picked up from power-ups; `player.level` tracks weapon power level (1–3)
- Weapon cooldown (`player.cooldown`) prevents overspray

### 10. **Scoring & Multiplier Chain**
- Base score per enemy type (grunt 10, interceptor 15, tank 35, boss varies)
- `chain` increments on each kill within `chainTimer` frames (roughly 0.5s)
- `multiplier = 1 + floor(chain / 5)` (caps at 4× on lunatic difficulty)
- Chain resets if player takes damage or a long time passes without kills
- Combo popup shows `"×4 CHAIN!"` when multiplier increases

## Key Modification Points

- **Difficulty tuning**: Edit `DIFFICULTIES` object (spawn rates, lives, bomb count, scaling)
- **Enemy behavior**: Modify `ENEMY_TYPES` or add new `enemySpawnPattern()` logic in `update()`
- **New weapons/levels**: Add to weapon switch logic in `usePowerUp()` and HUD rendering
- **Boss patterns**: Edit boss spawn detection, movement, and bullet patterns in the boss update section of `update()`
- **Sound design**: Adjust `sfx` object functions (frequency, duration, filter params)
- **Visual tweaks**: Edit CSS `#gameCanvas` styles or `draw*()` function colors/shapes
- **Accessibility**: Toggle `reduced-motion` class effects; verify touch target sizes (currently 50px+ radius)

## Common Development Tasks

- **Add an enemy type**: Add entry to `ENEMY_TYPES`, add spawn rate to boss/stage progression, add collision/explosion handling
- **Add a new boss**: Add entry to `BOSSES` array, add to stage progression logic, implement movement/attack patterns in `update()` boss section
- **Tune difficulty**: Adjust `DIFFICULTIES` multipliers (enemyScale, speedMul, spawnMul)
- **Adjust game feel**: Modify player speed, bullet speed, collision radii, particle counts, audio timing
- **Test on mobile**: Use browser responsive design mode (iPhone 12 Pro: 390×844) and check touch control positioning

## Performance Notes

- Canvas rendering is 2D (no WebGL), so performance is good on most devices
- Particle system caps at ~200 particles to maintain 60 FPS
- Audio is lazy-initialized (first user input) to avoid blocking on muted devices
- No build step; changes are live on browser refresh
