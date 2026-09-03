# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Sky Fighter** is a Raiden-style vertical shoot-'em-up game built as a single-file HTML5 application. Players control a fighter jet to destroy enemies, collect power-ups, defeat bosses, and earn a high score across multiple stages.

## Architecture

The entire game is contained in `index.html` (1669 lines) with three integrated sections:

### 1. HTML & DOM (Lines 1-420)
- Game canvas and HUD overlays
- Control buttons (start, pause, mute, motion toggle)
- Touch controls with virtual joystick for mobile
- Screen reader live region (`#srLive`) for accessibility
- Difficulty selection buttons
- Score display, weapon info, hearts/bombs indicators

### 2. CSS (Embedded in `<style>`)
- Dark space theme with gradient sky background
- Responsive design using `clamp()` for scaling
- Visual effects: vignette pulse on damage, flash on hit/bomb, heart-beat animation when critical
- HUD styling with backdrop blur and glassmorphism
- Touch-friendly sizing for mobile controls

### 3. JavaScript Game Engine (Lines 423-1669)
Self-contained IIFE (Immediately Invoked Function Expression) containing:

**Game Systems:**
- **Canvas & Rendering:** 2D canvas context, fullscreen adaptation, DPR handling
- **Game Loop:** RAF-based update/draw cycle with delta-time tracking
- **State Management:** Game states (menu, playing, paused, gameOver) and stage progression
- **Player:** Position, health, weapon level, bomb count, collision detection
- **Enemy Spawning:** `spawnEnemy()` creates random enemy types based on current stage/difficulty
- **Enemy Types:** grunt (quick, weak), interceptor (aimed shot), tank (armored)
- **Boss System:** `spawnBoss()` with `updateBoss()` handling stage-based boss battles using BOSS_PATTERNS
- **Weapon System:** Auto-targeting laser that upgrades through 3 levels; difficulty affects fire rate and damage
- **Particle Effects:** `spawnExplosion()` creates visual feedback for destroyed enemies
- **Input Handling:** Keyboard (Arrow keys, Spacebar), mouse, and touch/joystick controls
- **Audio:** Sound effects for firing, explosions, level up, boss spawn (Web Audio API)
- **HUD & UI:** Real-time score, multiplier, health hearts, bomb count, weapon info, boss health bar
- **Storage:** localStorage for best score, mute state, difficulty preference, reduced motion flag
- **Mobile Optimization:** Virtual joystick with deadzone, touch-friendly buttons, motion preference detection

## Key Configuration Constants

- **DIFFICULTIES:** Three difficulty levels affecting enemy spawn rate, speed, and player max health
- **KILLS_PER_STAGE:** 18 enemies per stage before boss encounter
- **ENEMY_TYPES:** 3 enemy variants with different behavior and scoring
- **BOSSES:** Array of boss configurations (one per stage)
- **BOSS_PATTERNS:** Movement and attack patterns indexed by boss
- **THEMES:** Cloud/background color schemes cycling through stages
- **BASE_W / BASE_H:** 480×720 base resolution (scales with window)

## Running the Game

1. **Development:** Open `index.html` directly in a web browser
   - Any modern browser (Chrome, Firefox, Safari, Edge)
   - No build step, no server required
   - Full game functionality with all features enabled

2. **Testing Specific Features:**
   - Keyboard controls: Arrow keys (move), Space (fire), B (bomb)
   - Touch: Joystick in lower-left, fire/bomb buttons in lower-right
   - Difficulty selection: Click before starting
   - Pause: Press P or click pause button during gameplay

3. **Debugging:**
   - Browser DevTools console for errors
   - Check localStorage via `localStorage.getItem('skyfighter.*')` commands
   - Frame rate monitoring visible in top-left (if enabled in code)

## Development Patterns

**Adding a New Enemy Type:**
1. Add entry to `ENEMY_TYPES` object with behavior/speed/health
2. Add to `EnemyTheme` in `spawnEnemy()` for visual rendering
3. Define explosion sound in `EXPLODE_SFX`
4. Add drawing function like `drawPlane()` or `drawBossShip()`

**Adding a New Boss:**
1. Append to `BOSSES` array (order determines stage appearance)
2. Define movement pattern in `BOSS_PATTERNS` array (must match length)
3. Implement drawing in `drawBossShip()` (check `bossType`)
4. Set health/rewards in boss config

**Adding a New Stage Theme:**
1. Add color config to `THEMES` array
2. Update `drawClouds()` if background visual needs change
3. Ensure boss is added to match new stage

**Adjusting Game Balance:**
- Fire rate: Adjust laser spawn interval in weapon level config
- Enemy difficulty curve: Modify difficulty constants or stage-based spawn logic
- Damage values: Change in collision detection sections
- Health/bombs: Modify `DIFFICULTIES[].maxHealth` or power-up drop rates

## Common Development Tasks

**Run the game:** Simply open `index.html` in browser (no npm/build required)

**Test a specific difficulty:** Click difficulty button before clicking "Start" 

**Adjust visual polish:** Modify CSS animations (vignette, flash, heartBeat) or particle colors in `spawnExplosion()`

**Change audio:** Web Audio API calls in `playSound()` — adjust frequency/duration for effects

**Mobile testing:** Use browser DevTools device emulation or physical device opening the file

**Performance profiling:** `trackPerf()` logs frame times to console; check for dropped frames above 16.67ms (60 FPS baseline)

## Storage Keys

The game persists user preferences via localStorage:
- `skyfighter.bestScore`: High score number
- `skyfighter.muted`: Boolean for audio mute state
- `skyfighter.difficulty`: Last selected difficulty (0-2)
- `skyfighter.reducedMotion`: Boolean for accessibility (disables animations if true)

Clear with `localStorage.clear()` to reset all game state.

## Accessibility Features

- Screen reader support: All game state announced via `#srLive` region
- Reduced motion: Respects `prefers-reduced-motion` media query and `reducedMotion` setting
- High contrast: Dark theme with high-contrast text
- Keyboard navigation: Full game playable with keyboard
- Touch-friendly: Large touch targets for buttons on mobile
